# SQL Dialect Conversion

The SQL Dialect Conversion feature supports converting SQL syntax between different databases, resolving syntax differences during database migration and cross-database queries.

## Features

- Supports conversion between MySQL, PostgreSQL, SQL Server, and Oracle dialects
- Two conversion modes: Rule-based (fast) and AI-powered (precise)
- SQL formatting
- Quick source/target dialect swap

## Page Layout

```
┌─────────────────────────────────────────────────┐
│  Source Dialect: [MySQL ▼]    Mode: [Rule-based ▼]   ⇄     │
│  Target Dialect: [PostgreSQL ▼]   Agent: [Select ▼]     │
├───────────────────────┬─────────────────────────┤
│  Source SQL           │  Conversion Result      │
│                       │                         │
│  SELECT               │  SELECT                 │
│    IFNULL(name, '')   │    COALESCE(name, '')   │
│  FROM users           │  FROM users             │
│  LIMIT 10             │  LIMIT 10               │
│                       │                         │
│  [Format]             │  [Format]               │
└───────────────────────┴─────────────────────────┘
```

## Steps

### 1. Select Dialects

- **Source Dialect** — The database type of the original SQL
- **Target Dialect** — The database type to convert to

### 2. Select Conversion Mode

| Mode | Description | Use Case |
|:-----|:------------|:---------|
| Rule-based | Fast conversion based on syntax rules, no AI needed | Simple syntax differences, large batch conversions |
| AI-powered | Intelligent conversion using AI Agent, more accurate | Complex queries, stored procedures, functions |

### 3. Input SQL

Enter or paste the source SQL in the left editor.

### 4. Execute Conversion

The system automatically converts the source SQL to the target dialect. Results are displayed on the right.

### 5. Format

Click the "Format" button to beautify the SQL layout. Source and target SQL can be formatted independently.

### 6. Swap Dialects

Click the **⇄** button to quickly swap source and target dialects.

## Supported Conversions

### Common Conversion Examples

| Source (MySQL) | Target (PostgreSQL) | Description |
|:---------------|:--------------------|:------------|
| `IFNULL(a, b)` | `COALESCE(a, b)` | Null handling |
| `NOW()` | `NOW()` | Current time |
| `LIMIT n` | `LIMIT n` | Row limit |
| `` `column` `` | `"column"` | Identifier quoting |
| `AUTO_INCREMENT` | `SERIAL` | Auto-increment column |
| `DATETIME` | `TIMESTAMP` | Date/time type |
| `GROUP_CONCAT()` | `STRING_AGG()` | String aggregation |

| Source (MySQL) | Target (SQL Server) | Description |
|:---------------|:--------------------|:------------|
| `LIMIT n` | `TOP n` | Row limit |
| `IFNULL(a, b)` | `ISNULL(a, b)` | Null handling |
| `NOW()` | `GETDATE()` | Current time |
| `CONCAT(a, b)` | `a + b` | String concatenation |

| Source (MySQL) | Target (Oracle) | Description |
|:---------------|:----------------|:------------|
| `LIMIT n` | `FETCH FIRST n ROWS` | Row limit |
| `IFNULL(a, b)` | `NVL(a, b)` | Null handling |
| `NOW()` | `SYSDATE` | Current time |
| `AUTO_INCREMENT` | `SEQUENCE` | Auto-increment column |

## AI Conversion Mode

The AI conversion mode uses large language models for intelligent conversion, suitable for complex scenarios:

- Stored procedure conversion
- Custom function conversion
- Complex nested query rewriting
- Database-specific syntax handling

To use AI conversion:

1. Select "AI-powered" mode
2. Choose an AI Agent
3. The system calls the Agent for conversion

> AI conversion requires a configured AI Provider (see [AI Provider Management](16-ai-provider.md)).

## Use Cases

### Database Migration

When migrating from MySQL to PostgreSQL, use dialect conversion to convert all MySQL-specific SQL statements to PostgreSQL-compatible syntax.

### Cross-Database Queries

When copying query statements between different databases, automatically convert syntax differences.

### Learning Reference

Understand syntax differences between different databases as a learning reference.
