# Data Sync

The Data Sync feature provides task-based data migration capabilities, supporting full, incremental, differential, and other synchronization modes.

## Features

- Create data sync tasks
- Multiple sync modes (full / incremental / DDL)
- Task status management (start / stop)
- Sync progress tracking
- Sync data archiving

## Sync Task List

The "Sync Tasks" page displays all created sync tasks:

| Column | Description |
|:-------|:------------|
| Name | Task name |
| Source | Data source |
| Target | Data destination |
| Source Table | Source table name |
| Target Table | Target table name |
| Sync Mode | full / incremental / ddl |
| Status | pending / running / completed / failed / stopped |
| Progress | Percentage progress bar |
| Actions | Start / Stop / Edit / Delete |

## Creating a Sync Task

1. Click the **"New"** button
2. Fill in the task information:

| Field | Description |
|:------|:------------|
| Name | Task name |
| Source | Select the source database connection |
| Target | Select the target database connection |
| Source Table | Source table to sync |
| Target Table | Target table to receive data |
| Sync Mode | Full / Incremental / DDL |

3. Click "OK" to save

## Sync Modes

| Mode | Description | Use Case |
|:-----|:------------|:---------|
| full | Full sync — reads all source table data and writes to target | Initial migration, full backup |
| incremental | Incremental sync — only syncs new/changed data | Periodic sync, continuous migration |
| ddl | DDL sync — syncs table structure definitions | Structure consistency maintenance |

## Task Status

| Status | Description |
|:-------|:------------|
| pending | Waiting to execute |
| running | Currently executing (progress bar shown) |
| completed | Finished |
| failed | Execution failed (error details available) |
| stopped | Manually stopped |

## Task Actions

| Action | Description |
|:-------|:------------|
| Start | Begin executing the sync task |
| Stop | Terminate a running task |
| Edit | Modify task configuration |
| Delete | Delete the task (requires confirmation) |

## Compare Page Sync

In addition to task-based sync, you can perform data synchronization directly from the "Structure Compare" page. This approach is better suited for ad-hoc sync needs.

### Steps

1. Go to the "Structure Compare" page
2. Select source and target databases, execute comparison
3. Select the tables to sync
4. Click the "Sync Data" button
5. Configure sync options
6. Execute sync

### Sync Options

| Option | Description |
|:-------|:------------|
| Truncate Target | TRUNCATE the target table before syncing |
| Sync ID Fields | Include auto-increment primary key columns |
| Full Table Sync | Sync all data from the source table |
| Selected Rows Only | Only sync rows selected on the compare page |
| Row-Diff Sync | Compare by specified fields, only sync differing rows |
| Sync Specific Columns | Only sync selected columns |

### Row-Diff Sync

Row-diff sync is the most granular sync mode:

1. Select "Row-Diff Sync" mode
2. Specify check fields (used to match rows between source and target)
3. Specify sync columns (only update values in these columns)
4. The system compares row by row, only syncing rows with differences

**Algorithm**:
- Build an index of target table rows by check fields
- Iterate through source table rows:
  - Target row doesn't exist → Mark as INSERT
  - Target row exists but sync column values differ → Mark as UPDATE
  - Target row exists and values are the same → Skip

## Sync Safety

### Audit Logs

Every data sync operation is recorded in the audit log, including:
- Operator
- Source/target data sources
- Table name
- Sync mode and options
- Number of synced rows
- Execution result

### Data Archiving

Synced data is automatically exported as CSV files, organized by date directory:

```
data/sync_archive/
├── 2026-06-05/
│   └── ds-abc_users_20260605.csv
├── 2026-06-06/
│   └── ds-abc_users_20260606.csv
```

Archive files can be viewed online or downloaded from the audit log page.

### Transaction Protection

Data sync executes within transactions. On failure, the entire table is rolled back to ensure data consistency.

### TRUNCATE Warning

If "Truncate Target" is checked, the system displays the target table's current row count as a warning before execution.
