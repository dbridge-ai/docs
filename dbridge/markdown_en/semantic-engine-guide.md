# DBridge Semantic Engine and Semantic Models

## 1. Overview

The DBridge Semantic Layer is the bridge connecting business users with underlying databases. It abstracts physical database tables into business-friendly **Cubes (data models)**, defining business metric systems through **Measures** and **Dimensions**, enabling non-technical users to perform complex data analysis through natural language or simple configurations.

Core value of the semantic layer:
- **Business Semantics Encapsulation**: Encapsulates SQL expressions, aggregation logic, and table relationships into reusable business metrics
- **Unified Query Interface**: Regardless of whether the underlying database is MySQL, PostgreSQL, or other databases, the upper layer uses a unified query syntax
- **AI-Friendly**: Provides structured metadata for Large Language Models (LLMs), enabling AI to accurately understand business data and generate correct queries

---

## 2. Core Concepts

### 2.1 Cube (Data Model)

A Cube is the fundamental unit of the semantic layer, representing a business subject area (e.g., "Orders", "Users", "Products").

**Database Storage**: `semantic_cubes` table

| Field | Type | Description |
|------|------|------|
| `id` | UUID | Primary key |
| `name` | string | Unique identifier, e.g., `orders` |
| `display_name` | string | Display name, e.g., `Orders` |
| `description` | text | Business description |
| `data_source_id` | string | Associated data source ID |
| `database` | string | Database name (optional) |
| `schema_name` | string | Schema name (optional) |
| `sql_table` | string | Physical table name, e.g., `public.orders` |
| `sql_query` | text | Custom SQL query (mutually exclusive with sql_table) |
| `measures_json` | JSON | Measure definition array |
| `dimensions_json` | JSON | Dimension definition array |
| `joins_json` | JSON | Join definition array |
| `pre_aggs_json` | JSON | Pre-aggregation definition array |
| `version` | int | Version number, incremented on each update |
| `tenant_id` | string | Tenant ID (multi-tenant isolation) |

### 2.2 Measure

Measures are aggregatable business metrics, such as "Total Order Amount", "Number of Users".

```json
{
  "name": "total_amount",
  "display_name": "Total Order Amount",
  "description": "Sum of amounts across all orders",
  "type": "sum",
  "sql": "SUM({field})",
  "field": "amount"
}
```

**Supported Aggregation Types**:
- `sum` — Sum
- `avg` — Average
- `min` / `max` — Minimum / Maximum
- `count` — Count
- `count_distinct` — Distinct count
- `number` — Number (no aggregation)
- `running_total` — Running total

### 2.3 Dimension

Dimensions are grouping or filtering attributes of data, such as "Date", "Region", "Status".

```json
{
  "name": "date",
  "display_name": "Order Date",
  "description": "Order creation date",
  "sql": "order_date",
  "time_granularity": "day"
}
```

**Time Dimension Granularities**: `day`, `week`, `month`, `quarter`, `year`

### 2.4 Join

Cubes can be associated with each other through Joins, enabling cross-model queries.

```json
{
  "join_cube_name": "users",
  "relationship": "many_to_one",
  "sql": "orders.user_id = users.id",
  "join_type": "left"
}
```

---

## 3. Semantic Engine Architecture

### 3.1 Overall Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Access Layer                           │
│  REST API  │  MCP Server  │  Agent SDK  │  Frontend UI  │
└────────────┴──────────────┴─────────────┴────────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │  SemanticQueryService│  ← Query orchestration service
              ─────────────────────┘
                         │
           ┌─────────────┼─────────────┐
           ▼             ▼             ▼
     Validation     Audit Log     Row-Level
      Service                      Security
           │
           ▼
    ┌──────────────┐
    │  Adapter Registry │  ← Adapter registry center
    └──────────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
LocalAdapter  CubeJSAdapter  ← Pluggable engines
    │
    ▼
┌──────────────┐
│ LocalCompiler │  ← SQL compiler
└──────────────┘
    │
    ▼
