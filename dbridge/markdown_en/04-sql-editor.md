# SQL Editor & Data Query

The SQL Editor is the core feature page of DBridge Pro, providing a one-stop workspace for SQL writing, execution, data browsing, and editing.

## Page Layout

The SQL Editor uses a four-zone layout:

```
┌──────────┬──────────────────────────┬──────────┐
│          │     SQL Editing Area     │          │
│  Schema  │   (Monaco Editor)        │   AI     │
│   Tree   │                          │Assistant │
│          ├──────────────────────────┤  Panel   │
│          │     Query Results        │          │
│          │   (Data Table)           │          │
└──────────┴──────────────────────────┴──────────┘
```

| Zone | Description |
|:-----|:------------|
| Left Panel | Schema tree navigation, resizable by dragging |
| Upper Center | Monaco SQL Editor with multi-tab support |
| Lower Center | Query results table |
| Right Panel | AI assistant conversation panel, collapsible |

## Schema Tree

The left panel displays the database structure of the current data source:

```
📁 Database name
  📁 schema (PostgreSQL/SQL Server)
    📋 Table name
      📄 Column name (type)
    📋 View name
```

### Tree Node Operations

| Operation | Method | Description |
|:----------|:-------|:------------|
| Expand/Collapse | Click arrow | Expand databases, schemas, table structures |
| Generate SELECT | Single-click table name | Auto-generates `SELECT * FROM table LIMIT 100` |
| View Table Structure | Right-click → View Table Structure | Opens table structure drawer |
| Copy DDL | Right-click → Copy DDL | Copies CREATE TABLE statement to clipboard |
| Create Table | Right-click → Create Table | Opens table creation wizard |
| Create View | Right-click → Create View | Opens create view dialog |
| View View Definition | Right-click → View View Definition | Read-only view of the view's SQL |

## SQL Editor

### Monaco Editor

Based on the same editor as VS Code, supporting:

- SQL syntax highlighting
- Auto-completion (table names, column names, keywords)
- Multi-cursor editing
- Find/Replace (Ctrl+F / Ctrl+H)
- Code folding
- Minimap

### Executing SQL

| Operation | Shortcut | Description |
|:----------|:---------|:------------|
| Execute All | Ctrl+Enter | Execute all SQL in the editor |
| Execute Selection | Ctrl+Shift+Enter | Execute only the selected SQL fragment |

### Multi-Tab Support

The editor supports up to 10 tabs:

- Each tab can be SQL editing, table data browsing, or Schema listing
- When exceeding 10 tabs, the least recently used tab is automatically closed (LRU)
- Tabs are independent and do not affect each other

## Query Results

After executing SQL, results are displayed in the table below:

### Result Table Features

| Feature | Description |
|:--------|:------------|
| Column Sorting | Click column header to sort |
| Column Filtering | Click column header filter button |
| Pagination | Pagination controls at the bottom |
| Horizontal Scrolling | Scroll horizontally when there are many columns |
| Row Numbers | Row numbers displayed on the left |
| Execution Time | Query duration shown at the bottom |
| Total Rows | Total result row count shown at the bottom |

### Exporting Results

Query results can be exported:

1. Click the **"Export"** button in the result area toolbar
2. Select export format (CSV / JSON)
3. Download the file

## Data Browsing & Editing

### Browsing Table Data

1. Click a table name in the Schema tree
2. The system auto-generates and executes a `SELECT` query
3. Results are displayed in the table below

### Editing Table Data

In the table data view, you can edit data directly:

| Operation | Method | Description |
|:----------|:-------|:------------|
| Edit Cell | Double-click cell | Enter edit mode, press Enter to save after modification |
| Add Row | Click **"+ Add Row"** | Adds an empty row at the bottom of the table |
| Delete Row | Select row → Click delete | Supports batch deletion |
| Row Details | Click row expand button | View all fields for a single row |

> Data editing operations are recorded in the audit log.

## Table Structure Drawer

Double-click a table name or right-click and select "View Table Structure" to open the table structure panel:

### Tabs

| Tab | Content |
|:----|:--------|
| Fields | Column name, type, length, nullable, default value, comments |
| Indexes | Index name, type, included columns |
| Constraints | Constraint name, type, reference relationships |
| DDL | DDL statement (Monaco read-only display) |
| Table Info | Engine, character set, collation, row count, creation time |

### Structure Editing (admin/operator)

Users with permissions can modify directly in the table structure Drawer:

- **Add Column** — Specify column name, type, length, nullable, default value, comments
- **Edit Column** — Modify column attributes
- **Delete Column** — Delete column (high-risk operation, requires double confirmation)
- **Modify Table Comment** — Update the table's comment information

Modification operations generate ALTER DDL previews before execution.

## Script Files

The SQL Editor integrates script file management:

1. SQL written in the editor can be saved as script files
2. Script files are organized in a directory tree
3. Supports auto-save (1-second debounce)
4. See the [Script Management](10-script-management.md) chapter for details

## AI Assistant Panel

The right-side AI assistant panel allows conversation within the current query context:

1. Select a data source and Schema
2. Enter a natural language question
3. AI generates SQL that can be executed directly
4. See the [AI Assistant](12-ai-assistant.md) chapter for details

## Tips

### Efficient Querying

- Double-click a table name in the Schema tree to quickly generate a SELECT statement
- Select partial SQL and press Ctrl+Shift+Enter to execute only the selection
- Use Ctrl+Space to trigger auto-completion

### Switching Data Sources

Switch data sources at the top of the page — the Schema tree refreshes automatically.

### Handling Large Result Sets

- The system adds LIMIT to SELECT queries by default
- To view full data, manually write SQL without LIMIT
- Note: Large result sets may affect browser performance
