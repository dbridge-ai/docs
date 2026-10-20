# DBridge 语义引擎与语义模型

## 1. 概述

DBridge 语义层（Semantic Layer）是连接业务用户与底层数据库的桥梁。它将物理数据库表抽象为业务友好的 **Cube（数据模型）**，通过 **Measures（度量）** 和 **Dimensions（维度）** 定义业务指标体系，使非技术人员也能通过自然语言或简单配置完成复杂数据分析。

语义层的核心价值：
- **业务语义封装**：将 SQL 表达式、聚合逻辑、表关联封装为可复用的业务指标
- **统一查询接口**：无论底层是 MySQL、PostgreSQL 还是其他数据库，上层使用统一的查询语法
- **AI 友好**：为大语言模型（LLM）提供结构化的元数据，使 AI 能准确理解业务数据并生成正确查询

---

## 2. 核心概念

### 2.1 Cube（数据模型）

Cube 是语义层的基本单元，代表一个业务主题域（如"订单"、"用户"、"产品"）。

**数据库存储**：`semantic_cubes` 表

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | UUID | 主键 |
| `name` | string | 唯一标识，如 `orders` |
| `display_name` | string | 显示名称，如 `订单` |
| `description` | text | 业务描述 |
| `data_source_id` | string | 关联的数据源 ID |
| `database` | string | 数据库名（可选） |
| `schema_name` | string | Schema 名（可选） |
| `sql_table` | string | 物理表名，如 `public.orders` |
| `sql_query` | text | 自定义 SQL 查询（与 sql_table 二选一） |
| `measures_json` | JSON | 度量定义数组 |
| `dimensions_json` | JSON | 维度定义数组 |
| `joins_json` | JSON | 关联定义数组 |
| `pre_aggs_json` | JSON | 预聚合定义数组 |
| `version` | int | 版本号，每次更新递增 |
| `tenant_id` | string | 租户 ID（多租户隔离） |

### 2.2 Measure（度量）

度量是可聚合的业务指标，如"订单总额"、"用户数"。

```json
{
  "name": "total_amount",
  "display_name": "订单总额",
  "description": "所有订单的金额总和",
  "type": "sum",
  "sql": "SUM({field})",
  "field": "amount"
}
```

**支持的聚合类型**：
- `sum` — 求和
- `avg` — 平均值
- `min` / `max` — 最小/最大值
- `count` — 计数
- `count_distinct` — 去重计数
- `number` — 数值（不聚合）
- `running_total` — 累计值

### 2.3 Dimension（维度）

维度是数据的分组或过滤属性，如"日期"、"地区"、"状态"。

```json
{
  "name": "date",
  "display_name": "订单日期",
  "description": "订单创建日期",
  "sql": "order_date",
  "time_granularity": "day"
}
```

**时间维度粒度**：`day`、`week`、`month`、`quarter`、`year`

### 2.4 Join（关联）

Cube 之间可以通过 Join 关联，实现跨模型查询。

```json
{
  "join_cube_name": "users",
  "relationship": "many_to_one",
  "sql": "orders.user_id = users.id",
  "join_type": "left"
}
```

---

## 3. 语义引擎架构

### 3.1 整体架构

```
┌─────────────────────────────────────────────────────────┐
│                    接入层                                 │
│  REST API  │  MCP Server  │  Agent SDK  │  前端 UI       │
└────────────┴──────────────┴─────────────┴────────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │  SemanticQueryService│  ← 查询编排服务
              ─────────────────────┘
                         │
           ┌─────────────┼─────────────┐
           ▼             ▼             ▼
     验证服务       审计日志       行级安全
           │
           ▼
    ┌──────────────┐
    │  Adapter Registry │  ← 适配器注册中心
    └──────────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
LocalAdapter  CubeJSAdapter  ← 可插拔引擎
    │
    ▼
┌──────────────┐
│ LocalCompiler │  ← SQL 编译器
└──────────────┘
    │
    ▼
──────────────┐
│  QueryService │  ← 底层 SQL 执行
└──────────────┘
```

### 3.2 核心组件

#### SemanticQueryService（查询编排）

位置：`internal/semantic/service/query_service.go`

负责完整的查询生命周期：
1. **验证** — 校验字段格式、运算符白名单、Limit 上限
2. **路由** — 根据引擎名称选择适配器
3. **行级安全** — 应用数据权限过滤
4. **执行** — 调用适配器的 Query 方法
5. **审计** — 记录查询日志（用户、SQL、耗时、行数）

