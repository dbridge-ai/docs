# DBridge Agent Memory System Detailed Guide

## 1. Overview

DBridge's memory system provides AI Agents with multi-turn dialogue context management and user preference persistence capabilities. The system employs a three-layer compression mechanism to maximize retention of valuable information within limited context windows, while asynchronously extracting critical information into long-term memory.

**Core Capabilities**:
- Multi-turn dialogue history management
- Session-level context compression (three-layer mechanism)
- User-level long-term memory extraction and recall
- Memory observability and monitoring

---

## 2. Architecture Design

### 2.1 Core Components

```
┌─────────────────────────────────────────────────────────┐
│                    Agent Execution Entry                  │
│              Executor.Execute() / ExecuteStream()        │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │   MemoryManager      │  ← Memory lifecycle management
              │  PreRun() / PostRun()│
              └─────────────────────┘
                         │
           ┌─────────────┼─────────────┐
           ▼             ▼             ▼
     History Load    User Memory    Compression
           │         Recall           Decision
           │             │             │
           ▼             ▼             ▼
    ──────────────┐
    │ MemoryService │  ← Core memory service
    │               │
    │ • Three-layer │
    │   compression │
    │ • Memory      │
    │   extraction  │
    │ • Memory      │
    │   recall      │
    │ • Token       │
    │   estimation  │
    └──────────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
Database       LLM Calls
Storage       (Compression/
(ai_messages,  Extraction)
 ai_session_memory,
 settings)
```

### 2.2 Key Files

| File | Responsibility |
|------|------|
| `internal/service/ai_memory.go` | `MemoryService` — Core memory service (compression, extraction, recall) |
| `internal/service/agentexec/memory.go` | `MemoryManager` — Execution lifecycle hooks (PreRun/PostRun) |
| `internal/service/agentexec/sdk_runner.go` | `sdkRunner` — Multi-turn Runner with full memory integration |
| `internal/service/agentexec/lite_runner.go` | `liteRunner` — Single-turn Runner without memory |
| `internal/service/agentexec/executor.go` | `Executor` — Calls `persistTranscript` to trigger memory saving |
| `internal/repository/models.go` | Database model definitions (`SessionMemory`, `Session`, `Message`) |
| `web/src/pages/MemoryMonitor.tsx` | Frontend monitoring dashboard |

---

## 3. Database Models

### 3.1 ai_sessions (Session Table)

```go
type Session struct {
    ID        string     `gorm:"primaryKey"`
    UserID    string
    AgentID   string
    Title     string
    ResetAt   *time.Time `gorm:"index"`  // Context reset timestamp
    CreatedAt time.Time
    UpdatedAt time.Time
}
```

**Key Fields**:
- `ResetAt`: Set when executing `/clear` or `/new` commands; history loading only reads messages after this time

### 3.2 ai_messages (Message Table)

```go
type Message struct {
    ID          string `gorm:"primaryKey"`
    SessionID   string `gorm:"index"`
    Role        string // "user" / "assistant" / "system"
    Content     string `gorm:"type:text"`
    SQLContent  string `gorm:"type:text"`  // Executed SQL
    QueryResult string `gorm:"type:text"`  // Query results
    Approval    string `gorm:"type:text"`  // Approval records
    CreatedAt   time.Time
}
```

**Stored Content**:
- User messages and assistant responses for each conversation turn
- Executed SQL statements (if any)
- Query result summaries (if any)
- Approval workflow records (if any)

### 3.3 ai_session_memory (Session Compression Table)

```go
type SessionMemory struct {
    ID           string    `gorm:"primaryKey"`
    SessionID    string    `gorm:"uniqueIndex"`
    Summary      string    `gorm:"type:text"`  // Compressed summary
    CompressedAt time.Time
    TokenCount   int
    MessageCount int
    LastMsgID    string
    Status       string    // "pending" / "success" / "failed"
}
```

**State Machine**:
- `pending`: L3 compression in progress (120-second TTL)
- `success`: Compression successful, summary available
- `failed`: Compression failed, can retry

### 3.4 settings (User Memory Storage)

