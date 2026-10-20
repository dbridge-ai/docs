# DBridge Agent 记忆系统详解

## 1. 概述

DBridge 的记忆系统为 AI Agent 提供多轮对话上下文管理和用户偏好持久化能力。系统采用三层压缩机制，在有限的上下文窗口内最大化保留有价值的信息，同时通过异步提取机制将关键信息沉淀为长期记忆。

**核心能力**：
- 多轮对话历史管理
- 会话级上下文压缩（三层机制）
- 用户级长期记忆提取与召回
- 记忆可观测性与监控

---

## 2. 架构设计

### 2.1 核心组件

```
┌─────────────────────────────────────────────────────────┐
│                    Agent 执行入口                         │
│              Executor.Execute() / ExecuteStream()        │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │   MemoryManager      │  ← 记忆生命周期管理
              │  PreRun() / PostRun()│
              └─────────────────────┘
                         │
           ┌─────────────┼─────────────┐
           ▼             ▼             ▼
     历史加载        用户记忆召回     压缩决策
           │             │             │
           ▼             ▼             ▼
    ──────────────┐
    │ MemoryService │  ← 核心记忆服务
    │               │
    │ • 三层压缩    │
    │ • 记忆提取    │
    │ • 记忆召回    │
    │ • Token 估算  │
    └──────────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
数据库存储      LLM 调用
(ai_messages,  (压缩/提取)
 ai_session_memory,
 settings)
```

### 2.2 关键文件

| 文件 | 职责 |
|------|------|
| `internal/service/ai_memory.go` | `MemoryService` — 核心记忆服务（压缩、提取、召回） |
| `internal/service/agentexec/memory.go` | `MemoryManager` — 执行生命周期钩子（PreRun/PostRun） |
| `internal/service/agentexec/sdk_runner.go` | `sdkRunner` — 集成完整记忆的多轮 Runner |
| `internal/service/agentexec/lite_runner.go` | `liteRunner` — 无记忆的单轮 Runner |
| `internal/service/agentexec/executor.go` | `Executor` — 调用 `persistTranscript` 触发记忆保存 |
| `internal/repository/models.go` | 数据库模型定义（`SessionMemory`、`Session`、`Message`） |
| `web/src/pages/MemoryMonitor.tsx` | 前端监控面板 |

---

## 3. 数据库模型

### 3.1 ai_sessions（会话表）

```go
type Session struct {
    ID        string     `gorm:"primaryKey"`
    UserID    string
    AgentID   string
    Title     string
    ResetAt   *time.Time `gorm:"index"`  // 上下文重置时间点
    CreatedAt time.Time
    UpdatedAt time.Time
}
```

**关键字段**：
- `ResetAt`：执行 `/clear` 或 `/new` 命令时设置，历史加载只读取此时间之后的消息

### 3.2 ai_messages（消息表）

```go
type Message struct {
    ID          string `gorm:"primaryKey"`
    SessionID   string `gorm:"index"`
    Role        string // "user" / "assistant" / "system"
    Content     string `gorm:"type:text"`
    SQLContent  string `gorm:"type:text"`  // 执行的 SQL
    QueryResult string `gorm:"type:text"`  // 查询结果
    Approval    string `gorm:"type:text"`  // 审批记录
    CreatedAt   time.Time
}
```

**存储内容**：
- 每轮对话的用户消息和助手回复
- 执行的 SQL 语句（如有）
- 查询结果摘要（如有）
- 审批流程记录（如有）

### 3.3 ai_session_memory（会话压缩表）

```go
type SessionMemory struct {
    ID           string    `gorm:"primaryKey"`
    SessionID    string    `gorm:"uniqueIndex"`
    Summary      string    `gorm:"type:text"`  // 压缩后的摘要
    CompressedAt time.Time
    TokenCount   int
    MessageCount int
    LastMsgID    string
    Status       string    // "pending" / "success" / "failed"
}
```

**状态机**：
- `pending`：L3 压缩进行中（120 秒 TTL）
- `success`：压缩成功，摘要可用
- `failed`：压缩失败，可重试