──────────────┐
│  QueryService │  ← Underlying SQL execution
└──────────────┘
```

### 3.2 Core Components

#### SemanticQueryService (Query Orchestration)

Location: `internal/semantic/service/query_service.go`

Responsible for the complete query lifecycle:
1. **Validation** — Validates field formats, operator whitelists, Limit upper bounds
2. **Routing** — Selects adapter based on engine name
3. **Row-Level Security** — Applies data permission filters
4. **Execution** — Calls adapter's Query method
5. **Auditing** — Records query logs (user, SQL, duration, row count)

#### Adapter Registry

Location: `internal/semantic/adapter/registry.go`

Manages all semantic engine instances:
- Supports `local` and `cubejs` adapter types
- Thread-safe concurrent access
- Supports setting a default engine

#### LocalAdapter (Local Engine)

Location: `internal/semantic/adapter/local_adapter.go`

Compiles semantic queries into SQL and executes them on the target database:
1. Loads Cube definitions from Registry into in-memory cache
2. Calls LocalCompiler to compile QueryRequest into SQL
3. Executes SQL via QueryService and returns results

#### LocalCompiler (SQL Compiler)

Location: `internal/semantic/adapter/local_compiler.go`

Core compilation logic:
- **SELECT**: Replaces Measure SQL templates (e.g., `SUM({field})`) with actual fields, wraps aggregate functions
- **FROM**: Uses Cube's `sql_table` or `sql_query`
- **JOIN**: Only joins Cubes referenced in the query
- **WHERE**: Converts Filter structures to SQL conditions (supports `equals`, `contains`, `gt`, `in`, etc.)
- **GROUP BY**: All non-aggregated dimensions
- **ORDER BY / LIMIT**: Passed through

---

## 4. MCP Tools Integration

### 4.1 Three Semantic Tools

DBridge exposes three semantic tools to external AI clients via MCP (Model Context Protocol):

| Tool Name | Function | Input | Output |
|--------|------|------|------|
| `semantic_list_engines` | List all semantic engines | None | Engine names + capability list |
| `semantic_list_cubes` | List all Cubes | `engine` (optional) | Cube metadata (name, measures, dimensions) |
| `semantic_query` | Execute semantic query | `engine`, `measures`, `dimensions`, `filters`, etc. | `columns`, `rows`, `sql`, `duration_ms` |

### 4.2 Tool Definitions

Location: `internal/mcp/tools/semantic.go`

```go
// semantic_query input structure
type SemanticQueryInput struct {
    Engine       string          `json:"engine"`
    DataSourceID string          `json:"data_source_id"`
    Measures     []string        `json:"measures"`      // ["orders.total_amount"]
    Dimensions   []string        `json:"dimensions"`    // ["orders.date"]
    Filters      []engine.Filter `json:"filters"`
    Order        []engine.OrderItem `json:"order"`
    Limit        int             `json:"limit"`
    Offset       int             `json:"offset"`
}

// semantic_query output structure
type SemanticQueryOutput struct {
    Columns   []string    `json:"columns"`
    Rows      []map[string]any `json:"rows"`
    TotalRows int         `json:"total_rows"`
    DurationMs int64      `json:"duration_ms"`
    SQL       string      `json:"sql"`
    CacheHit  bool        `json:"cache_hit"`
    Engine    string      `json:"engine"`
}
```

### 4.3 Tool Registration

Location: `internal/mcp/server.go` (lines 332-355)

```go
if svc.Semantic != nil {
    // Register semantic_list_engines
    mcp.AddTool(server, &mcp.Tool{
        Name:        "semantic_list_engines",
        Description: "List all registered semantic layer engines and their capability information",
        InputSchema: mcp.ToolInputSchema{Type: "object", Properties: map[string]any{}},
    }, handleListEngines)

    // Register semantic_list_cubes
    mcp.AddTool(server, &mcp.Tool{...}, handleListCubes)

    // Register semantic_query
    mcp.AddTool(server, &mcp.Tool{...}, handleSemanticQuery)
}
```

All semantic tools require `read` permission and belong to the `semantic` category.

---

## 5. Agent LLM Call Flow

### 5.1 Two Call Paths

#### Path A: Agent Built-in Tools (agentsdkwrap)

Location: `internal/service/agentsdkwrap/runner.go` (lines 506-601)

When an Agent executes, semantic tools are injected directly as built-in tools:

```go
// Default read-only tools include semantic discovery tools
DefaultReadTools = []string{
    "semantic_list_engines",
    "semantic_list_cubes",
    // ... other tools
}

