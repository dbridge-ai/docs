# Audit Logs

## 1. Overview

Audit Logs record all critical operations in the system, providing complete operation traceability. Through audit logs, administrators can understand who performed what operation at what time, whether it was successful, and the detailed parameters of the operation.

**Core Capabilities**:
- Comprehensive operation recording
- Filtering by module and result
- Expandable JSON details
- Paginated browsing
- Auto-cleanup configuration

---

## 2. Accessing Audit Logs

In the left navigation bar, click **Audit Logs** to enter the audit log page.

---

## 3. Log List

### 3.1 Filter Conditions

Two filter dropdowns are provided at the top of the page:

**Module Filter**:
| Module | Description |
|--------|-------------|
| All | Show logs from all modules |
| System | System-level operations (login, settings changes, etc.) |
| Datasource | Data source CRUD operations |
| Query | SQL query execution |
| Sync | Data sync operations |
| AI | AI assistant related operations |
| Security | Security related operations (masking rules, permission changes, etc.) |
| Report | Report operations |
| Export | Data export operations |
| Compare | Structure comparison operations |

**Result Filter**:
| Result | Description |
|--------|-------------|
| All | Show both successful and failed logs |
| Success | Only show successful operations |
| Failure | Only show failed operations |

### 3.2 List Fields

| Column | Description |
|--------|-------------|
| Module | Operation module (tag format) |
| Operation | Specific operation type (tag format), e.g., "User Login", "DS Create", "Query Execute", "DDL", "AI Chat" |
| Details | Summary of operation details (truncated, hover to see full content) |
| Result | Operation result (green tag for success, red tag for failure) |
| Operator | Username of the person performing the operation |
| Target | Name of the operation target |
| IP Address | Source IP of the operation |
| Time | When the operation was executed |

### 3.3 Expanding Details

Click the expand arrow before a table row to view the complete JSON details of the operation. Details include:
- Request parameters
- Operation context
- Error information (if the operation failed)
- Other metadata

---

## 4. Pagination

Audit logs support paginated browsing with selectable page sizes of 10, 20, or 50 records per page. The bottom of the page shows the total record count and current page number.

---

## 5. Auto Cleanup

To prevent audit logs from growing indefinitely, the system supports configuring automatic cleanup policies. See the audit log cleanup configuration in [System Settings](./27-settings.md).

---

## 6. Use Cases

1. **Security Auditing**: Regularly check sensitive operations (e.g., data source deletion, user permission changes)
2. **Troubleshooting**: Trace the cause of issues through operation logs
3. **Compliance**: Meet industry compliance requirements for operation auditing
4. **Behavior Analysis**: Understand user usage patterns and operation modes

---

## 7. Notes

- Audit logs only record operation information, not query result data
- Failed operations are also recorded for troubleshooting
- Operator information is based on the current logged-in user; API Key calls record the corresponding Key identifier
- It is recommended to periodically export important audit logs for archiving