#### Adapter Registry（适配器注册中心）

位置：`internal/semantic/adapter/registry.go`

管理所有语义引擎实例：
- 支持 `local` 和 `cubejs` 两种适配器类型
- 线程安全的并发访问
- 支持设置默认引擎

#### LocalAdapter（本地引擎）

位置：`internal/semantic/adapter/local_adapter.go`

将语义查询编译为 SQL 并在目标数据库执行：
1. 从 Registry 加载 Cube 定义到内存缓存
2. 调用 LocalCompiler 将 QueryRequest 编译为 SQL
3. 通过 QueryService 执行 SQL 并返回结果

#### LocalCompiler（SQL 编译器）

位置：`internal/semantic/adapter/local_compiler.go`

核心编译逻辑：
- **SELECT**：将 Measure 的 SQL 模板（如 `SUM({field})`）替换为实际字段，包裹聚合函数
- **FROM**：使用 Cube 的 `sql_table` 或 `sql_query`
- **JOIN**：仅关联查询中引用的 Cube
- **WHERE**：将 Filter 结构转换为 SQL 条件（支持 `equals`、`contains`、`gt`、`in` 等）
- **GROUP BY**：所有非聚合维度
- **ORDER BY / LIMIT**：透传

---

## 4. MCP Tools 集成

### 4.1 三个语义工具

DBridge 通过 MCP（Model Context Protocol）向外部 AI 客户端暴露三个语义工具：

| 工具名 | 功能 | 输入 | 输出 |
|--------|------|------|------|
| `semantic_list_engines` | 列出所有语义引擎 | 无 | 引擎名称 + 能力列表 |
| `semantic_list_cubes` | 列出所有 Cube | `engine`（可选） | Cube 元数据（名称、度量、维度） |
| `semantic_query` | 执行语义查询 | `engine`、`measures`、`dimensions`、`filters` 等 | `columns`、`rows`、`sql`、`duration_ms` |

### 4.2 工具定义

位置：`internal/mcp/tools/semantic.go`

```go
// semantic_query 输入结构
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

// semantic_query 输出结构
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

### 4.3 工具注册

位置：`internal/mcp/server.go` (lines 332-355)

```go
if svc.Semantic != nil {
    // 注册 semantic_list_engines
    mcp.AddTool(server, &mcp.Tool{
        Name:        "semantic_list_engines",
        Description: "列出所有已注册的语义层引擎及其能力信息",
        InputSchema: mcp.ToolInputSchema{Type: "object", Properties: map[string]any{}},
    }, handleListEngines)

    // 注册 semantic_list_cubes
    mcp.AddTool(server, &mcp.Tool{...}, handleListCubes)

    // 注册 semantic_query
    mcp.AddTool(server, &mcp.Tool{...}, handleSemanticQuery)
}
```

所有语义工具的权限要求为 `read`，类别为 `semantic`。

---

## 5. Agent LLM 调用流程

### 5.1 两条调用路径

#### 路径 A：Agent 内置工具（agentsdkwrap）

位置：`internal/service/agentsdkwrap/runner.go` (lines 506-601)

当 Agent 执行时，语义工具作为内置工具直接注入：

```go
// 默认只读工具包含语义发现工具
DefaultReadTools = []string{
    "semantic_list_engines",
    "semantic_list_cubes",
    // ... 其他工具
}

// 语义查询工具实现
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

#### 路径 B：MCP 协议（外部客户端）

位置：`internal/mcp/server.go`

外部 MCP 客户端（如 Claude Desktop、Codex）通过 MCP 协议连接，获取相同的三个工具。

### 5.2 完整调用示例

**场景**：用户问"帮我查一下上个月的订单总额"