// Semantic query tool implementation
semanticQueryTool := types.Tool{
    Name: "semantic_query",
    Exec: func(ctx context.Context, argsJSON string) (string, error) {
        var input SemanticQueryInput
        json.Unmarshal([]byte(argsJSON), &input)

        queryReq := &engine.QueryRequest{
            Measures:   input.Measures,
            Dimensions: input.Dimensions,
            Filters:    input.Filters,
        }

        result, err := semanticSvc.Query(ctx, queryReq, nil, "", "", "", "", "agent")
        return formatQueryResult(result), nil
    },
}
```

#### Path B: MCP Protocol (External Clients)

Location: `internal/mcp/server.go`

External MCP clients (e.g., Claude Desktop, Codex) connect via the MCP protocol and access the same three tools.

### 5.2 Complete Call Example

**Scenario**: User asks "Help me check last month's total order amount"

```
Step 1: LLM calls semantic_list_cubes
── Input: {}
├── Returns: [
│     { name: "orders", display_name: "Orders",
│       measures: ["total_amount", "order_count"],
│       dimensions: ["date", "region", "status"] },
│     { name: "users", ... }
│   ]
└── LLM understands: Order data is in the "orders" Cube

Step 2: LLM calls semantic_query
── Input: {
│     measures: ["orders.total_amount"],
│     dimensions: ["orders.date"],
│     filters: [{
│       field: "orders.date",
│       op: "gte",
│       value: "2026-09-01"
│     }, {
│       field: "orders.date",
│       op: "lt",
│       value: "2026-10-01"
│     }]
│   }
├── Internal flow:
│   ├── ValidateQueryRequest() — Validate field formats
│   ├── resolveEngine() — Select engine
│   ├── LocalCompiler.Compile() — Generate SQL:
│   │     SELECT SUM(amount) AS total_amount,
│   │            order_date AS date
│   │     FROM orders
│   │     WHERE order_date >= '2026-09-01'
│   │       AND order_date < '2026-10-01'
│   │     GROUP BY order_date
│   ├── querySvc.Execute() — Execute SQL
│   ── Returns: { columns, rows, sql, duration_ms }
└── LLM generates answer: "Last month's total order amount was $1,234,567"
```

### 5.3 ToolResolver

Location: `internal/service/agentexec/mcp_bridge.go`

During Agent execution, ToolResolver resolves tools in the following priority order:

1. **Built-in Go tools** — Matched from `agentsdkwrap.BuiltinTools` (semantic tools are here)
2. **MCP tool definitions** — Loaded from database `mcp_tool_definitions` table (custom SQL tools, etc.)
3. **Unresolved** — Logs a warning

Before each tool executes, `PermissionGuard.Check()` performs permission verification; after execution, `captureResult()` records the audit.

---

## 6. Data Model Details

### 6.1 CubeDefinition (In-Memory Structure)

Location: `internal/semantic/engine/metadata.go`

```go
type CubeDefinition struct {
    Name           string              `json:"name"`
    DisplayName    string              `json:"display_name"`
    Description    string              `json:"description"`
    DataSourceID   string              `json:"data_source_id"`
    Database       string              `json:"database,omitempty"`
    Schema         string              `json:"schema,omitempty"`
    SQLTable       string              `json:"sql_table,omitempty"`
    SQLQuery       string              `json:"sql_query,omitempty"`
    Measures       []*MeasureDetail    `json:"measures"`
    Dimensions     []*DimensionDetail  `json:"dimensions"`
    Joins          []*JoinDetail       `json:"joins,omitempty"`
    PreAggregations []*PreAggDetail    `json:"pre_aggregations,omitempty"`
    Version        int                 `json:"version"`
    UpstreamTables []string            `json:"upstream_tables,omitempty"`
}
```

### 6.2 MeasureDetail

```go
type MeasureDetail struct {
    MeasureMeta
    SQL         string   `json:"sql"`           // SQL expression template
    Filters     []Filter `json:"filters"`       // Measure-level filters
    DrillMembers []string `json:"drill_members"` // Drill-down members
    Metadata    map[string]any `json:"metadata"`
}

