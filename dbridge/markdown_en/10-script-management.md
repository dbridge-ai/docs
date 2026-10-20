# Script Management

The Script Management feature provides directory-based management of SQL script files, supporting creation, editing, organization of SQL scripts with automatic saving.

## Features

- Directory tree for organizing SQL script files
- Monaco Editor for writing SQL
- Auto-save (1-second debounce)
- Create files and folders
- Rename and delete
- Breadcrumb navigation

## Page Layout

```
┌──────────────┬─────────────────────────────┐
│ Script Tree  │  Breadcrumb: Home / Reports / Monthly │
│              ├─────────────────────────────┤
│  📁 Reports   │                             │
│    📄 monthly.sql │  SELECT                │
│    📄 daily.sql   │    date,               │
│  📁 Migrations  │    COUNT(*)              │
│    📄 v1.sql   │  FROM orders              │
│    📄 v2.sql   │  GROUP BY date            │
│              │                             │
│              │        Monaco Editor        │
└──────────────┴─────────────────────────────┘
```

## Directory Tree Operations

### Creating Files

1. Right-click the target folder
2. Select "New File"
3. Enter the file name (must end with `.sql`)
4. The editor opens automatically after creation

### Creating Folders

1. Right-click the target location
2. Select "New Folder"
3. Enter the folder name

### Renaming

1. Right-click the file or folder
2. Select "Rename"
3. Enter the new name

### Deleting

1. Right-click the file or folder
2. Select "Delete"
3. Confirm deletion (folders recursively delete all contained files)

## Editor

The script editor is based on Monaco Editor, using the same editing engine as the SQL Editor:

- SQL syntax highlighting
- Auto-completion
- Find/Replace
- Multi-cursor
- Code folding

### Auto-Save

Content is automatically saved without manual intervention:

- Auto-saves within 1 second after each keystroke
- Save status is displayed in the editor title bar
- Content is fully preserved when reopening after closing the page

## Breadcrumb Navigation

The path breadcrumb is displayed above the editor, such as:

```
Home / Reports / monthly.sql
```

Click any level in the breadcrumb to quickly navigate.

## Use Cases

### Frequently Used SQL

Save commonly used queries as script files, organized by business category:

```
📁 Common Queries
  📄 user_stats.sql
  📄 order_analysis.sql
  📄 inventory_check.sql
```

### Database Migration

Organize migration SQL by version:

```
📁 Migration Scripts
  📄 v1.0_init.sql
  📄 v1.1_add_user_role.sql
  📄 v1.2_create_index.sql
```

### Report Templates

Save report queries as templates for periodic execution:

```
📁 Reports
  📄 daily.sql
  📄 weekly.sql
  📄 monthly.sql
```
