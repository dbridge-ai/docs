# AI Assistant

The AI Assistant is the intelligent conversation interface of DBridge Pro, supporting natural language database queries, SQL explanation, performance optimization, and more.

## Features

- Multi-session conversation management
- Streaming output (SSE)
- Data source and Schema binding
- SQL block inline execution
- AI Skill quick tags
- Multiple execution modes
- Slash commands

## Page Layout

```
┌──────────────┬─────────────────────────────────────┐
│  Sessions    │  [Agent Select ▼] [Data Source ▼] [Schema ▼] │
│              ├─────────────────────────────────────┤
│  + New       │  Conversation Messages              │
│  Session     │                                     │
│              │  👤 Query total orders for last 7 days │
│  🔍 Search   │                                     │
│              │  🤖 Here's the total for last 7 days: │
│  📌 Session 1│                                     │
│  📌 Session 2│  ```sql                              │
│  📌 Session 3│  SELECT SUM(amount)                  │
│  ...        │  FROM orders                         │
│              │  WHERE created_at >= ...             │
│              │  ```                                 │
│              │  [▶ Run] [📋 Copy] [⚡ Optimize] [💡 Explain] │
│              │                                     │
│              │  ┌─────────────────────────────┐    │
│              │  │ Type a message...            │    │
│              │  └─────────────────────────────┘    │
└──────────────┴─────────────────────────────────────┘
```

## Session Management

### Creating a Session

1. Click the **"+ New Session"** button
2. The system creates a new conversation session
3. The session title defaults to "New Session"

### Switching Sessions

Click a session item in the left session list to switch to that session:
- Loads message history
- Restores bound data source and Schema

### Renaming a Session

- Double-click the session title
- Or click the edit icon
- Enter a new name and save

### Deleting a Session

Click the delete icon, confirm to delete the session and all its messages.

### Searching Sessions

Enter keywords in the search box to filter the session list by title.

### Collapsing Panel

Click the collapse button to hide the left session panel, giving the conversation area full width. The collapsed state is remembered.

## Conversation Features

### Sending Messages

1. Type your question in the bottom input box
2. Press Enter or click the send button
3. AI returns a streaming response

### SQL Block Actions

SQL code blocks in AI responses provide the following action buttons:

| Button | Description |
|:-------|:------------|
| ▶ Run | Execute this SQL on the bound data source |
| 📋 Copy | Copy SQL to clipboard |
| ⚡ Optimize | Ask AI to optimize this SQL |
| 💡 Explain | Ask AI to explain this SQL |

Execution results are displayed inline in the message as a table.

### AI Skill Tags

Available AI Skill tags are displayed above the input box. Clicking one auto-fills the Prompt template:

- Explain SQL
- Optimize SQL
- Generate SQL
- Data Insights
- ...

### Execution Modes

The SQL execution mode can be selected in the input area:

| Mode | Description |
|:-----|:------------|
| auto | Automatically execute AI-generated SQL |
| approval | Requires human approval before execution |
| yolo | Execute without restrictions (use with caution) |

### Approval Cards

In approval mode, AI-generated SQL displays an approval card:

- View SQL content
- View risk level
- Click "Approve" to execute
- Click "Reject" to cancel

## Slash Commands

Slash commands can be used in the input box:

| Command | Description |
|:--------|:------------|
| /clear | Clear current session context |
| /compact | Compress current session context |
| /memory | View current memory status |

## Binding Data Sources

Before conversing, select the database context to operate on:

| Option | Description |
|:-------|:------------|
| Agent | Select AI Agent (determines AI behavior and capabilities) |
| Data Source | Select the database connection to query |
| Schema | Select the database/Schema |

> After binding a data source, AI automatically retrieves Schema information to generate more accurate SQL.

## Conversation Modes

The AI Assistant supports three conversation modes:

| Mode | Description |
|:-----|:------------|
| chat | Free conversation, Text-to-SQL |
| explain | SQL explanation mode |
| optimize | SQL optimization mode |

## Run Tracing

Click the "View Trace" button next to a message to see the detailed AI execution process:

- Tool call records
- Security operation logs
- Token consumption
- Execution duration

## Auto Title

When the session title is "New Session", the system automatically uses the first 20 characters of the first user message as the session title.

## Tips

### Effective Questioning

- Specify table names and field names explicitly
- Describe business requirements rather than technical implementation
- Refine step by step: check rough results first, then drill into details

### Multi-Turn Conversations

The AI Assistant supports multi-turn conversations, allowing follow-up questions based on context:

```
👤 Query total orders for the last 7 days
🤖 SELECT SUM(amount) FROM orders WHERE ...

👤 Group by day
🤖 SELECT DATE(created_at), SUM(amount) ... GROUP BY ...

👤 Add order count
🤖 SELECT DATE(created_at), COUNT(*), SUM(amount) ...
```

### Combining with SQL Editor

1. Generate SQL in the AI Assistant
2. Click "Run" to see results
3. Once satisfied, copy to the SQL Editor for further modification
