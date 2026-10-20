# MCP Tools

MCP (Model Context Protocol) tools are functional units that AI Agents can call. DBridge Pro includes a rich set of built-in MCP tools and supports creating custom tools.

## Features

- View built-in MCP tools
- Create custom SQL tools
- Edit and delete custom tools
- Enable/disable tools
- Permission level management

## Tool List

The "MCP Tools" page displays all available tools:

| Column | Description |
|:-------|:------------|
| Name | Tool display name |
| Tool ID | Unique tool identifier |
| Type | builtin / custom |
| Category | Tool category |
| Permission | read / write / admin |
| Status | Enable/disable toggle |
| Description | Tool capability description |

### Filtering

- **Search** — Search by name
- **Category filter** — Filter by tool category
- **Type filter** — Built-in / Custom

## Built-in Tools

The system includes the following categories of MCP tools:

### Admin Tools

| Tool | Description |
|:-----|:------------|
| Server Info | Get server runtime status and version information |
| System Config | View and modify system configuration |

### DataSource Tools

| Tool | Description |
|:-----|:------------|
| Data Source List | Get all data sources |
| Data Source Detail | Get data source connection information |
| Test Connection | Test data source connectivity |

### Query Tools

| Tool | Description |
|:-----|:------------|
| Execute Query | Execute SQL query and return results |
| Execute DML | Execute INSERT/UPDATE/DELETE |

### Schema Tools

| Tool | Description |
|:-----|:------------|
| Database List | Get database list |
| Table List | Get table list for a specific database |
| Table Structure | Get table column information |
| Index Info | Get table indexes |

### Compare Tools

| Tool | Description |
|:-----|:------------|
| Structure Compare | Compare structure differences between two data sources |
| Table Data Compare | Compare data differences between two tables |

### Structure Tools

| Tool | Description |
|:-----|:------------|
| Create Table | Create a table in the target database |
| Modify Table | Modify table structure |
| Drop Table | Delete a table |

### Server Tools

| Tool | Description |
|:-----|:------------|
| Process Info | Get database process list |
| User List | Get database users |
| Role Info | Get roles and permissions |

### Metric Tools

| Tool | Description |
|:-----|:------------|
| Server Metrics | Get CPU, memory, connection count, and other metrics |
| Slow Query List | Get current slow queries |

### Slow SQL Tools

| Tool | Description |
|:-----|:------------|
| Slow SQL Analysis | Analyze slow query causes |
| EXPLAIN | Execute EXPLAIN to get query plan |

### Semantic Tools

| Tool | Description |
|:-----|:------------|
| Cube List | Get semantic layer Cube list |
| Metric Query | Execute semantic layer metric queries |

## Custom Tools

In addition to built-in tools, you can create custom SQL tools:

### Creating a Custom Tool

1. Click the "New" button
2. Fill in the tool information:

| Field | Description |
|:------|:------------|
| Name | Tool display name |
| Tool ID | Unique identifier (English, e.g., `my_custom_query`) |
| Description | Tool capability description (AI uses this to decide when to call) |
| Category | Tool category |
| Permission Level | read / write / admin |
| SQL Template | SQL statement the tool executes |

3. SQL templates support parameter placeholders:

```sql
SELECT * FROM {{table_name}}
WHERE created_at >= '{{start_date}}'
LIMIT {{limit}}
```

4. The tool is immediately available after saving

### Editing Custom Tools

1. Find the target tool
2. Click the "Edit" button
3. Modify the configuration
4. Save

> Built-in tools cannot be edited.

### Deleting Custom Tools

1. Find the target tool
2. Click the "Delete" button
3. Confirm deletion

> Built-in tools cannot be deleted.

## Enable / Disable

Control tool availability via toggle:

- **Enabled** — Agents can call the tool
- **Disabled** — Agents cannot call the tool

> Disabling a tool does not delete its configuration; it can be re-enabled at any time.

## Permission Levels

| Level | Description | Examples |
|:------|:------------|:---------|
| read | Read-only operations | Queries, view structure |
| write | Write operations | Create tables, modify data |
| admin | Administrative operations | Drop tables, modify configuration |

When an Agent calls a tool, the system checks whether the Agent's data source permissions satisfy the tool's permission requirements.

## Tool Details

Click a tool name to open the detail Drawer and view:

- Complete tool configuration
- SQL template
- Parameter definitions
- Usage instructions
- Call examples
