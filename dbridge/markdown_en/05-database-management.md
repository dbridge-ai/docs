# Database Management

The Database Management page provides visual management of database objects, including viewing and managing schemas, tables, views, indexes, and constraints.

## Features

- Browse database structure (Schema / Tables / Views)
- View detailed table structure (columns, indexes, constraints, DDL)
- Modify table structure online (add/modify/delete columns and indexes)
- Create tables and views
- View view definitions
- Database-level operations (varies by database type)

## Page Entry Points

There are two ways to access Database Management:

1. **Navigation Menu** — Click "Database Management" in the left sidebar
2. **Data Source Quick Action** — Click the management button next to "Query" in the data source list

## Database List

The top of the page displays all databases (MySQL) or schemas (PostgreSQL / SQL Server) under the current connection.

### Two-Level Database Structure

For single-level structures like MySQL and Oracle:
```
Database → Tables/Views
```

For two-level structures like PostgreSQL and SQL Server:
```
Database → Schema → Tables/Views
```

## Table List

After selecting a database/schema, the tables and views within it are displayed:

| Column | Description |
|:-------|:------------|
| Name | Table/view name |
| Type | TABLE / VIEW |
| Rows | Estimated row count |
| Size | Data space usage |
| Comment | Table comment |
| Engine | Storage engine (MySQL) |
| Created | Table creation time |

## Table Structure Management

Click a table name to view its structure details, with the following tabs:

### Fields Tab

Displays all column information for the table:

| Column | Description |
|:-------|:------------|
| Column Name | Field name |
| Type | Data type (varchar, int, datetime, etc.) |
| Length | Type length/precision |
| Nullable | Whether NULL is allowed |
| Default | Default value |
| Key | PRI (primary key) / UNI (unique) / MUL (indexed) |
| Extra | auto_increment, etc. |
| Comment | Column comment |

**Action Buttons** (visible to admin/operator):

- **Add Column** — Opens the add column dialog
- **Edit** — Modify column attributes
- **Delete** — Delete column (high-risk, requires confirmation)

### Indexes Tab

Displays all indexes on the table:

| Column | Description |
|:-------|:------------|
| Index Name | Index name |
| Type | PRIMARY / UNIQUE / INDEX / FULLTEXT |
| Columns | Columns included in the index |
| Comment | Index comment |

### Constraints Tab

Displays constraint information for the table:

| Column | Description |
|:-------|:------------|
| Constraint Name | Constraint name |
| Type | FOREIGN KEY / UNIQUE / CHECK |
| Columns | Columns involved in the constraint |
| Reference Table | Table referenced by foreign key |
| Reference Column | Column referenced by foreign key |
| ON DELETE | Behavior on delete (CASCADE / RESTRICT / SET NULL) |
| ON UPDATE | Behavior on update |

### DDL Tab

Displays the full CREATE TABLE DDL statement in a read-only Monaco editor.

- Syntax highlighting supported
- "Copy" button in the top-right corner for one-click clipboard copy

### Table Info Tab

Displays table metadata:

| Info | Description |
|:-----|:------------|
| Storage Engine | InnoDB / MyISAM, etc. |
| Character Set | utf8mb4, etc. |
| Collation | utf8mb4_unicode_ci, etc. |
| Row Format | Dynamic / Compact, etc. |
| Estimated Rows | Approximate row count |
| Created | Table creation time |
| Updated | Last modification time |
| Table Comment | Table comment information |

## Modifying Table Structure

### Adding a Column

1. Click the **"+ Add Column"** button
2. Fill in the column information:

| Field | Description |
|:------|:------------|
| Column Name | Name of the new column |
| Type | Data type (select from whitelist) |
| Length | Type length/precision |
| Nullable | Whether NULL is allowed |
| Default | Default value |
| Comment | Column comment |
| Position | Place after which column (MySQL only) |

3. Click "OK"
4. The system generates an ALTER TABLE DDL preview
5. After confirming, click "Execute"

### Modifying a Column

1. Find the target column in the field list
2. Click the "Edit" button
3. Modify column attributes
4. The system generates ALTER TABLE MODIFY COLUMN DDL
5. Preview and confirm execution

### Deleting a Column

1. Find the target column in the field list
2. Click the "Delete" button
3. The system shows a warning (high-risk operation)
4. After confirmation, generates ALTER TABLE DROP COLUMN DDL
5. Preview and confirm execution

> Deleting a column permanently loses all data in that column. Proceed with caution.

### Change Preview & Execution

All table structure modifications follow the "Preview → Confirm → Execute" workflow:

1. **Preview** — The system generates DDL statements and rollback DDL
2. **Warning** — High-risk operations (e.g., DROP COLUMN) display red warnings
3. **Confirm** — Users can fine-tune the DDL in the Monaco editor
4. **Execute** — After confirmation, the DDL is executed and recorded in the audit log
5. **Rollback** — The system automatically saves rollback scripts for recovery

## Creating Tables

1. Right-click a database/schema in the Schema tree
2. Select "Create Table"
3. Fill in the table name and comment
4. Add column definitions
5. Preview the DDL
6. Confirm creation

## Creating Views

1. Right-click a database/schema in the Schema tree
2. Select "Create View"
3. Fill in the view name
4. Write the view SQL
5. Preview the DDL
6. Confirm creation

## Viewing View Definitions

1. Right-click a view in the Schema tree
2. Select "View View Definition"
3. The system displays the view's SELECT statement in read-only mode

## Permission Control

| Role | View | Modify |
|:-----|:-----|:-------|
| admin | Yes | Yes |
| operator | Yes | Yes |
| developer | Yes | No |
| viewer | Yes | No |

Users without modification permissions will have all edit buttons hidden, and the Drawer operates in read-only mode.
