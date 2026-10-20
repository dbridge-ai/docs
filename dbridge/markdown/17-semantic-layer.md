# 语义层

语义层（Semantic Layer）是连接业务用户与底层数据库的桥梁。它将物理数据库表抽象为业务友好的 Cube（数据模型），通过度量（Measure）和维度（Dimension）定义业务指标体系。

> 本章介绍语义层的前端使用方式。语义引擎的详细技术文档请参考 [语义引擎指南](semantic-engine-guide.md)。

## 功能概述

- Cube 数据模型管理
- 度量（Measure）定义
- 维度（Dimension）定义
- 模型关联（Join）
- 指标查询
- AI 辅助生成 Cube
- 引擎管理

## 核心概念

### Cube（数据模型）

Cube 是语义层的基本单元，代表一个业务主题域（如"订单"、"用户"、"产品"）。

每个 Cube 包含：
- 物理表映射 — 对应数据库中的表或自定义 SQL 查询
- 度量定义 — 可聚合的业务指标（如订单总额、用户数）
- 维度定义 — 分组/过滤属性（如日期、地区、状态）
- 关联定义 — 与其他 Cube 的 JOIN 关系

### Measure（度量）

度量是可聚合的业务指标：

| 聚合类型 | 说明 |
|:---------|:-----|
| sum | 求和 |
| avg | 平均值 |
| min / max | 最小/最大值 |
| count | 计数 |
| count_distinct | 去重计数 |
| number | 数值（不聚合） |
| running_total | 累计值 |

### Dimension（维度）

维度是数据的分组或过滤属性：

| 属性 | 说明 |
|:-----|:-----|
| 名称 | 维度标识 |
| 显示名 | 业务名称 |
| SQL 表达式 | 对应的字段或表达式 |
| 时间粒度 | day / week / month / quarter / year |

## Cube 管理

### Cube 列表

进入「语义层 → Cube 管理」页面，展示所有 Cube：

| 列 | 说明 |
|:---|:-----|
| 名称 | Cube 标识名 |
| 显示名 | 业务显示名称 |
| 数据源 | 关联的数据源 |
| 物理表 | 映射的数据库表 |
| 版本 | 当前版本号 |

### 创建 Cube

1. 点击「新建」按钮
2. 填写 Cube 信息：

| 字段 | 说明 |
|:-----|:-----|
| 名称 | 唯一标识（如 `orders`） |
| 显示名 | 业务名称（如 `订单`） |
| 描述 | 业务描述 |
| 数据源 | 选择数据源 |
| 数据库 | 选择数据库 |
| 物理表 | 选择表名 |
| 或自定义 SQL | 编写自定义查询 |

3. 添加度量（Measures）
4. 添加维度（Dimensions）
5. 配置关联（Joins）
6. 保存

### AI 生成 Cube

系统支持通过 AI 自动生成 Cube 配置：

1. 点击「AI 生成」按钮
2. 输入描述，如："创建一个订单管理的 Cube，包含订单金额、数量等指标"
3. AI 自动生成度量、维度定义

## 度量配置

在 Cube 编辑页面添加度量：

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

## 维度配置

在 Cube 编辑页面添加维度：

```json
{
  "name": "order_date",
  "display_name": "订单日期",
  "sql": "order_date",
  "time_granularity": "day"
}
```

## 关联配置

定义 Cube 之间的 JOIN 关系：

```json
{
  "name": "orders_to_users",
  "from": "orders",
  "to": "users",
  "type": "many_to_one",
  "on": "orders.user_id = users.id"
}
```

| 关联类型 | 说明 |
|:---------|:-----|
| many_to_one | 多对一（如多个订单属于一个用户） |
| one_to_many | 一对多 |
| many_to_many | 多对多 |

## 指标查询

配置好 Cube 后，可以通过语义层 API 进行指标查询：

```json
{
  "cubes": ["orders"],
  "measures": ["orders.total_amount"],
  "dimensions": ["orders.order_date"],
  "filters": [
    {"member": "orders.order_date", "operator": "in_date_range", "values": ["last 30 days"]}
  ],
  "order": [["orders.order_date", "asc"]]
}
```

系统自动将语义查询翻译为 SQL 并执行。

## 引擎管理

进入「语义层 → 引擎管理」页面，查看语义层引擎状态：

| 信息 | 说明 |
|:-----|:-----|
| 适配器类型 | local / cubejs |
| 健康状态 | 引擎是否正常运行 |
| Cube 数量 | 已配置的 Cube 数 |

### 适配器

| 适配器 | 说明 |
|:-------|:-----|
| local | 内置引擎，直接查询数据库 |
| cubejs | 对接外部 CubeJS 服务 |

配置在 `configs/config.yaml` 中：

```yaml
semantic:
  default_adapter: local
  cubejs:
    api_url: ""
    api_token: ""
```

## 使用场景

### 业务指标平台

将复杂的 SQL 聚合逻辑封装为 Cube，业务人员通过简单的指标查询获取数据。

### AI 数据助手

为 AI Agent 提供结构化的业务元数据，使 AI 能准确理解业务数据并生成正确的查询。

### 统一指标口径

确保不同报表、不同人查询同一指标时，使用相同的计算逻辑。
