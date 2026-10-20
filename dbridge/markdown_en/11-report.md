# Reports

The Reports panel provides custom SQL reporting capabilities, displaying query results in various chart formats including tables, bar charts, line charts, pie charts, and more.

## Features

- Create custom SQL reports
- Multiple chart type support
- Report category management
- AI-assisted report generation
- Report preview
- Cache configuration

## Report List

On the "Reports" page:

- **Left side** — Report category tree
- **Right side** — Report card grid

### Category Management

The left category tree supports the following operations:

| Action | Description |
|:-------|:------------|
| New Category | Create a new report category |
| Edit Name | Modify category name |
| Move | Adjust category position |
| Add Subcategory | Create nested categories |
| Delete | Delete category (requires confirmation) |

### Report Cards

Each report is displayed as a card containing:
- Report name
- Description
- Chart preview thumbnail

## Creating / Editing Reports

### Basic Information

| Field | Description |
|:------|:------------|
| Name | Report name |
| Description | Report description |
| Data Source | Select the data source for queries |
| Database | Select the database |
| Schema | Select Schema (if applicable) |
| Category | Report category |
| Chart Type | Display format |
| Cache Time | Query result cache duration (seconds) |
| Chart Config | Chart parameters in JSON format |

### SQL Editing

Write the report query SQL in the Monaco editor:

```sql
SELECT
    DATE(created_at) AS date,
    COUNT(*) AS order_count,
    SUM(amount) AS total_amount
FROM orders
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY DATE(created_at)
ORDER BY date
```

### Chart Types

| Type | Description | Use Case |
|:-----|:------------|:---------|
| table | Data table | General data display |
| bar | Bar chart | Category comparison |
| line | Line chart | Trend visualization |
| pie | Pie chart | Proportion distribution |
| area | Area chart | Cumulative trends |
| bar_grouped | Grouped bar chart | Multi-series comparison |
| bar_stacked | Stacked bar chart | Composition analysis |
| bar_percent | Percentage bar chart | Percentage composition |
| area_percent | Percentage area chart | Percentage trends |
| area_basic | Basic area chart | Simple trends |
| line_series | Multi-series line chart | Multi-metric trends |

### Chart Configuration

Chart configuration is in JSON format, with different chart types having different configuration options. Examples:

**Line chart configuration**:
```json
{
  "xField": "date",
  "yField": "total_amount",
  "seriesField": "category",
  "title": "Monthly Sales Trend"
}
```

**Pie chart configuration**:
```json
{
  "angleField": "count",
  "colorField": "status",
  "title": "Order Status Distribution"
}
```

The page provides a "Chart Config Help" panel showing configuration examples for each chart type.

### Execute & Preview

Click the "Execute & Preview" button:

1. The system executes the SQL query
2. Renders the chart based on configuration
3. Displays results in the preview area

## AI-Generated Reports

The system supports automatically generating reports via AI:

1. On the report editing page, find the AI generation panel
2. Enter a report description, e.g., "Show daily order count and sales for the last 30 days"
3. AI automatically generates:
   - SQL query statement
   - Report name
   - Recommended chart type
4. Preview and confirm before saving

> AI generation requires a configured AI Provider.

## Report Preview

In the report list, click the "Preview" button on a report card:

- A preview window pops up
- The chart is rendered
- Data is displayed

## Cache Configuration

Reports support query result caching to avoid frequently executing complex SQL:

| Config | Description |
|:-------|:------------|
| Cache Time | Cache validity period (seconds), 0 means no caching |
| Cache Refresh | Automatically re-query after cache expires |

## Use Cases

### Business Dashboard

Create reports for key business metrics:

```
📁 Business Dashboard
  📄 Daily Active Users Trend (line chart)
  📄 Revenue Distribution (pie chart)
  📄 Regional Sales Comparison (bar chart)
```

### Operations Monitoring

Create database operations reports:

```
📁 Operations Monitoring
  📄 Slow Query Statistics (table)
  📄 Tablespace Usage (bar chart)
  📄 Connection Count Trend (line chart)
```

### Data Analysis

Create data analysis reports:

```
📁 Data Analysis
  📄 User Growth Trend (area chart)
  📄 Product Sales Ranking (grouped bar chart)
  📄 Channel Conversion Rate (percentage bar chart)
```