```
步骤 1: LLM 调用 semantic_list_cubes
── 输入: {}
├── 返回: [
│     { name: "orders", display_name: "订单",
│       measures: ["total_amount", "order_count"],
│       dimensions: ["date", "region", "status"] },
│     { name: "users", ... }
│   ]
└── LLM 理解: 订单数据在 "orders" Cube 中

步骤 2: LLM 调用 semantic_query
── 输入: {
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
├── 内部流程:
│   ├── ValidateQueryRequest() — 验证字段格式
│   ├── resolveEngine() — 选择引擎
│   ├── LocalCompiler.Compile() — 生成 SQL:
│   │     SELECT SUM(amount) AS total_amount,
│   │            order_date AS date
│   │     FROM orders
│   │     WHERE order_date >= '2026-09-01'
│   │       AND order_date < '2026-10-01'
│   │     GROUP BY order_date
│   ├── querySvc.Execute() — 执行 SQL
│   ── 返回: { columns, rows, sql, duration_ms }
└── LLM 生成回答: "上个月订单总额为 ¥1,234,567"
```

### 5.3 ToolResolver 工具解析

位置：`internal/service/agentexec/mcp_bridge.go`

Agent 执行时，ToolResolver 按以下优先级解析工具：

1. **内置 Go 工具** — 从 `agentsdkwrap.BuiltinTools` 匹配（语义工具在此）
2. **MCP 工具定义** — 从数据库 `mcp_tool_definitions` 表加载（自定义 SQL 工具等）
3. **未解析** — 记录警告日志

每个工具执行前经过 `PermissionGuard.Check()` 权限校验，执行后通过 `captureResult()` 记录审计。

---

## 6. 数据模型详解

### 6.1 CubeDefinition（内存结构）

位置：`internal/semantic/engine/metadata.go`

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
    SQL         string   `json:"sql"`           // SQL 表达式模板
    Filters     []Filter `json:"filters"`       // 度量级过滤
    DrillMembers []string `json:"drill_members"` // 下钻成员
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

### 6.4 CubeMeta（摘要视图）

用于列表展示，仅包含名称和成员列表：

```go
type CubeMeta struct {
    Name        string   `json:"name"`
    DisplayName string   `json:"display_name"`
    Description string   `json:"description"`
    Measures    []string `json:"measures"`    // 仅名称
    Dimensions  []string `json:"dimensions"`  // 仅名称
    DataSource  string   `json:"data_source"`
}
```

---

## 7. 查询验证规则

位置：`internal/semantic/service/validation.go`

### 7.1 字段格式

所有 Measure/Dimension/Filter 字段必须遵循 `cube_name.member_name` 格式：

```
✅ orders.total_amount
✅ users.created_date
❌ total_amount          // 缺少 Cube 前缀
 orders.               // 缺少成员名
 .total_amount         // 缺少 Cube 名
```

正则：`^[a-zA-Z_][a-zA-Z0-9_]*\.[a-zA-Z_][a-zA-Z0-9_]*$`

### 7.2 运算符白名单

| 运算符 | 说明 | 示例 |
|--------|------|------|
| `equals` | 等于 | `status equals "completed"` |
| `notEquals` | 不等于 | `status notEquals "cancelled"` |
| `contains` | 包含 | `name contains "张"` |
| `gt` / `gte` | 大于 / 大于等于 | `amount gt 100` |
| `lt` / `lte` | 小于 / 小于等于 | `date lte "2026-12-31"` |
| `in` | 在列表中 | `region in ["北京", "上海"]` |
| `set` / `notSet` | 已设置 / 未设置 | `email set` |

### 7.3 Limit 上限

最大返回行数限制为 **10,000**，防止大结果集导致内存溢出。

---

## 8. 与数据源的关联

### 8.1 数据源配置

Cube 通过 `data_source_id` 关联到具体的数据库连接。数据源配置存储在 `data_sources` 表中，包含：

- 数据库类型（MySQL、PostgreSQL、SQL Server 等）
- 连接信息（Host、Port、Database、Username、Password）
- 连接池配置

### 8.2 查询路由

当执行语义查询时：

1. 从 Cube 定义中获取 `data_source_id`
2. 通过 `DataSourceService` 获取连接配置
3. 将编译后的 SQL 发送到目标数据库执行
4. 返回结果集

### 8.3 多数据源支持

不同 Cube 可以关联不同的数据源，实现跨库查询：

```
Cube "orders" → MySQL 数据库 A
Cube "users"  → PostgreSQL 数据库 B
```

通过 Join 定义，可以在查询时自动关联不同数据源的 Cube（需引擎支持）。

---

## 9. 行级数据安全

### 9.1 配置方式

在 `SemanticQueryService` 中配置行级过滤器：