User-extracted memories are stored in JSON format in the `settings` table:

| key | category | value format |
|-----|----------|-----------|
| `ai.user_memory.<userID>` | `ai` | `[{"id","content","category","source",...}]` |
| `ai.memory.last_extract.<userID>` | `ai` | Last extraction timestamp (for rate limiting) |
| `memory.L1.truncation_trigger` | `memory_metrics` | Counter (integer) |
| `memory.L2.trigger` | `memory_metrics` | Counter |
| `memory.L3.trigger` | `memory_metrics` | Counter |
| `memory.L3.success` | `memory_metrics` | Counter |
| `memory.L3.failure` | `memory_metrics` | Counter |

---

## 4. Agent Configuration and Memory Interaction

### 4.1 RunnerType Configuration

The `RunnerType` field in the Agent model controls memory behavior:

```go
type Agent struct {
    // ...
    RunnerType string `gorm:"type:varchar(20);default:sdk" json:"runner_type"`
    // ...
}
```

| RunnerType | Memory Behavior | Use Case |
|------------|----------|----------|
| `"sdk"` | Full memory (history loading, user memory recall, post-run extraction) | Multi-turn dialogue, complex tasks |
| `"lite"` | No memory (single-turn execution, no history saved) | Simple queries, tool calls |

### 4.2 Runner Selection Logic

Location: `internal/service/agentexec/executor.go` (line 322)

```go
func (e *Executor) selectRunner(req *RunRequest, agent *repository.Agent) AgentRunner {
    // Rule 1: Force sdkRunner when SessionID is present (even if Agent is configured as lite)
    if req.SessionID != "" {
        return e.sdk
    }
    // Rule 2: When no SessionID, select based on Agent.RunnerType or explicit Mode
    if req.Mode == "lite" || agent.RunnerType == "lite" {
        return e.lite
    }
    return e.sdk
}
```

**Key Rules**:
- Multi-turn dialogue (`SessionID` not empty) → Force `sdkRunner`
- Single-turn execution (`SessionID` empty) → Select based on configuration

### 4.3 RunResult.Memory Flag

```go
type RunResult struct {
    // ...
    Memory bool `json:"memory"` // Whether this execution connected to session memory
}
```

- `sdkRunner`: `res.Memory = req.SessionID != ""`
- `liteRunner`: Always `res.Memory = false`

---

## 5. Three-Layer Compression Mechanism

### 5.1 L1 — Message Truncation

**Trigger Condition**: Single message content exceeds 2000 characters

**Mechanism**: Head-tail truncation, preserving first 1000 characters and last 1000 characters

```go
func (s *MemoryService) TruncateMessage(content string, maxLen int) string {
    if len(content) <= maxLen {
        return content
    }
    head := content[:1000]
    tail := content[len(content)-1000:]
    return head + "\n...[truncated]...\n" + tail
}
```

**Application Scenarios**:
- During history loading (`loadSessionContext`)
- During context construction (`PromptBuilder.Build`)

**Monitoring Metric**: `memory.L1.truncation_trigger` (in `settings` table)

### 5.2 L2 — Summary Reuse

**Trigger Condition**: Session token estimate exceeds 50% of context window

**Mechanism**:
1. Check if `ai_session_memory` table has summaries with `status=success`
2. If exists, reuse directly, skip recompression
3. Summary inserted as `system` message at history beginning

```go
func (s *MemoryService) CheckAndCompact(ctx context.Context, sessionID string, tokenEstimate int, windowSize int) (*SessionMemory, error) {
    if tokenEstimate < windowSize/2 {
        return nil, nil  // Threshold not reached
    }
    // Look for existing summary
    var mem SessionMemory
    err := s.db.Where("session_id = ? AND status = ?", sessionID, "success").First(&mem).Error
    if err == nil {
        return &mem, nil  // Reuse successful
    }
    return nil, nil
}
```

**Monitoring Metric**: `memory.L2.trigger`

### 5.3 L3 — LLM Full Compaction

**Trigger Condition**: Session token estimate exceeds 75% of context window

