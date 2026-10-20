# Export & Import

The Export & Import feature provides database-level data migration capabilities, supporting cross-database migration, backup and recovery, and other scenarios.

## Features

- Export database structure and data as SQL files
- Import from three sources: database, file upload, or file storage
- Selectable export scope (specific tables / entire Schema)
- Selectable export content (structure+data / structure only / data only)
- Import conflict strategies (fail / skip / replace)
- Task-based management with execution log viewing

## Export Tasks

### Creating an Export Task

1. Go to the "Export Tasks" page
2. Click the **"New Export"** button
3. Fill in the export configuration:

| Field | Description |
|:------|:------------|
| Data Source | Select the database connection to export from |
| Database | Select the database to export |
| Schema | Select Schema (PostgreSQL / SQL Server) |
| Scope | Selected tables / Entire Schema |
| Content | Structure+Data / Structure Only / Data Only |
| Format | SQL |
| Batch Size | Rows per export batch (default 5000) |

4. If "Selected tables" is chosen, check the tables to export
5. Click "OK" to create the task

### Export Content Options

| Option | Description |
|:-------|:------------|
| Structure+Data | Export CREATE TABLE + INSERT statements |
| Structure Only | Only export CREATE TABLE statements |
| Data Only | Only export INSERT statements |

### Export Output

After export completion, a SQL file is generated containing:
- CREATE TABLE statements (including indexes and constraints)
- INSERT INTO statements (data rows)
- Comments and separators

## Import Tasks

### Creating an Import Task

Import supports three data sources:

#### Method 1: Database to Database

1. Select source data source and Schema
2. Select tables to import
3. Select target data source and Schema
4. Configure import options

#### Method 2: Upload File

1. Upload SQL file (supports .sql, .zip)
2. Select target data source and Schema
3. Configure import options

#### Method 3: From File Storage

1. Browse from configured file storage
2. Select the file to import
3. Select target data source and Schema
4. Configure import options

### Import Options

| Option | Description |
|:-------|:------------|
| Target Data Source | Which database connection to import into |
| Target Database | Which database to import into |
| Target Schema | Which Schema to import into |
| Content | Structure+Data / Structure Only / Data Only |
| Conflict Strategy | Fail / Skip / Replace |
| Skip Safety Checks | Skip target table existence check |

### Conflict Strategies

| Strategy | Description |
|:---------|:------------|
| Fail | Stop with error if target table already exists |
| Skip | Skip the table if target table already exists |
| Replace | Drop the target table and recreate it |

## Task Management

### Task List

| Column | Description |
|:-------|:------------|
| Type | Export / Import |
| Data Source | Source/target data source |
| Status | pending / running / completed / failed |
| Progress | Execution progress |
| Actions | Start / Cancel / Delete / View Details |

### Execution Details

Click "View Details" to expand a sub-table showing execution steps:

| Info | Description |
|:-----|:------------|
| Step Name | Current execution step |
| Status | Success / Failed / In Progress |
| Log | Execution log (detailed content viewable) |
| Manifest | Export/import file manifest |

### Create Import from Export

After an export completes, you can directly create an import task based on the export result:

1. Click the "Create Import" button in the export task list
2. The system auto-fills source information (export output becomes import source)
3. Select the target data source
4. Configure import options
5. Execute import

## Use Cases

### Database Migration

```
1. Create an export task on the source database (structure+data)
2. Wait for export to complete
3. Create an import task based on the export result
4. Select the target database, set conflict strategy to "Replace"
5. Execute import
```

### Data Backup

```
1. Create an export task
2. Select "Structure Only" or "Structure+Data"
3. Download the exported SQL file
4. Save to a secure location
```

### Environment Synchronization

```
1. Export from production (structure only)
2. Import into the test environment
3. Set conflict strategy to "Replace"
4. Test environment now has the same structure as production
```