### 3.4 settings（用户记忆存储）

用户提取的记忆以 JSON 格式存储在 `settings` 表中：

| key | category | value 格式 |
|-----|----------|-----------|
| `ai.user_memory.<userID>` | `ai` | `[{"id","content","category","source",...}]` |
| `ai.memory.last_extract.<userID>` | `ai` | 上次提取时间戳（用于限流） |
| `memory.L1.truncation_trigger` | `memory_metrics` | 计数器（整数） |
| `memory.L2.trigger` | `memory_metrics` | 计数器 |
| `memory.L3.trigger` | `memory_metrics` | 计数器 |
| `memory.L3.success` | `memory_metrics` | 计数器 |
| `memory.L3.failure` | `memory_metrics` | 计数器 |

---

## 4. Agent 配置与记忆交互

### 4.1 RunnerType 配置

Agent 模型中的 `RunnerType` 字段控制记忆行为：

```go
type Agent struct {
    // ...
    RunnerType string `gorm:"type:varchar(20);default:sdk" json:"runner_type"`
    // ...
}
```

| RunnerType | 记忆行为 | 适用场景 |
|------------|----------|----------|
| `"sdk"` | 完整记忆（历史加载、用户记忆召回、后运行提取） | 多轮对话、复杂任务 |
| `"lite"` | 无记忆（单轮执行，不保存历史） | 简单查询、工具调用 |

### 4.2 Runner 选择逻辑

位置：`internal/service/agentexec/executor.go` (line 322)

```go
func (e *Executor) selectRunner(req *RunRequest, agent *repository.Agent) AgentRunner {
    // 规则 1: 有 SessionID 时强制使用 sdkRunner（即使 Agent 配置为 lite）
    if req.SessionID != "" {
        return e.sdk
    }
    // 规则 2: 无 SessionID 时按 Agent.RunnerType 或显式 Mode 选择
    if req.Mode == "lite" || agent.RunnerType == "lite" {
        return e.lite
    }
    return e.sdk
}
```

**关键规则**：
- 多轮对话（`SessionID` 非空）→ 强制 `sdkRunner`
- 单轮执行（`SessionID` 为空）→ 按配置选择

### 4.3 RunResult.Memory 标志

```go
type RunResult struct {
    // ...
    Memory bool `json:"memory"` // 本次执行是否连接会话记忆
}
```

- `sdkRunner`：`res.Memory = req.SessionID != ""`
- `liteRunner`：始终 `res.Memory = false`

---

## 5. 三层压缩机制

### 5.1 L1 — 消息截断（Truncation）

**触发条件**：单条消息内容超过 2000 字符

**机制**：头尾截断，保留前 1000 字符和后 1000 字符

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

**应用场景**：
- 历史加载时（`loadSessionContext`）
- 上下文构建时（`PromptBuilder.Build`）

**监控指标**：`memory.L1.truncation_trigger`（在 `settings` 表中）

### 5.2 L2 — 摘要复用（Summary Reuse）

**触发条件**：会话 Token 估算值超过上下文窗口的 50%

**机制**：
1. 检查 `ai_session_memory` 表是否存在 `status=success` 的摘要
2. 如存在，直接复用，跳过重新压缩
3. 摘要作为 `system` 消息插入历史开头

```go
func (s *MemoryService) CheckAndCompact(ctx context.Context, sessionID string, tokenEstimate int, windowSize int) (*SessionMemory, error) {
    if tokenEstimate < windowSize/2 {
        return nil, nil  // 未达阈值
    }
    // 查找已有摘要
    var mem SessionMemory
    err := s.db.Where("session_id = ? AND status = ?", sessionID, "success").First(&mem).Error
    if err == nil {
        return &mem, nil  // 复用成功
    }
    return nil, nil
}
```

**监控指标**：`memory.L2.trigger`

### 5.3 L3 — LLM 全量压缩（Full Compaction）

**触发条件**：会话 Token 估算值超过上下文窗口的 75%

