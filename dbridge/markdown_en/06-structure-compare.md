# Structure Compare

The Structure Compare feature compares database structure differences between two data sources, providing visual diff display and one-click synchronization capabilities.

## Features

- Cross-database structure comparison (supports comparison between MySQL, PostgreSQL, Oracle, SQL Server, and SQLite)
- Difference visualization (source only / target only / both)
- Column-level structure comparison
- DDL statement comparison
- Structure synchronization (DDL preview → edit → execute)
- Data comparison and synchronization

## Page Layout

```
┌─────────────────────────────────────────────────────┐
│  Source: [Select]  Database: [Select]   ⇄   Target: [Select]  Database: [Select]  │
│                                    [Compare]        │
├──────────────┬──────────────────────────────────────┤
│  Object List │  Detail Panel                        │
│              │  [Data] [Structure] [DDL]            │
│  🔍 Search   │                                      │
│              │  ┌──────────┬──────────┐             │
│  ✅ users    │  │ Source   │ Target   │             │
│  ⚠️ orders   │  │ Data     │ Data     │             │
│  🔵 products │  └──────────┴──────────┘             │
│              │                                      │
│              │  [Sync Structure]  [Sync Data]        │
└──────────────┴──────────────────────────────────────┘
```

## Steps

### 1. Select Data Sources

- **Source** — Select the left-side database for comparison
- **Target** — Select the right-side database for comparison
- Select the corresponding database/Schema for each

> For two-level databases (PostgreSQL, SQL Server), you need to select both database and Schema.
> For single-level databases (MySQL, Oracle), only select the database.

### 2. Execute Comparison

Click the **"Compare"** button. The system connects to both databases, queries `information_schema` for table/view metadata, and computes differences.

### 3. View Differences

After comparison, the left object list displays results:

| Icon | Meaning | Description |
|:-----|:--------|:------------|
| ✅ Green | Both have it | Table/view exists in both source and target |
| ⚠️ Orange | Source only | Table/view only exists in source, missing from target |
| 🔵 Blue | Target only | Table/view only exists in target |

Click an object name to display detailed information on the right.

### 4. View Details

The right detail panel has three tabs:

#### Data Tab

Displays source and target table data side by side:

- Supports paginated browsing
- Source data table has a Checkbox column for selecting rows
- Used for data-level comparison

#### Structure Tab

Column-level structure comparison:

| Column | Description |
|:-------|:------------|
| Column Name | Field name |
| Source Type | Data type in source database |
| Target Type | Data type in target database |
| Difference | Highlights differences (yellow background) |

Difference types include:
- Type differences (e.g., varchar(50) vs varchar(100))
- Nullability differences
- Default value differences
- Comment differences

#### DDL Tab

Displays the CREATE TABLE DDL statements from both source and target for easy comparison.

### 5. Swap Source/Target

Click the **⇄** button to quickly swap source and target data sources.

### 6. Comparison History

The system stores the last 5 comparison results (in browser localStorage) for quick review.

## Structure Synchronization

After identifying structural differences, you can synchronize directly:

### Sync Workflow (Two-Step)

1. **Preview** — Click "Sync Structure", the system generates DDL statements
2. **Edit** — Review/edit the DDL in the popup text area
3. **Execute** — After confirmation, execute the DDL on the target database

### Sync Actions

| Action | Scenario | Generated DDL |
|:-------|:---------|:--------------|
| create | Target doesn't have this table | CREATE TABLE (adapted to target dialect) |
| alter | Target has this table but structure differs | ALTER TABLE ADD/MODIFY COLUMN |

### DDL Preview

System-generated DDL can be edited before execution:

- Modify column definitions
- Add additional statements
- Remove unwanted changes

> It is recommended to carefully review the DDL before executing, especially in production environments.

## Data Synchronization

In addition to structure sync, data synchronization is also supported:

### Sync Modes

| Mode | Description |
|:-----|:------------|
| Full Sync | Write all source table data to the target table |
| Selected Rows Sync | Only sync rows selected during data comparison |
| Row-Diff Sync | Compare by specified fields, only sync rows with differences |

### Sync Options

| Option | Description |
|:-------|:------------|
| Truncate Target | TRUNCATE the target table before syncing |
| Sync ID Fields | Whether to sync auto-increment primary key columns |
| Sync Specific Columns | Only sync selected columns (Transfer component) |
| Check Fields | Field list used for comparison during row-diff sync |

### Sync Strategy

- Duplicate keys are automatically skipped (INSERT IGNORE / ON CONFLICT DO NOTHING)
- Executed within transactions; failure rolls back the entire table
- Synced data is automatically archived as CSV files
- Operations are recorded in the audit log

## Supported Cross-Database Comparison

Structure comparison supports comparison between any two of the following databases:

| Source | Target | Description |
|:-------|:-------|:------------|
| MySQL | PostgreSQL | Auto-adapts dialect differences |
| MySQL | Oracle | Type mapping conversion |
| PostgreSQL | SQL Server | Schema structure adaptation |
| Oracle | MySQL | Data type conversion |
| ... | ... | Any combination is supported |

> During cross-database comparison, the system automatically handles data type mapping differences (e.g., MySQL's datetime vs PostgreSQL's timestamp).
