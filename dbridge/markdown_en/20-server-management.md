# Server Management

## 1. Overview

Server Management provides comprehensive monitoring and management capabilities for database server runtime status. Based on the database type, the system displays corresponding management tabs including overview information, real-time monitoring, process management, user management, role management, tablespace management, and more.

**Core Capabilities**:
- Database overview information display
- Real-time performance metric monitoring
- Process/session management
- User and role management
- Tablespace management
- Database-type adaptive tabs

---

## 2. Accessing Server Management

1. Go to the **Data Source Management** page
2. Find the target data source and click the **Server Management** button (monitor icon)
3. The system loads the corresponding management tabs based on the database type

---

## 3. Tab Descriptions

Server Management tabs are dynamically displayed based on database type. Below are detailed descriptions of each tab.

### 3.1 Overview

The Overview tab displays basic information and key runtime metrics of the database server. Different database types use different overview components:

| Database Type | Overview Component |
|--------------|-------------------|
| MySQL / MariaDB / OceanBase | MySQL Overview |
| PostgreSQL | PostgreSQL Overview |
| Oracle | Oracle Overview |
| SQL Server | SQL Server Overview |
| SQLite | SQLite Overview |

Overview information typically includes:
- Database version
- Uptime
- Connection statistics
- Database size
- Key configuration parameters
- Character set and collation

### 3.2 Real-Time Monitor

The Real-Time Monitor tab displays a database performance metrics dashboard, including:

- **Connection Metrics**: Current connections, active connections, waiting connections
- **Query Metrics**: Queries per second (QPS), slow query count
- **Cache Metrics**: Buffer pool hit rate, cache usage
- **Lock Metrics**: Lock wait count, deadlock count
- **Replication Metrics**: Master-slave lag, replication status

Metrics auto-refresh and are displayed visually in charts.

### 3.3 Database List

Displays all databases on the server, including:
- Database name
- Size
- Table count
- Character set
- Creation time

### 3.4 Process Management

Displays the current active process/session list:

| Column | Description |
|--------|-------------|
| Process ID | Session/process identifier |
| User | Connected user |
| Database | Target database |
| Command | Current command type being executed |
| Duration | Execution time so far |
| Status | Process state |
| Info | SQL text or status information |

Supported actions:
- **Refresh**: Reload the process list
- **Terminate**: End a specific process (requires appropriate permissions)

### 3.5 User Management

Displays the database user list with viewing and management operations.

> **Permission Requirement**: User management requires database administrator privileges.

### 3.6 Role Management

Displays the database role list, only shown when the database supports roles (e.g., PostgreSQL).

### 3.7 Tablespace Management

Displays database tablespace usage:
- Tablespace name
- Total size
- Used size
- Usage percentage
- Status

### 3.8 Slow SQL

Integrates slow SQL monitoring. See [Slow SQL Monitor](./18-slow-sql.md) for details.

### 3.9 Inspection

Integrates database inspection. See [Database Inspection](./19-inspection.md) for details.

---

## 4. Database-Specific Tabs

Some tabs are only displayed for specific database types.

### 4.1 SQL Server Specific

| Tab | Description |
|-----|-------------|
| Logins | Manage SQL Server login accounts |
| Security | View database-level security configuration |

### 4.2 PostgreSQL Specific

| Tab | Description |
|-----|-------------|
| Extensions | Manage PostgreSQL extensions, supporting install and uninstall |

### 4.3 SQLite Specific

| Tab | Description |
|-----|-------------|
| Storage | View SQLite file storage information |
| PRAGMA Config | View and modify SQLite PRAGMA settings |

---

## 5. Permission Control

Server Management features are controlled by the following factors:

1. **Database Type**: Different databases support different management features
2. **Capabilities**: The system determines which tabs to display based on capabilities returned during connection testing
3. **Feature Flags**: Some features (e.g., inspection) can be toggled via system configuration
4. **Database User Permissions**: Management operations require appropriate database permissions

---

## 6. Recommendations

1. **Daily Monitoring**: Use the Overview and Real-Time Monitor tabs to understand database health
2. **Troubleshooting**: Combine Process Management, Slow SQL, and Inspection to investigate performance issues
3. **Security Management**: Regularly audit user and role permissions
4. **Capacity Planning**: Monitor tablespace usage and database size trends
5. **Terminate Abnormal Processes Promptly**: Terminate long-running or abnormal processes immediately