type MeasureMeta struct {
    Name        string `json:"name"`
    DisplayName string `json:"display_name"`
    Description string `json:"description"`
    Type        string `json:"type"`  // sum, avg, count, etc.
}
```

### 6.3 DimensionDetail

```go
type DimensionDetail struct {
    DimensionMeta
    SQL            string `json:"sql"`
    PrimaryKey     bool   `json:"primary_key"`
    TimeGranularity string `json:"time_granularity"` // day, week, month, etc.
    Metadata       map[string]any `json:"metadata"`
}
```

### 6.4 CubeMeta (Summary View)

Used for list display, contains only names and member lists:

```go
type CubeMeta struct {
    Name        string   `json:"name"`
    DisplayName string   `json:"display_name"`
    Description string   `json:"description"`
    Measures    []string `json:"measures"`    // Names only
    Dimensions  []string `json:"dimensions"`  // Names only
    DataSource  string   `json:"data_source"`
}
```

---

## 7. Query Validation Rules

Location: `internal/semantic/service/validation.go`

### 7.1 Field Format

All Measure/Dimension/Filter fields must follow the `cube_name.member_name` format:

```
✅ orders.total_amount
✅ users.created_date
❌ total_amount          // Missing Cube prefix
 orders.               // Missing member name
 .total_amount         // Missing Cube name
```

Regex: `^[a-zA-Z_][a-zA-Z0-9_]*\.[a-zA-Z_][a-zA-Z0-9_]*$`

### 7.2 Operator Whitelist

| Operator | Description | Example |
|--------|------|------|
| `equals` | Equals | `status equals "completed"` |
| `notEquals` | Not equals | `status notEquals "cancelled"` |
| `contains` | Contains | `name contains "John"` |
| `gt` / `gte` | Greater than / Greater than or equal to | `amount gt 100` |
| `lt` / `lte` | Less than / Less than or equal to | `date lte "2026-12-31"` |
| `in` | In list | `region in ["North", "South"]` |
| `set` / `notSet` | Is set / Is not set | `email set` |

### 7.3 Limit Upper Bound

Maximum return row count is limited to **10,000** to prevent memory overflow from large result sets.

---

## 8. Data Source Association

### 8.1 Data Source Configuration

Cubes are associated with specific database connections via `data_source_id`. Data source configurations are stored in the `data_sources` table, including:

- Database type (MySQL, PostgreSQL, SQL Server, etc.)
- Connection information (Host, Port, Database, Username, Password)
- Connection pool configuration

### 8.2 Query Routing

When executing a semantic query:

1. Retrieve `data_source_id` from Cube definition
2. Get connection configuration via `DataSourceService`
3. Send compiled SQL to target database for execution
4. Return result set

### 8.3 Multi-Data Source Support

Different Cubes can be associated with different data sources, enabling cross-database queries:

```
Cube "orders" → MySQL Database A
Cube "users"  → PostgreSQL Database B
```

Through Join definitions, Cubes from different data sources can be automatically associated during queries (engine support required).

---

## 9. Row-Level Data Security

### 9.1 Configuration Method

Configure row-level filters in `SemanticQueryService`:

```go
rowLevelFilters := []engine.RowLevelFilter{
    {
        CubeName: "orders",
        Field:    "region",
        Resolver: func(ctx context.Context, userRoles []string) []string {
            // Return allowed regions based on user roles
            return getUserRegions(ctx)
        },
    },
}
```

### 9.2 Execution Timing

Row-level filters are applied before query execution, automatically appended to WHERE conditions:

```sql
-- Original query
SELECT SUM(amount) FROM orders WHERE date >= '2026-09-01'

-- After applying row-level security
SELECT SUM(amount) FROM orders
WHERE date >= '2026-09-01'
  AND region IN ('East', 'North')  -- Automatically appended
