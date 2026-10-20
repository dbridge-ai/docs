# Dashboard

The Dashboard is the landing page after login, providing an overview of system health and custom data views.

## Overview

The Dashboard contains two types of content:

- **System Overview** — Fixed statistics cards showing core metrics such as data source count, sync task count, query count, and success rate
- **Custom Dashboards** — User-created tabs linked to backend dashboard views, displaying custom data as charts

## System Overview

After entering the system, the default overview tab is displayed with the following information:

### Statistics Cards

| Card | Description |
|:-----|:------------|
| Data Sources | Total number of configured data sources |
| Sync Tasks | Total number of created sync tasks |
| Query Count | Cumulative SQL query executions |
| Success Rate | Query success percentage |

### System Information

| Info | Description |
|:-----|:------------|
| System Version | Current DBridge Pro version number |
| Running Status | Whether the service is running normally |
| Audit Logs | Cumulative audit log count |

## Custom Dashboards

### Adding Tabs

1. Click the **"+"** button on the right side of the tab bar
2. The system creates a new dashboard tab
3. The new tab links to a backend Dashboard View

### Dashboard Views

Each custom tab corresponds to a dashboard view containing multiple chart components. Supported chart types:

| Chart Type | Description |
|:-----------|:------------|
| Bar Chart | Categorical data comparison |
| Line Chart | Trend visualization |
| Pie Chart | Proportion distribution |
| Area Chart | Cumulative trends |
| Grouped Bar Chart | Multi-series comparison |
| Stacked Bar Chart | Composition analysis |
| Percentage Bar Chart | Percentage composition |

### Deleting Tabs

Click the close button on a tab to delete it. The system overview tab cannot be deleted.

## Use Cases

- **Daily Monitoring** — Quickly understand overall system health
- **Data Display** — Showcase key business metrics through custom dashboards
- **Team Sharing** — All team members see the same system status