```go
rowLevelFilters := []engine.RowLevelFilter{
    {
        CubeName: "orders",
        Field:    "region",
        Resolver: func(ctx context.Context, userRoles []string) []string {
            // 根据用户角色返回允许访问的地区
            return getUserRegions(ctx)
        },
    },
}
```

### 9.2 执行时机

行级过滤器在查询执行前应用，自动追加到 WHERE 条件：

```sql
-- 原始查询
SELECT SUM(amount) FROM orders WHERE date >= '2026-09-01'

-- 应用行级安全后
SELECT SUM(amount) FROM orders
WHERE date >= '2026-09-01'
  AND region IN ('华东', '华北')  -- 自动追加
```

---

## 10. 审计日志

每次语义查询都会记录审计日志，包含：

| 字段 | 说明 |
|------|------|
| `user_id` | 执行查询的用户 ID |
| `cube_name` | 查询的 Cube 名称 |
| `sql` | 生成的 SQL 语句 |
| `duration_ms` | 查询耗时（毫秒） |
| `row_count` | 返回行数 |
| `status` | 执行状态（success/error） |
| `error_message` | 错误信息（如有） |
| `created_at` | 执行时间 |

审计日志存储在 `audit_logs` 表中，模块标识为 `semantic`。

---

## 11. 前端集成

### 11.1 Cube 管理界面

位置：`web/src/pages/semantic/CubeEdit.tsx`

提供 Cube 的创建、编辑、查看功能：
- 基本信息配置（名称、描述、数据源、表名）
- 度量定义（名称、类型、SQL 表达式）
- 维度定义（名称、SQL 表达式、时间粒度）
- Join 配置（关联 Cube、关联条件、关联类型）

### 11.2 AI 生成立方体

位置：`web/src/pages/semantic/CubeEdit.tsx` (lines 253-278)

用户可以用自然语言描述需求，AI Agent 自动生成 Cube 配置：

```
用户输入: "创建一个订单分析模型，包含订单总额、订单数量等指标，按日期和地区分组"
    ↓
AI 调用 LLM 生成 Cube JSON
    ↓
前端解析 JSON 并填充表单
    ↓
用户确认后保存
```

### 11.3 语义查询 API

前端通过 REST API 执行语义查询：

```typescript
// 列出 Cube
GET /api/v1/semantic/cubes

// 执行查询
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

## 12. 配置示例

### 12.1 配置文件

位置：`configs/config.yaml`

```yaml
semantic:
  # 默认适配器
  default_adapter: "local"

  # Cube.js 适配器（可选）
  cubejs:
    api_url: "http://localhost:4000"
    api_token: "${CUBEJS_API_TOKEN}"
