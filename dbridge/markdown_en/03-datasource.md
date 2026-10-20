# Data Source Management

Data sources are configurations that connect DBridge Pro to external databases. All database operations are based on configured data sources.

## Features

- Create, edit, and delete database connections
- Support for 7 database types
- SSH tunnel connections
- SSL/TLS encrypted connections
- Connection testing
- Data source import/export
- Tags and environment labels

## Supported Data Source Types

| Database | Connection Method | Special Configuration |
|:---------|:-----------------|:---------------------|
| MySQL | TCP | Standard connection parameters |
| MariaDB | TCP | Same as MySQL |
| OceanBase | TCP | Same as MySQL |
| PostgreSQL | TCP | SSL Mode |
| SQL Server | TCP | Standard connection parameters |
| Oracle | TCP | Service Name / SID, connection mode, role |
| SQLite | File | File upload or storage browser |

## Creating a Data Source

### Steps

1. Go to the "Data Source Management" page
2. Click the **"New"** button
3. Fill in the connection information (see below)
4. Click **"Test Connection"** to verify connectivity
5. Click **"OK"** to save

### Connection Information

The data source creation dialog contains 4 tabs:

#### Connection Tab

| Field | Description | Required |
|:------|:------------|:---------|
| Name | Display name of the data source | Yes |
| Type | Database type (MySQL / PostgreSQL / Oracle / SQL Server / SQLite) | Yes |
| Host | Database server address (except SQLite) | Yes |
| Port | Database port number (except SQLite) | Yes |
| Database | Database name (file path for SQLite) | Yes |
| Username | Database username (except SQLite) | Yes |
| Password | Database password (stored with AES-256-GCM encryption) | Yes |

#### TLS/SSL Tab

For configuring encrypted connections:

| Field | Description |
|:------|:------------|
| Client Certificate | TLS client certificate file content |
| Client Key | TLS client private key file content |
| Root CA | CA root certificate file content |

#### SSH Tunnel Tab

When the database server is not directly accessible, you can connect through an SSH tunnel:

| Field | Description |
|:------|:------------|
| SSH Host | Jump server address |
| SSH Port | SSH port (default 22) |
| SSH Username | Login username |
| Auth Method | Password / Key / Key+Password / Agent / None |
| Timeout | Connection timeout (seconds) |

#### Advanced Tab

Additional driver parameters configured in JSON format.

### Oracle-Specific Configuration

Oracle data sources have additional connection options:

| Field | Options | Description |
|:------|:--------|:------------|
| Connection Mode | Service Name / SID | Oracle connection identifier type |
| Role | Default / SYSDBA / SYSOPER | Connection privilege level |

### SQLite-Specific Configuration

SQLite data sources have two configuration methods:

- **File Upload** — Directly upload `.db` / `.sqlite` files
- **Storage Browser** — Select from configured file storage

## Data Source List

The list displays all configured data sources with the following information:

| Column | Description |
|:-------|:------------|
| Name | Data source name |
| Type | Database type (color-coded tag) |
| Host:Port | Connection address |
| Database | Database name |
| Username | Connection username |
| Environment | dev / test / prod label |
| Tags | Feature tags (database_management / data_query) |
| Actions | Query, Edit, Delete |

### Filtering & Search

- **Search box** — Search by name
- **Tag filter** — Filter by feature tags
- **Environment filter** — Filter by environment label

## Connection Testing

In the create/edit dialog, click the **"Test Connection"** button:

- Success — Shows green message "Connection successful"
- Failure — Shows red error message with specific reason

Testing validates:
- Network connectivity
- Username/password correctness
- Database existence
- SSH tunnel availability (if SSH is configured)

## Tags & Environments

### Feature Tags

Add feature tags to data sources to control their visibility in the system:

| Tag | Description |
|:----|:------------|
| database_management | Visible on the Database Management page |
| data_query | Visible in the SQL Editor |

### Environment Labels

Label data sources with their environment for easy identification:

| Environment | Color | Description |
|:------------|:------|:------------|
| dev | Green | Development environment |
| test | Yellow | Testing environment |
| prod | Red | Production environment |

## Import / Export

### Exporting Data Sources

Export data source configurations to a JSON file (passwords are encrypted):

1. Click the **"Export"** button in the toolbar
2. Select the data sources to export
3. Set an export password
4. Download the JSON file

### Importing Data Sources

Import data source configurations from a JSON file:

1. Click the **"Import"** button in the toolbar
2. Select the JSON file
3. Enter the import password
4. Confirm import

> Import/export is ideal for sharing data source configurations between teams or migrating across environments.

## Password Security

Data source passwords use **AES-256-GCM** symmetric encryption:

- Encrypted at rest — Passwords are encrypted before being written to the database
- Never returned via API — API responses never include password fields
- Key management — Encryption keys are configured via the `DBRIDGE_CRYPTO_KEY` environment variable

## Quick Actions

Each row in the data source list provides quick actions:

| Action | Description |
|:-------|:------------|
| Query | Open this data source directly in the SQL Editor |
| Edit | Modify data source configuration |
| Delete | Delete data source (requires confirmation) |