**Mechanism**:
1. Asynchronous goroutine calls LLM
2. Uses structured prompt (10 dimensions)
3. Summarizes all history except latest 5 messages
4. State machine management (pending → success/failed)
5. 120-second TTL prevents stuck state after crashes

**Compression Prompt** (`compactPrompt`, ai_memory.go line 506):

```
Please provide a structured summary of the following conversation history, covering these 10 dimensions:
1. Main requests and intentions
2. Key decisions and conclusions
3. Created/modified objects (tables, fields, configurations, etc.)
4. Schema changes
5. Data operations (INSERT/UPDATE/DELETE)
6. Errors and solutions
7. To-do items (TODO)
8. User preferences
9. Tool call records
10. Important context information

Output format: JSON
```

**State Machine Flow**:

```
Trigger L3
   │
   ▼
Create SessionMemory (status=pending)
   │
   ▼
Async LLM compression call
   │
   ├─ Success → status=success, save summary
   │
   └─ Failure → status=failed, log error
```

**Monitoring Metrics**:
- `memory.L3.trigger` — L3 trigger count
- `memory.L3.success` — Success count
- `memory.L3.failure` — Failure count

---

## 6. User Memory Extraction and Recall

### 6.1 Extraction Trigger Timing

**Location**: `MemoryManager.PostRun()` → `ExtractMemories()`

**Trigger Conditions**:
1. `result.Memory == true` (sdkRunner only)
2. More than 30 seconds since last extraction (rate limiting)
3. Executes asynchronously, does not block main flow

```go
func (m *MemoryManager) PostRun(ctx context.Context, req *RunRequest, result *RunResult) {
    // 1. Save user message and assistant message to ai_messages
    m.saveMessages(req, result)

    // 2. If sdkRunner and has SessionID, trigger memory extraction
    if result.Memory && req.SessionID != "" {
        go m.memorySvc.ExtractMemories(ctx, req.UserID, req.SessionID)
    }
}
```

### 6.2 Extraction Process

Location: `internal/service/ai_memory.go` (line 729)

```
1. Check rate limiting (no repeated extraction within 30 seconds)
   │
   ▼
2. Load last 10 messages from session
   │
   ▼
3. Call LLM to extract memories
   Prompt: "Extract user preferences, project information, database knowledge, etc. from the following conversation..."
   Output format: [{"content": "...", "category": "preference|db_info|project|pattern"}]
   │
   ▼
4. Save to settings table
   - Maximum 50 items per user
   - Maximum 1000 characters per item
   - Evict oldest when limit exceeded
```

**Memory Categories**:

| Category | Description | Example |
|----------|------|------|
| `preference` | User preferences | "User prefers concise response style" |
| `db_info` | Database knowledge | "orders table status field enum values: pending/paid/shipped" |
| `project` | Project information | "Project uses MySQL 8.0, deployed on Alibaba Cloud" |
| `pattern` | Behavioral patterns | "User typically checks sales reports on Monday mornings" |

### 6.3 Recall Mechanism

Location: `internal/service/ai_memory.go` (line 654)

**Trigger Timing**: `MemoryManager.PreRun()` → `recall()` → `GetUserMemories()`

**Recall Process**:

```
1. Load user memory JSON from settings table
   │
   ▼
2. Filter by DataSourceID
   - Match memories for current data source
   - Global memories (DataSourceID is empty)
   │
   ▼
3. Calculate score for each memory
   Score = Keyword match score + Time decay score + Recency bonus
   │
   ▼
4. Return Top 3 memories
```

**Scoring Algorithm**:

```go
func scoreMemory(item MemoryItem, keywords []string, now time.Time) float64 {
    score := 0.0

    // 1. Keyword match (+50)
    for _, kw := range keywords {
        if strings.Contains(strings.ToLower(item.Content), strings.ToLower(kw)) {
            score += 50
            break
        }
    }

    // 2. Time decay (100 * (1 - days/180))
    days := now.Sub(item.CreatedAt).Hours() / 24
    decayScore := 100 * (1 - days/180)
    if decayScore < 0 {
        decayScore = 0
    }
    score += decayScore

    // 3. Recency bonus (+30 within 30 minutes)
    if days < 0.02 {  // 30 minutes
        score += 30
    }

    return score
}
```