```

---

## 10. Audit Logs

Every semantic query records an audit log, containing:

| Field | Description |
|------|------|
| `user_id` | User ID who executed the query |
| `cube_name` | Queried Cube name |
| `sql` | Generated SQL statement |
| `duration_ms` | Query duration (milliseconds) |
| `row_count` | Number of returned rows |
| `status` | Execution status (success/error) |
| `error_message` | Error message (if any) |
| `created_at` | Execution time |

Audit logs are stored in the `audit_logs` table with module identifier `semantic`.

---

## 11. Frontend Integration

### 11.1 Cube Management Interface

Location: `web/src/pages/semantic/CubeEdit.tsx`

Provides Cube creation, editing, and viewing capabilities:
- Basic information configuration (name, description, data source, table name)
- Measure definitions (name, type, SQL expression)
- Dimension definitions (name, SQL expression, time granularity)
- Join configuration (associated Cube, join condition, join type)

### 11.2 AI-Generated Cubes

Location: `web/src/pages/semantic/CubeEdit.tsx` (lines 253-278)

Users can describe requirements in natural language, and the AI Agent automatically generates Cube configurations:

```
User input: "Create an order analysis model with metrics like total order amount and order count, grouped by date and region"
    ↓
AI calls LLM to generate Cube JSON
    ↓
Frontend parses JSON and populates form
    ↓
User confirms and saves
```

### 11.3 Semantic Query API

The frontend executes semantic queries via REST API:

```typescript
// List Cubes
GET /api/v1/semantic/cubes

// Execute query
POST /api/v1/semantic/query
{
  "engine": "local",
  "measures": ["orders.total_amount"],
  "dimensions": ["orders.date"],
  "filters": [...],
  "limit": 100
}
```

---

## 12. Configuration Examples

### 12.1 Configuration File

Location: `configs/config.yaml`

```yaml
semantic:
  # Default adapter
  default_adapter: "local"

  # Cube.js adapter (optional)
  cubejs:
    api_url: "http://localhost:4000"
    api_token: "${CUBEJS_API_TOKEN}"
```

### 12.2 Environment Variables

| Configuration | Environment Variable |
|--------|----------|
| `semantic.default_adapter` | `DBRIDGE_SEMANTIC_DEFAULT_ADAPTER` |
| `semantic.cubejs.api_url` | `DBRIDGE_SEMANTIC_CUBEJS_API_URL` |
| `semantic.cubejs.api_token` | `DBRIDGE_SEMANTIC_CUBEJS_API_TOKEN` |

---

## 13. Key Code Location Index

| Component | File Path | Description |
|------|----------|------|
| **Semantic Query Service** | `internal/semantic/service/query_service.go` | Query orchestration core |
| **Query Validation** | `internal/semantic/service/validation.go` | Field format, operator validation |
| **Adapter Registry** | `internal/semantic/adapter/registry.go` | Engine management |
| **Local Adapter** | `internal/semantic/adapter/local_adapter.go` | SQL compilation and execution |
| **SQL Compiler** | `internal/semantic/adapter/local_compiler.go` | Semantic → SQL conversion |
| **Local Registry** | `internal/semantic/adapter/local_registry.go` | Cube cache management |
| **Engine Metadata** | `internal/semantic/engine/metadata.go` | Cube/Measure/Dimension definitions |
| **MCP Tool Definitions** | `internal/mcp/tools/semantic.go` | Three semantic tool implementations |
| **MCP Service Registration** | `internal/mcp/server.go` | Tool registration to MCP Server |
| **Agent Built-in Tools** | `internal/service/agentsdkwrap/runner.go` | Agent direct call path |
| **Tool Resolver** | `internal/service/agentexec/mcp_bridge.go` | Tool matching and permission verification |
| **Database Models** | `internal/repository/models.go` (L875-911) | SemanticCube table structure |
| **Frontend Cube Editor** | `web/src/pages/semantic/CubeEdit.tsx` | Cube management interface |
| **Frontend API Calls** | `web/src/api/semantic.ts` | Semantic layer REST API wrapper |

---

## 14. Best Practices

### 14.1 Cube Design Principles

1. **Single Responsibility**: Each Cube represents a clear business subject
2. **Naming Convention**: Use lowercase English + underscores (e.g., `order_items`)
3. **Measure Naming**: Use business-friendly names (e.g., `total_revenue` instead of `sum_amount`)
4. **Dimension Granularity**: Explicitly define granularity for time dimensions (day/week/month/quarter/year)

### 14.2 Performance Optimization

1. **Pre-aggregation**: Configure PreAggregation for high-frequency queries
2. **Limit Restriction**: Always set a reasonable Limit to avoid full table scans
3. **Index Optimization**: Create database indexes on frequently filtered fields
4. **Caching Strategy**: Enable query caching for repeated queries

### 14.3 Security Recommendations

1. **Row-Level Security**: Configure row-level filters for sensitive data
2. **Permission Control**: Restrict accessible Cubes via MCP Token
3. **Audit Logs**: Regularly review query logs to detect abnormal access
4. **SQL Injection Prevention**: Use parameterized queries for all parameters; prohibit SQL concatenation

---

## 15. Troubleshooting

### 15.1 Common Issues

| Issue | Possible Cause | Solution |
|------|----------|----------|
| Query returns empty results | Cube not associated with data source | Check `data_source_id` configuration |
| SQL syntax error | Measure SQL template error | Check `{field}` placeholders are correct |
| Insufficient permissions | MCP Token lacks read permission | Check Token permission configuration |
| Query timeout | Data volume too large or no indexes | Add Limit or optimize database indexes |
| Cube not loaded | Registry not initialized | Check `semantic.default_adapter` configuration |

### 15.2 Debugging Methods

1. **Check audit logs**: `SELECT * FROM audit_logs WHERE module = 'semantic' ORDER BY created_at DESC LIMIT 20`
2. **Check generated SQL**: The `sql` field returned by `semantic_query` contains the actual executed SQL
3. **Validate Cube definitions**: Use `semantic_list_cubes` to verify Cube metadata is correct
4. **Test data source connections**: Test connection on the data source management page

---

## Appendix A: Complete Query Examples

### A.1 Simple Aggregation Query

**Requirement**: Query total order amount

```json
{
  "measures": ["orders.total_amount"],
  "limit": 1
}
```

**Generated SQL**:
```sql
SELECT SUM(amount) AS total_amount
FROM orders
LIMIT 1
```

### A.2 With Dimension Grouping

**Requirement**: Query total order amount by region

```json
{
  "measures": ["orders.total_amount"],
  "dimensions": ["orders.region"],
  "order": [{"field": "orders.total_amount", "direction": "desc"}],
  "limit": 10
}
```

**Generated SQL**:
```sql
SELECT SUM(amount) AS total_amount,
       region_name AS region