**机制**：
1. 异步 goroutine 调用 LLM
2. 使用结构化提示词（10 个维度）
3. 摘要除最新 5 条消息外的所有历史
4. 状态机管理（pending → success/failed）
5. 120 秒 TTL 防止崩溃后卡死

**压缩提示词**（`compactPrompt`，ai_memory.go line 506）：

```
请对以下对话历史进行结构化摘要，覆盖以下 10 个维度：
1. 主要请求与意图
2. 关键决策与结论
3. 创建/修改的对象（表、字段、配置等）
4. Schema 变更
5. 数据操作（INSERT/UPDATE/DELETE）
6. 错误与解决方案
7. 待办事项（TODO）
8. 用户偏好
9. 工具调用记录
10. 重要上下文信息

输出格式：JSON
```

**状态机流程**：

```
触发 L3
   │
   ▼
创建 SessionMemory (status=pending)
   │
   ▼
异步调用 LLM 压缩
   │
   ├─ 成功 → status=success, 保存摘要
   │
   └─ 失败 → status=failed, 记录错误
```

**监控指标**：
- `memory.L3.trigger` — L3 触发次数
- `memory.L3.success` — 成功次数
- `memory.L3.failure` — 失败次数

---

## 6. 用户记忆提取与召回

### 6.1 提取触发时机

**位置**：`MemoryManager.PostRun()` → `ExtractMemories()`

**触发条件**：
1. `result.Memory == true`（仅 sdkRunner）
2. 距上次提取超过 30 秒（限流）
3. 异步执行，不阻塞主流程

```go
func (m *MemoryManager) PostRun(ctx context.Context, req *RunRequest, result *RunResult) {
    // 1. 保存用户消息和助手消息到 ai_messages
    m.saveMessages(req, result)

    // 2. 如果是 sdkRunner 且有 SessionID，触发记忆提取
    if result.Memory && req.SessionID != "" {
        go m.memorySvc.ExtractMemories(ctx, req.UserID, req.SessionID)
    }
}
```

### 6.2 提取流程

位置：`internal/service/ai_memory.go` (line 729)

```
1. 检查限流（30 秒内不重复提取）
   │
   ▼
2. 加载会话最近 10 条消息
   │
   ▼
3. 调用 LLM 提取记忆
   提示词: "从以下对话中提取用户偏好、项目信息、数据库知识等..."
   输出格式: [{"content": "...", "category": "preference|db_info|project|pattern"}]
   │
   ▼
4. 保存到 settings 表
   - 每用户最多 50 条
   - 每条最多 1000 字符
   - 超出时淘汰最旧的
```

**记忆类别**：

| Category | 说明 | 示例 |
|----------|------|------|
| `preference` | 用户偏好 | "用户喜欢简洁的回答风格" |
| `db_info` | 数据库知识 | "orders 表的 status 字段枚举值：pending/paid/shipped" |
| `project` | 项目信息 | "项目使用 MySQL 8.0，部署在阿里云" |
| `pattern` | 行为模式 | "用户通常在周一上午查看销售报表" |

### 6.3 召回机制

位置：`internal/service/ai_memory.go` (line 654)

**触发时机**：`MemoryManager.PreRun()` → `recall()` → `GetUserMemories()`

**召回流程**：

```
1. 从 settings 表加载用户记忆 JSON
   │
   ▼
2. 按 DataSourceID 过滤
   - 匹配当前数据源的记忆
   - 全局记忆（DataSourceID 为空）
   │
   ▼
3. 计算每条记忆的得分
   得分 = 关键词匹配分 + 时间衰减分 + 近期加分
   │
   ▼
4. 返回 Top 3 记忆
```

**评分算法**：

```go
func scoreMemory(item MemoryItem, keywords []string, now time.Time) float64 {
    score := 0.0

    // 1. 关键词匹配（+50）
    for _, kw := range keywords {
        if strings.Contains(strings.ToLower(item.Content), strings.ToLower(kw)) {
            score += 50
            break
        }
    }

    // 2. 时间衰减（100 * (1 - days/180)）
    days := now.Sub(item.CreatedAt).Hours() / 24
    decayScore := 100 * (1 - days/180)
    if decayScore < 0 {
        decayScore = 0
    }
    score += decayScore

    // 3. 近期加分（30 分钟内 +30）
    if days < 0.02 {  // 30 分钟
        score += 30
    }

    return score
}
```