**Recall Example**:

```
User asks: "Help me check order data"
Keywords: ["订单", "order"]

Memory store:
- "User focuses on amount and status fields in orders table" (30 days ago, keyword match)
  Score: 50(match) + 83(decay) = 133

- "orders table is on MySQL master" (10 days ago, no keyword match)
  Score: 0(no match) + 94(decay) = 94

- "User prefers grouping by date" (20 minutes ago, no keyword match)
  Score: 0(no match) + 99(decay) + 30(recency) = 129

Return Top 3: [Memory 1, Memory 3, Memory 2]
```

---

## 7. Memory Lifecycle

### 7.1 Creation (PreRun)

Location: `internal/service/agentexec/memory.go` (line 40)

```go
func (m *MemoryManager) PreRun(ctx context.Context, req *RunRequest) (*MemoryContext, error) {
    // 1. Skip when no SessionID
    if req.SessionID == "" {
        return &MemoryContext{}, nil
    }

    // 2. Recall user memories
    memories := m.memorySvc.GetUserMemories(ctx, req.UserID, req.DataSourceID)

    // 3. Load session history (respect reset_at and summary boundaries)
    messages, err := m.loadSessionContext(ctx, req.SessionID)

    // 4. Compression decision
    tokenEstimate := estimateTokens(messages)
    summary, _ := m.memorySvc.CheckAndCompact(ctx, req.SessionID, tokenEstimate, windowSize)
    m.memorySvc.TriggerL3IfNeeded(ctx, req.SessionID, tokenEstimate, windowSize)

    return &MemoryContext{
        History:      messages,
        UserMemories: memories,
        TokenEstimate: tokenEstimate,
    }, nil
}
```

### 7.2 Storage (PostRun)

Location: `internal/service/agentexec/memory.go` (line 84)

```go
func (m *MemoryManager) PostRun(ctx context.Context, req *RunRequest, result *RunResult) {
    // 1. Save messages to ai_messages (sole write point)
    m.saveMessages(req, result)

    // 2. Trigger async memory extraction (sdkRunner only)
    if result.Memory && req.SessionID != "" {
        go m.memorySvc.ExtractMemories(ctx, req.UserID, req.SessionID)
    }
}
```

**Call Chain**:
```
Executor.Execute()
   │
   ▼
runCoreSec()
   │
   ▼
runner.Run()  ← sdkRunner or liteRunner
   │
   ▼
Executor.persistTranscript()
   │
   ▼
MemoryManager.PostRun()
```

### 7.3 Expiration and Reset

**Session Reset** (`/clear` or `/new` command):

```go
func (h *SessionHandler) Reset(c *gin.Context) {
    sessionID := c.Param("id")
    now := time.Now()
    db.Model(&Session{}).Where("id = ?", sessionID).Update("reset_at", now)
}
```

Effect:
- `loadSessionContext()` only loads messages after `reset_at`
- Historical context is "cleared" (actual data retained, just not loaded)

**User Memory Eviction**:
- No explicit expiration time
- Time decay scoring lowers scores for older memories
- Hard limit of 50 items, evicts oldest when exceeded

---

## 8. Observability Metrics

### 8.1 Metric Definitions

All metrics stored in `settings` table with category `memory_metrics`:

| Metric Key | Meaning | Trigger Timing |
|----------|------|----------|
| `memory.L1.truncation_trigger` | L1 truncation trigger count | When truncating messages in `PromptBuilder.Build()` |
| `memory.L2.trigger` | L2 summary reuse trigger count | When `CheckAndCompact()` detects Token > 50% window |
| `memory.L3.trigger` | L3 compression trigger count | When `TriggerL3IfNeeded()` detects Token > 75% window |
| `memory.L3.success` | L3 compression success count | When L3 goroutine completes successfully |
| `memory.L3.failure` | L3 compression failure count | When L3 goroutine fails |

### 8.2 Monitoring Dashboard

