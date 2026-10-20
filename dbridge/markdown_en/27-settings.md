# System Settings

## 1. Overview

The System Settings page provides global configuration management capabilities, including theme switching and audit log auto-cleanup configuration.

**Core Capabilities**:
- Theme switching (light/dark/follow system)
- Audit log auto-cleanup
- Cleanup strategy configuration

---

## 2. Accessing System Settings

In the left navigation bar, click **Settings** to enter the settings page.

---

## 3. Theme Switching

A theme switcher component is provided at the top of the page, supporting the following themes:

| Theme | Description |
|-------|-------------|
| Light | Light background, suitable for daytime use |
| Dark | Dark background, suitable for nighttime use |
| Follow System | Automatically matches the operating system's theme setting |

Theme changes take effect immediately without page refresh.

---

## 4. Audit Log Cleanup

To prevent audit logs from growing indefinitely and consuming storage space, the system supports configuring automatic cleanup policies.

### 4.1 Enabling Auto Cleanup

Use the **Enable Auto Clean** toggle to control whether auto-cleanup is active. When turned off, the configuration fields below are disabled.

### 4.2 Configuration Fields

| Config | Description | Default | Range |
|--------|-------------|---------|-------|
| Retention Days | Maximum number of days to keep audit logs | 90 days | 1 - 365 days |
| Clean Time | Cleanup task execution time (Cron expression) | `0 3 * * *` | Standard Cron format |
| Batch Delete Size | Number of records deleted per cleanup run | 1000 | 100 - 5000 |

### 4.3 Cron Expression Format

The cleanup time uses Cron expression format:

```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6)
│ │ │ │ │
* * * * *
```

**Common Examples**:

| Expression | Description |
|-----------|-------------|
| `0 3 * * *` | Every day at 3:00 AM |
| `0 2 * * 0` | Every Sunday at 2:00 AM |
| `0 4 1 * *` | 1st of every month at 4:00 AM |
| `0 */6 * * *` | Every 6 hours |

### 4.4 Saving Configuration

After modifying the configuration, click **Save** to apply. The configuration takes effect immediately, and the next cleanup task runs with the new settings.

---

## 5. Cleanup Mechanism

### 5.1 Cleanup Logic

1. The system triggers cleanup tasks on schedule based on the configured Cron expression
2. Calculates the cutoff date: current time - retention days
3. Batch deletes audit logs before the cutoff date
4. Each deletion does not exceed the configured batch delete size
5. If remaining records still exceed the batch size, the next scheduled task continues cleanup

### 5.2 Notes

- Cleanup operations are irreversible; deleted records cannot be recovered
- Start with a longer retention period, then shorten after confirming no issues
- Don't set the batch delete size too large to avoid impacting database performance
- Cleanup tasks run in the background without affecting normal usage

---

## 6. Best Practices

1. **Enable auto-cleanup in production**: Prevent indefinite audit log growth
2. **Set reasonable retention days**: Configure based on compliance requirements; 90 days is usually sufficient
3. **Choose off-peak hours for cleanup**: Default is 3:00 AM; adjust based on business needs
4. **Regularly verify cleanup effectiveness**: Confirm cleanup tasks are running normally
5. **Archive important logs in advance**: Export important audit logs before they are cleaned up