**召回示例**：

```
用户问: "帮我查一下订单数据"
关键词: ["订单", "order"]

记忆库:
- "用户关注订单表的 amount 和 status 字段" (30 天前, 匹配关键词)
  得分: 50(匹配) + 83(衰减) = 133

- "orders 表在 MySQL 主库" (10 天前, 无关键词匹配)
  得分: 0(无匹配) + 94(衰减) = 94

- "用户喜欢按日期分组查看" (20 分钟前, 无关键词匹配)
  得分: 0(无匹配) + 99(衰减) + 30(近期) = 129

返回 Top 3: [记忆 1, 记忆 3, 记忆 2]
```

---

## 7. 记忆生命周期

### 7.1 创建（PreRun）

位置：`internal/service/agentexec/memory.go` (line 40)

```go
func (m *MemoryManager) PreRun(ctx context.Context, req *RunRequest) (*MemoryContext, error) {
    // 1. 无 SessionID 时跳过
    if req.SessionID == "" {
        return &MemoryContext{}, nil
    }

    // 2. 召回用户记忆
    memories := m.memorySvc.GetUserMemories(ctx, req.UserID, req.DataSourceID)

    // 3. 加载会话历史（尊重 reset_at 和摘要边界）
    messages, err := m.loadSessionContext(ctx, req.SessionID)

    // 4. 压缩决策
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

### 7.2 存储（PostRun）

位置：`internal/service/agentexec/memory.go` (line 84)

```go
func (m *MemoryManager) PostRun(ctx context.Context, req *RunRequest, result *RunResult) {
    // 1. 保存消息到 ai_messages（唯一写入点）
    m.saveMessages(req, result)

    // 2. 触发异步记忆提取（仅 sdkRunner）
    if result.Memory && req.SessionID != "" {
        go m.memorySvc.ExtractMemories(ctx, req.UserID, req.SessionID)
    }
}
```

**调用链**：
```
Executor.Execute()
   │
   ▼
runCoreSec()
   │
   ▼
runner.Run()  ← sdkRunner 或 liteRunner
   │
   ▼
Executor.persistTranscript()
   │
   ▼