Location: `web/src/pages/MemoryMonitor.tsx`

**Features**:
- Displays counters and summary cards for 5 metrics
- Documents three-layer compression logic
- Explains available commands (`/clear`, `/compact`, `/memory`)

**Access Path**: System Management → Memory Monitor (or directly enter `/memory-monitor` in URL)

### 8.3 Log Output

Code uses `zap.S().Infof` and `zap.S().Errorf` to log key events:

```
[agentexec] PreRun: loading history for session=xxx
[agentexec] PreRun: loaded 15 messages for session=xxx
[agentexec] save user message failed: ...
[MEMORY] L3 compaction SUCCESS session=xxx, tokens=4500
[MEMORY] L3 compaction FAILED session=xxx: context window exceeded
```

### 8.4 Audit Logs

Location: `internal/service/agentexec/executor.go` (line 136)

Each Agent run recorded to `audit_logs` table:

```json
{
  "module": "ai",
  "operation": "agent_run",
  "details": {
    "agent_id": "agent_xxx",
    "session_id": "session_xxx",
    "memory": true,
    "runner_used": "sdk",
    "token_estimate": 3500,
    "duration_ms": 1234
  }
}
```

---

## 9. Viewing Memory Content

### 9.1 API Endpoints

| Endpoint | Method | Purpose |
|------|------|------|
| `/api/v1/ai-sessions` | GET | List all sessions for current user |
| `/api/v1/ai-sessions/:id` | GET | Get session details (with messages) |
| `/api/v1/ai-sessions/:id/messages` | POST | Append messages |
| `/api/v1/ai-sessions/:id/reset` | POST | Reset session context (set `reset_at`) |
| `/api/v1/ai-sessions/:id/compact` | POST | Manually trigger L3 compression |
| `/api/v1/memory` | GET | Get extracted memories for current user |

Route definition location: `internal/handler/routes.go` (lines 339-357)

### 9.2 Bot Commands

Location: `internal/service/bot_command.go`

| Command | Function |
|------|------|
| `/clear` or `/new` | Reset session (set `reset_at`, clear context) |
| `/compact` | Manually trigger L3 compression |
| `/memory` | Display all extracted user memories (with categories) |

### 9.3 Database Queries

```sql
-- 1. View session conversation history
SELECT id, role, LEFT(content, 100) AS content_preview, created_at
FROM ai_messages
WHERE session_id = '<session_id>'
ORDER BY created_at ASC;

-- 2. View session compression status
SELECT session_id, summary, token_count, message_count, status, compressed_at
FROM ai_session_memory
WHERE session_id = '<session_id>';

-- 3. View user extracted memories (JSON format)
SELECT key, LEFT(value, 500) AS memory_preview
FROM settings
WHERE key = 'ai.user_memory.<user_id>';

-- 4. View memory metrics
SELECT key, value
FROM settings
WHERE category = 'memory_metrics';

-- 5. View session reset status
SELECT id, reset_at, created_at, updated_at
FROM ai_sessions
WHERE id = '<session_id>';

-- 6. View memory-related info in audit logs
SELECT created_at, details->>'$.memory' AS memory_enabled,
       details->>'$.runner_used' AS runner,
       details->>'$.token_estimate' AS tokens
FROM audit_logs
WHERE module = 'ai' AND operation = 'agent_run'
ORDER BY created_at DESC
LIMIT 20;
```

### 9.4 Frontend Interface

**MemoryMonitor Page** (`/memory-monitor`):
- Displays compression metric counters
- Shows three-layer compression mechanism documentation
- Lists available commands

**ChatPanel Component** (`web/src/components/ChatPanel.tsx`):
- Renders session message history
- Displays SQL execution results
- Shows approval workflows

**AIAssistant Page** (`/ai-assistant`):
- Main chat interface
- Session management (create, switch, reset)
- Memory viewing (`/memory` command)

---

## 10. Memory Scope and Isolation

