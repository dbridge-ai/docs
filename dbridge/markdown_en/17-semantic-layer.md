# Semantic Layer

The Semantic Layer is a bridge between business users and the underlying database. It abstracts physical database tables into business-friendly Cubes (data models), defining business metric systems through Measures and Dimensions.

> This chapter covers the frontend usage of the Semantic Layer. For detailed technical documentation on the semantic engine, refer to the [Semantic Engine Guide](semantic-engine-guide.md).

## Features

- Cube data model management
- Measure definitions
- Dimension definitions
- Model relationships (Joins)
- Metric queries
- AI-assisted Cube generation
- Engine management

## Core Concepts

### Cube (Data Model)

A Cube is the basic unit of the semantic layer, representing a business domain (e.g., "Orders", "Users", "Products").

Each Cube contains:
- Physical table mapping — Corresponds to a table in the database or a custom SQL query
- Measure definitions — Aggregatable business metrics (e.g., total order amount, user count)
- Dimension definitions — Grouping/filtering attributes (e.g., date, region, status)
- Join definitions — JOIN relationships with other Cubes

### Measure

Measures are aggregatable business metrics:

| Aggregation Type | Description |
|:----------------|:------------|
| sum | Sum |
| avg | Average |
| min / max | Minimum / Maximum |
| count | Count |
| count_distinct | Distinct count |
| number | Number (no aggregation) |
| running_total | Running total |

### Dimension

Dimensions are grouping or filtering attributes of data:

| Attribute | Description |
|:----------|:------------|
| Name | Dimension identifier |
| Display Name | Business name |
| SQL Expression | Corresponding field or expression |
| Time Granularity | day / week / month / quarter / year |

## Cube Management

### Cube List

Go to "Semantic Layer → Cube Management" to view all Cubes:

| Column | Description |
|:-------|:------------|
| Name | Cube identifier |
| Display Name | Business display name |
| Data Source | Associated data source |
| Physical Table | Mapped database table |
| Version | Current version number |

### Creating a Cube

1. Click the "New" button
2. Fill in the Cube information:

| Field | Description |
|:------|:------------|
| Name | Unique identifier (e.g., `orders`) |
| Display Name | Business name (e.g., `Orders`) |
| Description | Business description |
| Data Source | Select data source |
| Database | Select database |
| Physical Table | Select table name |
| Or Custom SQL | Write a custom query |

3. Add Measures
4. Add Dimensions
5. Configure Joins
6. Save

### AI-Generated Cubes

The system supports automatically generating Cube configurations via AI:

1. Click the "AI Generate" button
2. Enter a description, e.g., "Create an order management Cube with metrics like order amount and quantity"
3. AI automatically generates measure and dimension definitions

## Measure Configuration

Add measures on the Cube editing page:

```json
{
  "name": "total_amount",
  "display_name": "Total Order Amount",
  "description": "Sum of all order amounts",
  "type": "sum",
  "sql": "SUM({field})",
  "field": "amount"
}
```

## Dimension Configuration

Add dimensions on the Cube editing page:

```json
{
  "name": "order_date",
  "display_name": "Order Date",
  "sql": "order_date",
  "time_granularity": "day"
}
```

## Join Configuration

Define JOIN relationships between Cubes:

```json
{
  "name": "orders_to_users",
  "from": "orders",
  "to": "users",
  "type": "many_to_one",
  "on": "orders.user_id = users.id"
}
```

| Join Type | Description |
|:----------|:------------|
| many_to_one | Many-to-one (e.g., multiple orders belong to one user) |
| one_to_many | One-to-many |
| many_to_many | Many-to-many |

## Metric Queries

After configuring Cubes, you can perform metric queries through the semantic layer API:

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

The system automatically translates semantic queries into SQL and executes them.

## Engine Management

Go to "Semantic Layer → Engine Management" to view the semantic layer engine status:

| Info | Description |
|:-----|:------------|
| Adapter Type | local / cubejs |
| Health Status | Whether the engine is running normally |
| Cube Count | Number of configured Cubes |

### Adapters

| Adapter | Description |
|:--------|:------------|
| local | Built-in engine, queries the database directly |
| cubejs | Connects to an external CubeJS service |

Configuration in `configs/config.yaml`:

```yaml
semantic:
  default_adapter: local
  cubejs:
    api_url: ""
    api_token: ""
```

## Use Cases

### Business Metrics Platform

Encapsulate complex SQL aggregation logic into Cubes, allowing business users to retrieve data through simple metric queries.

### AI Data Assistant

Provide structured business metadata to AI Agents, enabling AI to accurately understand business data and generate correct queries.

### Unified Metric Definitions

Ensure that different reports and different people querying the same metric use the same calculation logic.
