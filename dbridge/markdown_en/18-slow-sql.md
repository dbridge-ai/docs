# Slow SQL Monitor

## 1. Overview

The Slow SQL Monitor helps DBAs and developers identify performance bottlenecks in databases in real time. By monitoring SQL statements that exceed a configurable execution time threshold, analyzing historical Top SQL, and viewing lock wait chains, you can quickly locate and resolve performance issues.

**Core Capabilities**:
- Real-time slow SQL monitoring with customizable thresholds
- Historical Top SQL statistics and sorting
- Blocking chain analysis with visual lock dependency display
- Online EXPLAIN execution plan analysis
- One-click session termination

---

## 2. Accessing Slow SQL Monitor

Slow SQL monitoring is integrated into **Server Management**. Steps:

1. Go to the **Data Source Management** page
2. Find the target data source and click the **Server Management** button
3. In the server management window, switch to the **Slow SQL** tab

> **Prerequisites**: The system automatically performs pre-checks (e.g., whether MySQL's `performance_schema` is enabled). If prerequisites are not met, a notification is displayed at the top of the page.

---

## 3. Real-Time Slow SQL

The real-time slow SQL tab displays currently executing SQL statements that exceed the set threshold.

### 3.1 Threshold Settings

Select the slow SQL threshold from the dropdown at the top of the page:

| Threshold | Use Case |
|-----------|----------|
| 1 second | High-concurrency OLTP systems, response-time sensitive |
| 2 seconds | Standard web applications |
| 5 seconds | Mixed workload systems |
| 10 seconds | Report-oriented queries |
| 30 seconds | Data warehouses, batch processing |
| 60 seconds | Only extreme slow queries |

After selecting a threshold, click the **Refresh** button to load data.

### 3.2 Column Descriptions

| Column | Description |
|--------|-------------|
| Session ID | Database session identifier |
| User | Database user executing the SQL |
| Database | Target database name |
| Duration | SQL execution time so far. Red indicates >10s, orange indicates >5s |
| SQL Text | The SQL statement being executed |
| Wait Event | Current wait event type |
| Tags | Auto-classification labels |
| Actions | EXPLAIN (view execution plan), KILL (terminate session) |

### 3.3 Auto Tags

The system automatically adds tags based on SQL status:

- **Blocking Chain** (red): This SQL is blocking other sessions
- **Blocking Source** (orange): This SQL is blocked by another session
- **Severe Timeout**: Execution time far exceeds the threshold
- **Full Table Scan**: A full table scan operation is detected

### 3.4 Terminating Sessions

For long-running blocking or abnormal sessions, click the **KILL** button to terminate. A confirmation dialog appears; after confirmation, the `KILL` command is executed.

> **Note**: Terminating a session interrupts ongoing operations. Confirm that the session can be safely terminated.

---

## 4. Historical Top SQL

The historical Top SQL tab displays SQL statements with the highest cumulative execution count or longest average execution time.

### 4.1 Column Descriptions

| Column | Description |
|--------|-------------|
| SQL | SQL statement text |
| Calls | Cumulative execution count (sortable) |
| Avg Duration | Average execution time (sortable) |
| Actions | EXPLAIN (view execution plan) |

### 4.2 Usage Recommendations

- Sort by **Calls** to find frequently executed SQL; consider adding caching or optimization
- Sort by **Avg Duration** to find the slowest SQL; focus on optimizing execution plans
- Click **EXPLAIN** to view the execution plan and analyze issues like full table scans

---

## 5. Blocking Chains

The blocking chain tab displays lock wait relationships in the current database, helping quickly identify blocking sources.

### 5.1 Column Descriptions

| Column | Description |
|--------|-------------|
| Blocker ID | Session ID holding the lock (red tag) |
| Blocked IDs | Session IDs waiting for the lock (orange tags) |
| Depth | Depth of the blocking chain; 1 indicates direct blocking |

### 5.2 Analysis Method

1. Find the **Blocker ID** — this is the session holding the lock without releasing it
2. Check the corresponding **Blocked IDs** to understand how many sessions are blocked
3. Find the blocker's SQL statement in the **Real-Time Slow SQL** tab
4. Decide whether to terminate the blocker session or use EXPLAIN to analyze its execution plan

---

## 6. Execution Plan Analysis

The EXPLAIN feature helps analyze SQL execution plans to identify performance bottlenecks.

### 6.1 Steps

1. In the real-time slow SQL or historical Top SQL list, click the **EXPLAIN** button
2. The EXPLAIN dialog opens, showing the original SQL
3. You can edit the SQL text (e.g., add hints, modify conditions)
4. Click the **Execute EXPLAIN** button
5. View the execution plan results

### 6.2 Result Display

- **Standard format**: Displays the execution plan in table format (for MySQL, PostgreSQL, etc.)
- **XML format**: SQL Server returns XML execution plans; the system auto-formats with indentation

### 6.3 Copying Results

Click the **Copy** button to copy the execution plan to the clipboard for pasting into documents or sharing with your team.

---

## 7. Supported Databases

| Database | Support | Notes |
|----------|---------|-------|
| MySQL | Full | Requires `performance_schema` enabled |
| PostgreSQL | Full | Via `pg_stat_activity` and `pg_stat_statements` |
| SQL Server | Full | Via DMV views |
| Oracle | Partial | Via `V$SESSION` and `V$SQL` |
| SQLite | Not supported | SQLite is an embedded database with no separate process monitoring |

---

## 8. Best Practices

1. **Regular Inspection**: Check historical Top SQL daily to identify potential performance issues early
2. **Set Appropriate Thresholds**: Choose thresholds based on your business scenario to avoid excessive noise or missed issues
3. **Use Execution Plans**: After identifying slow SQL, always use EXPLAIN to analyze the execution plan
4. **Terminate Sessions Cautiously**: Confirm the session is not a critical business operation before terminating
5. **Monitor Blocking Chains**: Blocking chains cause large request backlogs and should be prioritized