```

### 12.2 环境变量

| 配置项 | 环境变量 |
|--------|----------|
| `semantic.default_adapter` | `DBRIDGE_SEMANTIC_DEFAULT_ADAPTER` |
| `semantic.cubejs.api_url` | `DBRIDGE_SEMANTIC_CUBEJS_API_URL` |
| `semantic.cubejs.api_token` | `DBRIDGE_SEMANTIC_CUBEJS_API_TOKEN` |

---

## 13. 关键代码位置索引

| 组件 | 文件路径 | 说明 |
|------|----------|------|
| **语义查询服务** | `internal/semantic/service/query_service.go` | 查询编排核心 |
| **查询验证** | `internal/semantic/service/validation.go` | 字段格式、运算符校验 |
| **适配器注册中心** | `internal/semantic/adapter/registry.go` | 引擎管理 |
| **本地适配器** | `internal/semantic/adapter/local_adapter.go` | SQL 编译与执行 |
| **SQL 编译器** | `internal/semantic/adapter/local_compiler.go` | 语义→SQL 转换 |
| **本地注册表** | `internal/semantic/adapter/local_registry.go` | Cube 缓存管理 |
| **引擎元数据** | `internal/semantic/engine/metadata.go` | Cube/Measure/Dimension 定义 |
| **MCP 工具定义** | `internal/mcp/tools/semantic.go` | 三个语义工具实现 |
| **MCP 服务注册** | `internal/mcp/server.go` | 工具注册到 MCP Server |
| **Agent 内置工具** | `internal/service/agentsdkwrap/runner.go` | Agent 直接调用路径 |
| **工具解析器** | `internal/service/agentexec/mcp_bridge.go` | 工具匹配与权限校验 |
| **数据库模型** | `internal/repository/models.go` (L875-911) | SemanticCube 表结构 |
| **前端 Cube 编辑** | `web/src/pages/semantic/CubeEdit.tsx` | Cube 管理界面 |
| **前端 API 调用** | `web/src/api/semantic.ts` | 语义层 REST API 封装 |

---

## 14. 最佳实践

### 14.1 Cube 设计原则

1. **单一职责**：每个 Cube 代表一个明确的业务主题
2. **命名规范**：使用英文小写 + 下划线（如 `order_items`）
3. **度量命名**：使用业务友好的名称（如 `total_revenue` 而非 `sum_amount`）
4. **维度粒度**：时间维度明确粒度（日/周/月/季/年）

### 14.2 性能优化

1. **预聚合**：对高频查询配置 PreAggregation
2. **Limit 限制**：始终设置合理的 Limit，避免全表扫描
3. **索引优化**：在常用过滤字段上建立数据库索引
4. **缓存策略**：对重复查询启用查询缓存

### 14.3 安全建议

1. **行级安全**：为敏感数据配置行级过滤器
2. **权限控制**：通过 MCP Token 限制可访问的 Cube
3. **审计日志**：定期检查查询日志，发现异常访问
4. **SQL 注入防护**：所有参数使用参数化查询，禁止拼接 SQL

---

## 15. 故障排查

### 15.1 常见问题

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 查询返回空结果 | Cube 未关联数据源 | 检查 `data_source_id` 配置 |
| SQL 语法错误 | Measure SQL 模板错误 | 检查 `{field}` 占位符是否正确 |
| 权限不足 | MCP Token 缺少 read 权限 | 检查 Token 权限配置 |
| 查询超时 | 数据量过大或无索引 | 添加 Limit 或优化数据库索引 |
| Cube 未加载 | Registry 未初始化 | 检查 `semantic.default_adapter` 配置 |

### 15.2 调试方法

1. **查看审计日志**：`SELECT * FROM audit_logs WHERE module = 'semantic' ORDER BY created_at DESC LIMIT 20`
2. **检查生成的 SQL**：`semantic_query` 返回的 `sql` 字段包含实际执行的 SQL
3. **验证 Cube 定义**：通过 `semantic_list_cubes` 查看 Cube 元数据是否正确
4. **测试数据源连接**：在数据源管理页面测试连接是否正常

---

## 附录 A：完整查询示例

### A.1 简单聚合查询

**需求**：查询订单总额

```json
{
  "measures": ["orders.total_amount"],
  "limit": 1
}
```

**生成 SQL**：
```sql
SELECT SUM(amount) AS total_amount
FROM orders
LIMIT 1
```

### A.2 带维度分组

**需求**：按地区统计订单总额

```json
{
  "measures": ["orders.total_amount"],
  "dimensions": ["orders.region"],
  "order": [{"field": "orders.total_amount", "direction": "desc"}],
  "limit": 10
}
```

**生成 SQL**：
```sql
SELECT SUM(amount) AS total_amount,
       region_name AS region
FROM orders
GROUP BY region_name
ORDER BY total_amount DESC
LIMIT 10
```

### A.3 带时间过滤

**需求**：查询 2026 年 Q3 的订单数量

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

**生成 SQL**：
```sql
SELECT COUNT(id) AS order_count,
       order_date AS date
FROM orders
WHERE order_date >= '2026-07-01'
  AND order_date < '2026-10-01'
GROUP BY order_date
```

### A.4 跨 Cube 关联查询

**需求**：查询每个用户的订单总额

```json
{
  "measures": ["orders.total_amount"],
  "dimensions": ["users.name"],
  "filters": []
}
```

**生成 SQL**（假设已配置 Join）：
```sql
SELECT SUM(o.amount) AS total_amount,
       u.name AS name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
GROUP BY u.name
```

---

## 附录 B：MCP Tool 调用示例

### B.1 Claude Desktop 配置

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

### B.2 自然语言查询

在 Claude Desktop 中直接输入：

```
帮我分析一下上个月的销售数据，按产品类别分组，只看销售额前 10 的类别
```

Claude 会自动：
1. 调用 `semantic_list_cubes` 找到销售相关 Cube
2. 调用 `semantic_query` 执行查询
3. 用自然语言总结结果

---

**文档版本**: 1.0  
**最后更新**: 2026-10-19  
**维护者**: DBridge 开发团队