MemoryManager.PostRun()
```

### 7.3 过期与重置

**会话重置**（`/clear` 或 `/new` 命令）：

```go
func (h *SessionHandler) Reset(c *gin.Context) {
    sessionID := c.Param("id")
    now := time.Now()
    db.Model(&Session{}).Where("id = ?", sessionID).Update("reset_at", now)
}
```

效果：
- `loadSessionContext()` 只加载 `reset_at` 之后的消息
- 历史上下文被"清空"（实际数据保留，只是不再加载）

**用户记忆淘汰**：
- 无显式过期时间
- 时间衰减评分使旧记忆得分降低
- 硬上限 50 条，超出时淘汰最旧的

---

## 8. 观测指标

### 8.1 指标定义

所有指标存储在 `settings` 表，category 为 `memory_metrics`：

| 指标 Key | 含义 | 触发时机 |
|----------|------|----------|
| `memory.L1.truncation_trigger` | L1 截断触发次数 | `PromptBuilder.Build()` 中截断消息时 |
| `memory.L2.trigger` | L2 摘要复用触发次数 | `CheckAndCompact()` 检测到 Token > 50% 窗口时 |
| `memory.L3.trigger` | L3 压缩触发次数 | `TriggerL3IfNeeded()` 检测到 Token > 75% 窗口时 |
| `memory.L3.success` | L3 压缩成功次数 | L3 goroutine 成功完成时 |
| `memory.L3.failure` | L3 压缩失败次数 | L3 goroutine 失败时 |

### 8.2 监控面板

位置：`web/src/pages/MemoryMonitor.tsx`

**功能**：
- 显示 5 个指标的计数器和摘要卡片
- 文档化三层压缩逻辑
- 说明可用命令（`/clear`、`/compact`、`/memory`）

**访问路径**：系统管理 → 记忆监控（或直接在 URL 输入 `/memory-monitor`）

### 8.3 日志输出

代码使用 `zap.S().Infof` 和 `zap.S().Errorf` 记录关键事件：

```
[agentexec] PreRun: loading history for session=xxx
[agentexec] PreRun: loaded 15 messages for session=xxx
[agentexec] save user message failed: ...
[MEMORY] L3 compaction SUCCESS session=xxx, tokens=4500
[MEMORY] L3 compaction FAILED session=xxx: context window exceeded
```

### 8.4 审计日志

位置：`internal/service/agentexec/executor.go` (line 136)

每次 Agent 运行记录到 `audit_logs` 表：

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

## 9. 查看记忆内容

### 9.1 API 端点

| 端点 | 方法 | 用途 |
|------|------|------|
| `/api/v1/ai-sessions` | GET | 列出当前用户的所有会话 |
| `/api/v1/ai-sessions/:id` | GET | 获取会话详情（含消息） |
| `/api/v1/ai-sessions/:id/messages` | POST | 追加消息 |
| `/api/v1/ai-sessions/:id/reset` | POST | 重置会话上下文（设置 `reset_at`） |
| `/api/v1/ai-sessions/:id/compact` | POST | 手动触发 L3 压缩 |
| `/api/v1/memory` | GET | 获取当前用户的提取记忆 |

路由定义位置：`internal/handler/routes.go` (lines 339-357)

### 9.2 Bot 命令

位置：`internal/service/bot_command.go`

| 命令 | 功能 |
|------|------|
| `/clear` 或 `/new` | 重置会话（设置 `reset_at`，清空上下文） |
| `/compact` | 手动触发 L3 压缩 |
| `/memory` | 显示所有提取的用户记忆（含类别） |

### 9.3 数据库查询

```sql
-- 1. 查看会话对话历史
SELECT id, role, LEFT(content, 100) AS content_preview, created_at
FROM ai_messages
WHERE session_id = '<session_id>'
ORDER BY created_at ASC;

-- 2. 查看会话压缩状态
SELECT session_id, summary, token_count, message_count, status, compressed_at
FROM ai_session_memory
WHERE session_id = '<session_id>';

-- 3. 查看用户提取的记忆（JSON 格式）
SELECT key, LEFT(value, 500) AS memory_preview
FROM settings
WHERE key = 'ai.user_memory.<user_id>';

-- 4. 查看记忆指标
SELECT key, value
FROM settings
WHERE category = 'memory_metrics';

-- 5. 查看会话重置状态
SELECT id, reset_at, created_at, updated_at
FROM ai_sessions
WHERE id = '<session_id>';

-- 6. 查看审计日志中的记忆相关信息
SELECT created_at, details->>'$.memory' AS memory_enabled,
       details->>'$.runner_used' AS runner,
       details->>'$.token_estimate' AS tokens
