# File & Storage Management

## 1. Overview

DBridge provides comprehensive file management and storage management capabilities. The File Management page is used for browsing, uploading, downloading, and managing files; the Storage Management page is used for configuring storage backend instances and module bindings. The system supports two backend types: local filesystem and S3-compatible storage.

**Core Capabilities**:
- Directory tree navigation and file browsing
- File upload, download, and batch operations
- Multi-storage instance management (Local / S3)
- Cross-storage file copy, move, and sync
- Module-level storage binding

---

## 2. File Management

### 2.1 Accessing File Management

In the left navigation bar, click **Files** to enter the file management page.

### 2.2 Page Layout

The file management page is divided into two areas:

**Left Panel**:
- **Storage Profile Selector**: Dropdown to switch between storage instances, showing quota usage (used/total)
- **Directory Tree**: Tree structure showing folder hierarchy; click folders to expand or navigate

**Right Panel**:
- **Breadcrumb Navigation**: Shows current path; click to quickly jump to parent directories
- **File List**: Table displaying files and folders in the current directory

### 2.3 File Operations

| Action | Description |
|--------|-------------|
| Upload | Click **Upload** button, select local files to upload to the current directory |
| New Folder | Click **New Folder** button, enter folder name |
| Download | Click **Download** in the file action bar |
| Rename | Click **Rename** in the file action bar, enter new name |
| Delete | Click **Delete** in the file action bar, confirm to delete |
| Copy To | Copy file to another storage instance or directory |
| Move To | Move file to another storage instance or directory |

### 2.4 Batch Operations

After selecting multiple files, batch action buttons appear in the top toolbar:

| Action | Description |
|--------|-------------|
| Batch Download | Package selected files as ZIP for download |
| Batch Copy | Copy selected files to another location in batch |
| Batch Move | Move selected files to another location in batch |
| Sync To | Sync selected files to another storage instance |
| Batch Delete | Delete selected files in batch |

> **Note**: Directories cannot be selected; batch operations only apply to files.

### 2.5 Search

Enter keywords in the search box to quickly filter files in the current directory.

### 2.6 Cross-Storage Operations

**Copy/Move**:

1. Select files, then click **Copy** or **Move** button
2. In the dialog, select the target storage instance
3. Select the target path
4. Check whether to overwrite existing files
5. Confirm execution

**Sync**:

1. Select files, then click **Sync To** button
2. Configure target storage and path in the sync dialog
3. Confirm sync execution

---

## 3. Storage Management

### 3.1 Accessing Storage Management

In the left navigation bar, click **Storage Management** to enter the storage management page.

### 3.2 Storage Instances

Switch to the **Instances** tab to manage storage backend instances.

**Instance List Fields**:

| Column | Description |
|--------|-------------|
| Name | Storage instance name (default instance marked with star) |
| Code | Unique identifier code |
| Backend Type | Local or S3 |
| Connection Status | Not tested / Connected / Connection failed |
| Enabled | Toggle to enable/disable |
| Actions | Test connection, Edit, Delete |

**Creating a Storage Instance**:

Click the **Add Instance** button and fill in:

*Local Storage*:
- Name
- Code (unique identifier, cannot be modified after creation)
- Backend type: Select "Local"
- Root Directory: Local filesystem path

*S3 Storage*:
- Name
- Code
- Backend type: Select "S3"
- Bucket: S3 bucket name
- Endpoint: S3 service endpoint
- Region: Region
- Access Key: Access key
- Secret Key: Secret key

> **Tip**: The system automatically detects MinIO (based on endpoint patterns) and adjusts configuration accordingly.

**Test Connection**: Click the **Test Connection** button to verify the storage configuration.

**Set as Default**: Each storage type can only have one default instance. The default instance is used as the default storage for system modules.

### 3.3 Module Bindings

Switch to the **Bindings** tab to manage the binding relationships between system modules and storage instances.

Binding configuration includes:
- **Module Name**: The system module using storage
- **Storage Instance**: The bound storage instance (only enabled instances shown)
- **Base Path**: The root path for this module in storage

> **Path Restrictions**: Base paths must not contain `..`, must not use absolute paths, and must not contain special characters.

---

## 4. Quota Management

Each storage instance can be configured with storage quotas. Next to the storage profile selector on the file management page, the current quota usage is displayed (e.g., `2.5 GB / 10 GB`).

Global file upload limits can also be set in system configuration:

| Config | Description |
|--------|-------------|
| Max Single File | Maximum size for a single file |
| Max Directory Depth | Maximum allowed directory nesting level |
| Total Quota | Total storage capacity limit |
| Allowed File Types | List of allowed file extensions for upload |

---

## 5. Use Cases

1. **SQL File Management**: Upload and manage SQL script files
2. **Data Import**: Upload CSV, JSON, and other data files for import
3. **Backup Management**: Store database backup files
4. **Report Resources**: Manage data files and templates used by reports
5. **Multi-Environment Storage**: Configure different storage instances for development, testing, and production environments

---

## 6. Best Practices

1. **Use S3 in production**: S3 storage offers better reliability and scalability
2. **Regular cleanup**: Delete files no longer needed to free up storage space
3. **Organize directories properly**: Use directory structures to organize files for easier management
4. **Set quotas**: Configure quotas for storage instances to prevent disk space exhaustion
5. **Test connections**: After creating a storage instance, always test the connection
