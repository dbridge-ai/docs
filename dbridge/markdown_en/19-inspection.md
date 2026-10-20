# Database Inspection

## 1. Overview

The Database Inspection feature collects database server runtime metrics and combines them with AI analysis to generate comprehensive database health check reports. The system automatically performs multiple rounds of metric collection to capture the database's operational state at different moments, then AI performs comprehensive analysis and provides optimization recommendations.

**Core Capabilities**:
- 10 rounds of automatic metric collection (every 10 seconds)
- Multi-dimensional metric coverage (connections, throughput, cache, locks, replication, storage)
- AI-powered intelligent analysis and diagnosis
- Inspection report history management
- Report detail viewing and copying

---

## 2. Accessing Inspection

The inspection feature is integrated into **Server Management**. Steps:

1. Go to the **Data Source Management** page
2. Find the target data source and click the **Server Management** button
3. In the server management window, switch to the **Inspection** tab

---

## 3. Enhanced Inspection

### 3.1 Starting Inspection

Click the **Enhanced Inspection** button to begin the inspection process. The entire process has two phases:

**Phase 1: Metric Collection**

The system automatically performs 10 rounds of metric collection, with a 10-second interval between rounds, totaling approximately 100 seconds.

During collection:
- The page displays a progress bar showing completed rounds (e.g., `5/10 rounds`)
- Remaining time is shown at the bottom
- If a round fails, the system automatically retries (up to 3 times with exponential backoff)

**Phase 2: AI Analysis**

After metric collection, the system sends the metric data and server information to AI for analysis. The AI will:
- Analyze metric trends
- Identify abnormal metrics
- Diagnose potential issues
- Provide optimization recommendations

During analysis, phased prompts are displayed ("Diagnosing", "Generating Report"). The entire process may take 1-3 minutes.

### 3.2 Canceling Collection

To cancel during collection, click the active button (which turns red) to terminate collection.

### 3.3 Collected Metrics

| Dimension | Metrics |
|-----------|---------|
| Connections | Current connections, max connections, active queries, lock-waiting connections |
| Throughput | Transaction commits/rollbacks, queries per second |
| Cache | Buffer hit rate, dirty pages |
| Locks | Total locks, blocked count, deadlock count |
| Replication | Replication lag, replication status |
| Storage | Database size, tablespace usage |

> **Note**: Different database types support different metrics. The system automatically adjusts based on the actual data returned.

---

## 4. Inspection Report List

After inspection, reports appear in the table below.

### 4.1 List Fields

| Column | Description |
|--------|-------------|
| Time | When the inspection was executed |
| Status | Inspection result status: ok (green), warning (orange), critical/error (red) |
| Summary | Brief description of the inspection result |
| Duration | Total inspection time (including metric collection and AI analysis) |
| Actions | View details, Delete |

### 4.2 Viewing Details

Click **View Details** to open a detail dialog containing:

- **Status**: Inspection result status
- **Time**: When the inspection was executed
- **Duration**: Total inspection time
- **Summary**: Brief description of the inspection result
- **Detail Content**: Complete AI-generated inspection report (rendered in Markdown)

### 4.3 Copying Reports

In the detail dialog, you can copy report content:

- **Copy Markdown**: Copy the raw Markdown format text
- **Copy Plain Text**: Copy the converted plain text content (Markdown markup removed)

---

## 5. Deleting Reports

For unwanted reports, click the **Delete** button to remove them. A confirmation dialog appears before deletion.

---

## 6. Supported Databases

| Database | Metric Collection | AI Analysis |
|----------|------------------|-------------|
| MySQL | Supported | Supported |
| PostgreSQL | Supported | Supported |
| SQL Server | Supported | Supported |
| Oracle | Supported | Supported |
| SQLite | Limited | Supported |

---

## 7. Recommendations

1. **Regular Inspection**: Perform an inspection on core databases daily
2. **Monitor Trends**: Compare multiple inspection reports to observe metric trends
3. **Heed Warnings**: If inspection status is warning or critical, investigate promptly
4. **Combine with Slow SQL**: After inspection reveals performance issues, use Slow SQL Monitor for deeper analysis
5. **Save Reports**: Copy and save important inspection reports as operational archives