FROM orders
GROUP BY region_name
ORDER BY total_amount DESC
LIMIT 10
```

### A.3 With Time Filter

**Requirement**: Query order count for Q3 2026

```json
{
  "measures": ["orders.order_count"],
  "dimensions": ["orders.date"],
  "filters": [
    {"field": "orders.date", "op": "gte", "value": "2026-07-01"},
    {"field": "orders.date", "op": "lt", "value": "2026-10-01"}
  ]
}
```

**Generated SQL**:
```sql
SELECT COUNT(id) AS order_count,
       order_date AS date
FROM orders
WHERE order_date >= '2026-07-01'
  AND order_date < '2026-10-01'
GROUP BY order_date
```

### A.4 Cross-Cube Join Query

**Requirement**: Query total order amount per user

```json
{
  "measures": ["orders.total_amount"],
  "dimensions": ["users.name"],
  "filters": []
}
```

**Generated SQL** (assuming Join is configured):
```sql
SELECT SUM(o.amount) AS total_amount,
       u.name AS name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
GROUP BY u.name
```

---

## Appendix B: MCP Tool Call Examples

### B.1 Claude Desktop Configuration

```json
{
  "mcpServers": {
    "dbridge": {
      "url": "http://localhost:8003/mcp",
      "headers": {
        "Authorization": "Bearer db_mcp_xxxxxxxxxxxx"
      }
    }
  }
}
```

### B.2 Natural Language Query

Enter directly in Claude Desktop:

```
Analyze last month's sales data, grouped by product category, showing only the top 10 categories by sales amount
```

Claude will automatically:
1. Call `semantic_list_cubes` to find sales-related Cubes
2. Call `semantic_query` to execute the query
3. Summarize results in natural language

---

**Document Version**: 1.0
**Last Updated**: 2026-10-19
**Maintainer**: DBridge Development Team