| Scope | Mechanism | Storage Location |
|------|------|----------|
| **Session-level** | `ai_messages` isolated by `SessionID` | `ai_messages` table |
| **Session-level** | `ai_session_memory` isolated by `SessionID` | `ai_session_memory` table |
| **User-level** | User memories isolated by `UserID` | `settings` table (key contains userID) |
| **Data source-level** | `MemoryItem.DataSourceID` filters recall | `settings` table (field within JSON) |
| **Agent-level** | `MemoryItem.AgentID` marks source | `settings` table (field within JSON) |

**Isolation Rules**:
- Session history: Different sessions completely isolated
- User memories: Isolated by user, can be further segmented by `DataSourceID`
- Cross-session: User memories can be shared across sessions (same user)

---

## 11. Performance and Limits

### 11.1 Token Estimation

Location: `internal/service/ai_memory.go`

```go
func estimateTokenCount(messages []types.ChatCompletionMessage) int {
    total := 0
    for _, msg := range messages {
        // Rough estimate: 1 Chinese character ≈ 1.5 tokens, 1 English character ≈ 0.25 tokens
        total += len(msg.Content) * 2 / 3
    }
    return total
}
```

### 11.2 Limit Parameters

| Parameter | Value | Description |
|------|-----|------|
| L1 truncation threshold | 2000 characters | Triggers truncation when single message exceeds this value |
| L1 preserved length | 1000 + 1000 characters | Preserves 1000 characters each from head and tail |
| L2 trigger threshold | 50% context window | Checks summary when token estimate exceeds 50% of window |
| L3 trigger threshold | 75% context window | Triggers compression when token estimate exceeds 75% of window |
| L3 TTL | 120 seconds | Pending status times out, considered failed |
| L3 preserved message count | 5 messages | Preserves latest 5 messages uncompressed during compaction |
| User memory limit | 50 items/user | Evicts oldest when exceeded |
| Single memory length | 1000 characters | Truncated during extraction |
| Extraction rate limiting | 30 seconds/user | Prevents frequent LLM calls |

### 11.3 Performance Optimization

1. **Asynchronous extraction**: `ExtractMemories()` executes in goroutine, does not block main flow
2. **Summary reuse**: L2 mechanism avoids repeated compression
3. **Rate limiting protection**: 30-second extraction interval prevents LLM call overload
4. **Index optimization**: `ai_messages.session_id` and `ai_session_memory.session_id` both indexed

---

## 12. Best Practices

### 12.1 Agent Configuration Recommendations

| Scenario | RunnerType | Description |
|------|------------|------|
| Complex multi-turn dialogue | `sdk` | Requires full memory and context |
| Simple queries/tool calls | `lite` | No memory needed, faster execution |
| Mixed scenarios | `sdk` + manual SessionID management | Flexible control |

### 12.2 Session Management Recommendations

1. **Reset promptly**: Use `/clear` to reset context when switching topics
2. **Manual compression**: Use `/compact` to proactively compress during long conversations
3. **Regular review**: Use `/memory` to check if extracted memories are accurate

### 12.3 Monitoring Recommendations

1. **Monitor L3 failure rate**: `memory.L3.failure / memory.L3.trigger` should be < 10%
2. **Monitor L1 trigger frequency**: Frequent truncation may indicate messages are too long
3. **Regular cleanup**: Delete sessions no longer needed to free storage space

---

## 13. Troubleshooting

### 13.1 Common Issues

| Issue | Possible Cause | Solution |
|------|----------|----------|
| Memory not saved | Agent configured as `lite` | Change to `sdk` or ensure SessionID is present |
| Compression not triggered | Token estimate below threshold | Check context window configuration |
| L3 compression failed | LLM call timeout or error | Check logs, verify LLM service status |
| Memory recall inaccurate | Scoring algorithm mismatch | Check keywords and time decay |
| User memories empty | Extraction conditions not met | Check rate limiting and SessionID |

### 13.2 Debugging Methods

1. **Check logs**:
   ```bash
   tail -f ~/.local/share/dbridge/logs/dbridge.log | grep -i memory
   ```

2. **Check database**:
   ```sql
   SELECT * FROM ai_session_memory WHERE session_id = '<session_id>';
   SELECT * FROM settings WHERE key LIKE 'ai.user_memory.%';
   ```