FROM audit_logs
WHERE module = 'ai' AND operation = 'agent_run'
ORDER BY created_at DESC
LIMIT 20;
```

### 9.4 前端界面

**MemoryMonitor 页面**（`/memory-monitor`）：
- 显示压缩指标计数器
- 展示三层压缩机制说明
- 列出可用命令

**ChatPanel 组件**（`web/src/components/ChatPanel.tsx`）：
- 渲染会话消息历史
- 显示 SQL 执行结果
- 展示审批流程

**AIAssistant 页面**（`/ai-assistant`）：
- 主聊天界面
- 会话管理（新建、切换、重置）
- 记忆查看（`/memory` 命令）

---

## 10. 记忆范围与隔离

| 范围 | 机制 | 存储位置 |
|------|------|----------|
| **会话级** | `ai_messages` 按 `SessionID` 隔离 | `ai_messages` 表 |
| **会话级** | `ai_session_memory` 按 `SessionID` 隔离 | `ai_session_memory` 表 |
| **用户级** | 用户记忆按 `UserID` 隔离 | `settings` 表（key 含 userID） |
| **数据源级** | `MemoryItem.DataSourceID` 过滤召回 | `settings` 表（JSON 内字段） |
| **Agent 级** | `MemoryItem.AgentID` 标记来源 | `settings` 表（JSON 内字段） |

**隔离规则**：
- 会话历史：不同会话完全隔离
- 用户记忆：按用户隔离，但可通过 `DataSourceID` 进一步细分
- 跨会话：用户记忆可在不同会话间共享（同一用户）

---

## 11. 性能与限制

### 11.1 Token 估算

位置：`internal/service/ai_memory.go`

```go
func estimateTokenCount(messages []types.ChatCompletionMessage) int {
    total := 0
    for _, msg := range messages {
        // 粗略估算：1 个中文字符 ≈ 1.5 tokens，1 个英文字符 ≈ 0.25 tokens
        total += len(msg.Content) * 2 / 3
    }
    return total
}
```

### 11.2 限制参数

| 参数 | 值 | 说明 |
|------|-----|------|
| L1 截断阈值 | 2000 字符 | 单条消息超过此值触发截断 |
| L1 保留长度 | 1000 + 1000 字符 | 头尾各保留 1000 字符 |
| L2 触发阈值 | 50% 上下文窗口 | Token 估算值超过窗口 50% 时检查摘要 |
| L3 触发阈值 | 75% 上下文窗口 | Token 估算值超过窗口 75% 时触发压缩 |
| L3 TTL | 120 秒 | pending 状态超时后视为失败 |
| L3 保留消息数 | 5 条 | 压缩时保留最新 5 条不压缩 |
| 用户记忆上限 | 50 条/用户 | 超出时淘汰最旧的 |
| 单条记忆长度 | 1000 字符 | 提取时截断 |
| 提取限流 | 30 秒/用户 | 防止频繁调用 LLM |

### 11.3 性能优化

1. **异步提取**：`ExtractMemories()` 在 goroutine 中执行，不阻塞主流程
2. **摘要复用**：L2 机制避免重复压缩
3. **限流保护**：30 秒提取间隔防止 LLM 调用过载
4. **索引优化**：`ai_messages.session_id`、`ai_session_memory.session_id` 均有索引

---

## 12. 最佳实践

### 12.1 Agent 配置建议

| 场景 | RunnerType | 说明 |
|------|------------|------|
| 复杂多轮对话 | `sdk` | 需要完整记忆和上下文 |
| 简单查询/工具调用 | `lite` | 无需记忆，执行更快 |
| 混合场景 | `sdk` + 手动管理 SessionID | 灵活控制 |

### 12.2 会话管理建议

1. **及时重置**：话题切换时使用 `/clear` 重置上下文
2. **手动压缩**：长对话中使用 `/compact` 主动压缩
3. **定期查看**：使用 `/memory` 检查提取的记忆是否准确

### 12.3 监控建议

1. **关注 L3 失败率**：`memory.L3.failure / memory.L3.trigger` 应 < 10%
2. **关注 L1 触发频率**：频繁截断可能意味着消息过长
3. **定期清理**：删除不再需要的会话，释放存储空间

---

## 13. 故障排查

### 13.1 常见问题

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 记忆未保存 | Agent 配置为 `lite` | 改为 `sdk` 或确保有 SessionID |
| 压缩未触发 | Token 估算未达阈值 | 检查上下文窗口配置 |
| L3 压缩失败 | LLM 调用超时或错误 | 查看日志，检查 LLM 服务状态 |
| 记忆召回不准确 | 评分算法不匹配 | 检查关键词和时间衰减 |
| 用户记忆为空 | 未达到提取条件 | 检查限流和 SessionID |

### 13.2 调试方法

1. **查看日志**：
   ```bash
   tail -f ~/.local/share/dbridge/logs/dbridge.log | grep -i memory
   ```

2. **检查数据库**：
   ```sql
   SELECT * FROM ai_session_memory WHERE session_id = '<session_id>';
   SELECT * FROM settings WHERE key LIKE 'ai.user_memory.%';
   ```

3. **手动触发压缩**：
   ```bash
   curl -X POST http://localhost:8003/api/v1/ai-sessions/<session_id>/compact \
     -H "Authorization: Bearer <token>"
   ```

4. **查看提取的记忆**：
   ```bash
   curl http://localhost:8003/api/v1/memory \
     -H "Authorization: Bearer <token>"
   ```

---

## 14. 完整示例

### 14.1 多轮对话流程

```
用户: "帮我创建一个订单分析模型"
   │
   ▼
