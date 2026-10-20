# Agent Management

Agents are the core unit of AI capabilities in DBridge Pro. Each Agent defines AI behavior, available tools, permission scope, and more — powering the AI Assistant, report generation, database inspection, and other features.

## Features

- Create and manage AI Agents
- Configure system prompts and output formats
- Bind MCP tools and Skills
- Sub-agent orchestration
- Data source permission control
- AI-assisted Agent generation
- Execution tracing

## Agent List

The "Agent Management" page displays all Agents:

| Column | Description |
|:-------|:------------|
| Name | Agent name |
| Description | Agent capability description |
| Runner Type | sdk / lite |
| Model | AI model in use |
| Scope | user (personal) / global |
| Actions | Edit, Delete, View Traces |

## Creating an Agent

Click the "New" button and fill in the Agent configuration:

### Basic Information

| Field | Description |
|:------|:------------|
| Name | Agent name |
| Description | Agent capability description |
| Runner Type | sdk (full features) / lite (lightweight single-turn) |
| Streaming | Whether to enable streaming output |

### Runner Types

| Type | Description | Use Case |
|:-----|:------------|:---------|
| sdk | Full Agent supporting multi-turn conversation, memory, tool calls | AI Assistant, complex tasks |
| lite | Lightweight Agent, single-turn execution, no memory | Auxiliary tools, one-time generation |

### Prompt Configuration

| Field | Description |
|:------|:------------|
| System Prompt | Agent behavior instructions (defines role, capabilities, constraints) |
| Result Format | Expected output structure |
| Output Format | markdown / json / yaml / text |

**System Prompt Example**:

```
You are a database assistant, specifically helping users write and optimize SQL queries.
Rules:
1. Only generate SELECT statements
2. Always add LIMIT constraints
3. Use the user's specified database dialect
4. Explain your SQL logic
```

### Model Configuration

| Field | Description |
|:------|:------------|
| Model | Select AI model (from configured Providers) |
| Temperature | Controls output randomness (0-1, recommend 0.1 for SQL generation) |
| Max Tokens | Maximum output token count |

### Tool Binding

| Type | Description |
|:-----|:------------|
| MCP Tools | Select MCP tools the Agent can call |
| Skills | Associated AI Skills |
| Sub-Agents | Delegatable sub-agents |

### Data Source Permissions

Fine-grained control over which data sources the Agent can access:

| Field | Description |
|:------|:------------|
| Data Source | Select allowed data sources |
| Database | Restrict to specific databases |
| Schema | Restrict to specific Schemas |
| Permission | read / write / admin |

> Agents without configured data source permissions can access all data sources. It is recommended to configure according to the principle of least privilege.

## AI-Generated Agents

The system supports automatically generating Agent configurations via AI:

1. Click the "AI Generate" button
2. Enter an Agent description, e.g., "An Agent specialized in database inspection"
3. AI automatically generates:
   - System prompt
   - Recommended tool bindings
   - Output format

## Built-in Agents

The system comes with several pre-configured Agents:

| Agent | Description |
|:------|:------------|
| Database Assistant | General SQL query assistant for the AI Assistant page |
| Report Generator | Automatically generates SQL reports from descriptions |
| Cube Generator | Generates semantic layer Cube models from descriptions |
| Mask Rule Generator | Analyzes table structure to generate data masking rules |
| Agent Generator | Generates new Agent configurations from descriptions |
| Skill Generator | Generates new Skill configurations from descriptions |

## Execution Tracing

Click the "Trace" button in the Agent list to view execution history:

| Info | Description |
|:-----|:------------|
| Execution Time | When it was executed |
| Input | User input content |
| Output | Agent output content |
| Tool Calls | Which MCP tools were called |
| Security Actions | Security rules triggered |
| Duration | Execution time |
| Token Usage | Input/output token count |

## Use Cases

### Database Query Assistant

```
Name: Database Assistant
Prompt: You are a database query assistant helping users write SQL
Tools: query_execute, schema_info
Permissions: read (read-only)
```

### Database Inspection

```
Name: Inspection Assistant
Prompt: Analyze database performance metrics and generate inspection reports
Tools: server_metrics, slow_sql_list, process_info
Output Format: markdown
```

### Report Generation

```
Name: Report Generator
Prompt: Generate SQL reports based on user descriptions
Tools: query_execute, schema_info
Output Format: json
```