3. **Manually trigger compression**:
   ```bash
   curl -X POST http://localhost:8003/api/v1/ai-sessions/<session_id>/compact \
     -H "Authorization: Bearer <token>"
   ```

4. **View extracted memories**:
   ```bash
   curl http://localhost:8003/api/v1/memory \
     -H "Authorization: Bearer <token>"
   ```

---

## 14. Complete Examples

### 14.1 Multi-turn Dialogue Flow

```
User: "Help me create an order analysis model"
   │
   ▼
PreRun:
   - Load history (empty, first turn)
   - Recall user memories (none)
   - Token estimate: 100
   │
   ▼
LLM execution → Generate Cube configuration
   │
   ▼
PostRun:
   - Save messages to ai_messages
   - Trigger memory extraction (async)
   - Extracted: "User is creating order analysis model" (category: project)
   │
   ▼
User: "Add dimension for grouping by region"
   │
   ▼
PreRun:
   - Load history (2 messages)
   - Recall user memories (matches "order" keyword)
   - Token estimate: 500
   │
   ▼
LLM execution → Update Cube configuration (combining history and memory)
   │
   ▼
PostRun:
   - Save messages
   - Update memory: "User created order Cube, includes region dimension"
```

### 14.2 Long Conversation Compression Flow

```
Conversation reaches turn 20
   │
   ▼
PreRun:
   - Load history (40 messages)
   - Token estimate: 8000 (80% of 10000 window)
   │
   ▼
L2 check:
   - 80% > 50% → Check summary
   - No existing summary → Continue
   │
   ▼
L3 trigger:
   - 80% > 75% → Trigger compression
   - Create SessionMemory (status=pending)
   - Async LLM compression of first 35 messages
   │
   ▼
Compression complete:
   - status=success
   - Summary: "User created order Cube, added region and time dimensions, configured 5 measures..."
   │
   ▼
Next turn PreRun:
   - Load summary (as system message)
   - Load latest 5 messages
   - Token estimate: 2000 (significant decrease)
```

---

## 15. Key Code Location Index

| Component | File Path | Key Line Numbers |
|------|----------|----------|
| **MemoryService Core** | `internal/service/ai_memory.go` | Throughout |
| L1 truncation | `internal/service/ai_memory.go` | line 119 |
| L2 summary reuse | `internal/service/ai_memory.go` | line 332 |
| L3 compression prompt | `internal/service/ai_memory.go` | line 506 |
| L3 compression execution | `internal/service/ai_memory.go` | line 434 |
| Memory extraction | `internal/service/ai_memory.go` | line 729 |
| Memory recall | `internal/service/ai_memory.go` | line 654 |
| Scoring algorithm | `internal/service/ai_memory.go` | line 670 |
| **MemoryManager** | `internal/service/agentexec/memory.go` | Throughout |
| PreRun hook | `internal/service/agentexec/memory.go` | line 40 |
| PostRun hook | `internal/service/agentexec/memory.go` | line 84 |
| **Runner Implementation** | `internal/service/agentexec/sdk_runner.go` | Throughout |
| Memory integration | `internal/service/agentexec/sdk_runner.go` | line 68 |
| **Executor** | `internal/service/agentexec/executor.go` | Throughout |
| Runner selection | `internal/service/agentexec/executor.go` | line 322 |
| Transcript persistence | `internal/service/agentexec/executor.go` | line 136 |
| **Database Models** | `internal/repository/models.go` | Throughout |
| SessionMemory | `internal/repository/models.go` | line 836 |
| Session | `internal/repository/models.go` | line 244 |
| Message | `internal/repository/models.go` | line 263 |
| **Frontend Monitoring** | `web/src/pages/MemoryMonitor.tsx` | Throughout |
| **API Routes** | `internal/handler/routes.go` | lines 339-357 |
| **Bot Commands** | `internal/service/bot_command.go` | Throughout |

---

**Document Version**: 1.0
**Last Updated**: 2026-10-19
**Maintainer**: DBridge Development Team