PreRun:
   - 加载历史（空，首轮）
   - 召回用户记忆（无）
   - Token 估算：100
   │
   ▼
LLM 执行 → 生成 Cube 配置
   │
   ▼
PostRun:
   - 保存消息到 ai_messages
   - 触发记忆提取（异步）
   - 提取："用户正在创建订单分析模型" (category: project)
   │
   ▼
用户: "加上按地区分组的维度"
   │
   ▼
PreRun:
   - 加载历史（2 条消息）
   - 召回用户记忆（匹配"订单"关键词）
   - Token 估算：500
   │
   ▼
LLM 执行 → 更新 Cube 配置（结合历史和记忆）
   │
   ▼
PostRun:
   - 保存消息
   - 更新记忆："用户创建了订单 Cube，包含地区维度"
```

### 14.2 长对话压缩流程

```
对话进行到第 20 轮
   │
   ▼
PreRun:
   - 加载历史（40 条消息）
   - Token 估算：8000（窗口 10000 的 80%）
   │
   ▼
L2 检查：
   - 80% > 50% → 检查摘要
   - 无现有摘要 → 继续
   │
   ▼
L3 触发：
   - 80% > 75% → 触发压缩
   - 创建 SessionMemory (status=pending)
   - 异步调用 LLM 压缩前 35 条消息
   │
   ▼
压缩完成：
   - status=success
   - 摘要："用户创建了订单 Cube，添加了地区、时间维度，配置了 5 个度量..."
   │
   ▼
下一轮 PreRun:
   - 加载摘要（作为 system 消息）
   - 加载最近 5 条消息
   - Token 估算：2000（大幅下降）
```

---

## 15. 关键代码位置索引

| 组件 | 文件路径 | 关键行号 |
|------|----------|----------|
| **MemoryService 核心** | `internal/service/ai_memory.go` | 全文 |
| L1 截断 | `internal/service/ai_memory.go` | line 119 |
| L2 摘要复用 | `internal/service/ai_memory.go` | line 332 |
| L3 压缩提示词 | `internal/service/ai_memory.go` | line 506 |
| L3 压缩执行 | `internal/service/ai_memory.go` | line 434 |
| 记忆提取 | `internal/service/ai_memory.go` | line 729 |
| 记忆召回 | `internal/service/ai_memory.go` | line 654 |
| 评分算法 | `internal/service/ai_memory.go` | line 670 |
| **MemoryManager** | `internal/service/agentexec/memory.go` | 全文 |
| PreRun 钩子 | `internal/service/agentexec/memory.go` | line 40 |
| PostRun 钩子 | `internal/service/agentexec/memory.go` | line 84 |
| **Runner 实现** | `internal/service/agentexec/sdk_runner.go` | 全文 |
| 记忆集成 | `internal/service/agentexec/sdk_runner.go` | line 68 |
| **Executor** | `internal/service/agentexec/executor.go` | 全文 |
| Runner 选择 | `internal/service/agentexec/executor.go` | line 322 |
| Transcript 持久化 | `internal/service/agentexec/executor.go` | line 136 |
| **数据库模型** | `internal/repository/models.go` | 全文 |
| SessionMemory | `internal/repository/models.go` | line 836 |
| Session | `internal/repository/models.go` | line 244 |
| Message | `internal/repository/models.go` | line 263 |
| **前端监控** | `web/src/pages/MemoryMonitor.tsx` | 全文 |
| **API 路由** | `internal/handler/routes.go` | lines 339-357 |
| **Bot 命令** | `internal/service/bot_command.go` | 全文 |

---

**文档版本**: 1.0  
**最后更新**: 2026-10-19  
**维护者**: DBridge 开发团队
