# DBridge Pro Product Overview

## What is DBridge Pro?

DBridge Pro is a **web-based database management and synchronization platform** designed for developers and database administrators (DBAs). Through a single browser, you can perform daily management, SQL queries, structure comparison, data synchronization, import/export, and AI-powered analysis across five major databases: MySQL, PostgreSQL, Oracle, SQL Server, and SQLite.

## Core Values

### Say Goodbye to Desktop Clients

A pure B/S architecture — open your browser and start using it. Team members share the same instance without needing to install individual database client tools.

### Cross-Database Comparison & Synchronization

Supports structure comparison and one-click synchronization between MySQL, PostgreSQL, Oracle, SQL Server, and SQLite — a rare capability among similar products.

### SQL Dialect Conversion

SQL dialects across all five databases can be converted to one another. No more manually rewriting SQL during database migrations.

### Deep AI Integration

Built-in Agent / Skill / MCP tool system supporting natural language SQL generation, SQL explanation, performance optimization, database inspection, and automated report generation.

### Semantic Layer Engine

A built-in OLAP semantic layer with Cube modeling, metric definitions, and row-level security enables non-technical users to understand and work with data.

### IM Bot Integration

Supports DingTalk, Feishu (Lark), and WeCom bot integration — interact with your databases directly through instant messaging tools.

### Zero-Dependency Deployment

A single binary + SQLite embedded database — start with one command, no additional dependencies required.

## Feature Overview

| Module | Description |
|:-------|:------------|
| Dashboard | System overview, statistics cards, custom data views |
| Data Source Management | Multi-datasource connections, SSH tunnels, SSL/TLS, password encryption |
| SQL Editor | Monaco editor, Schema tree, data browsing/editing, AI assistant |
| Database Management | Table/view/index/constraint management, DDL preview and execution |
| Structure Compare | Cross-database table structure diff visualization |
| Data Sync | Full / selected rows / row-diff synchronization |
| Export & Import | SQL/CSV export, cross-database data migration |
| SQL Transpile | MySQL ↔ PostgreSQL ↔ SQL Server ↔ Oracle |
| Script Management | SQL script directory tree management with auto-save |
| Reports | Custom SQL reports with multiple chart types |
| AI Assistant | Multi-session conversations, Text-to-SQL, SQL optimization |
| Agent Management | AI Agent definition, execution, sub-agent orchestration |
| Skill Management | AI skill templates with version management |
| MCP Tools | Built-in MCP Server with 11+ tool categories |
| Semantic Layer | Cube modeling, metric queries, row-level security |
| Slow SQL Monitor | Real-time slow queries, historical Top SQL, blocking chain analysis |
| Database Inspection | Metrics collection with AI analysis reports |
| Server Management | Process/user/role/tablespace/login/security monitoring |
| Audit Logs | Full-chain operation records with sync data archiving |
| AI Security | Data masking, risk classification, approval execution |
| File Management | File upload/download with local/S3 storage |
| IM Bot | DingTalk / Feishu / WeCom bot integration |
| Authentication & Subscription | OAuth 2.0 / Local accounts / License management |
| System Settings | Theme and audit log retention policies |

## Supported Databases

| Database | Version Requirement | Connection Method |
|:---------|:-------------------|:-----------------|
| MySQL / MariaDB / OceanBase | 5.7+ | TCP |
| PostgreSQL | 12+ | TCP |
| SQL Server | 2012+ | TCP |
| Oracle | 11g+ | Service Name / SID |
| SQLite | 3.38+ | File path |

## Technical Architecture

```
Browser (React 19 + Ant Design 6)
         │
         │ REST API / WebSocket
         ▼
Go Application Server (Gin)
  ├── Authentication (JWT / OAuth 2.0 / License)
  ├── Data Source Management (5 database drivers)
  ├── SQL Execution Engine (dialect conversion)
  ├── Compare & Sync Engine (structure + data)
  ├── AI Agent Executor (Skill / MCP)
  ├── Semantic Layer Engine (OLAP / Cube)
  ├── IM Bot Adapters (DingTalk / Feishu / WeCom)
  └── Audit Logs
         │
    ┌────┼────┐
    ▼    ▼    ▼
  SQLite  Redis  NATS
  (metadata) (cache)  (messaging)
```

## Product Edition Relationships

The DBridge product family includes the following editions:

| Edition | Description |
|:--------|:------------|
| **dbridge-community** | Community edition — open source with basic database management features |
| **dbridge-pro** (this project) | Professional edition with all advanced features (AI, Semantic Layer, IM Bot, etc.) |
| **dbridge-desktop** | Desktop console for managing dbridge-pro service startup, configuration, logs, and auto-updates |
| **usercenter** | Membership center web application for managing subscriptions, licenses, devices, and users |

## Reading Guide

This book is organized by functional modules, with each chapter covering an independent feature. You can read it sequentially or jump directly to the feature you're interested in.

**Fundamentals** (recommended reading order):
- Quick Start → Dashboard → Data Source Management → SQL Editor

**Data Management**:
- Database Management → Structure Compare → Data Sync → Export & Import → SQL Transpile

**Productivity Tools**:
- Script Management → Reports → Slow SQL Monitor → Database Inspection

**AI Capabilities**:
- AI Assistant → Agent Management → Skill Management → MCP Tools → Semantic Layer

**Operations & Security**:
- Server Management → Audit Logs → AI Security → File & Storage Management

**Integration & Deployment**:
- IM Bot → Authentication & Subscription → License Management → System Settings → Deployment Guide
