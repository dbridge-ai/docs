# AI Security

## 1. Overview

The AI Security module provides multi-layered security protection mechanisms for the AI Assistant, preventing sensitive data leaks and dangerous operations. The system supports data masking, risk control, input sanitization, rate limiting, and other security capabilities, with per-environment risk strategy configuration.

**Core Capabilities**:
- Security subsystem toggle management
- Data masking rules (data source level + global level)
- AI risk configuration (environment-based classification)
- Approval execution mode
- AI-powered masking rule recommendations

---

## 2. AI Security Control Panel

Go to **AI Center** > **AI Safety** to see toggles for six security subsystems:

| Subsystem | Description |
|-----------|-------------|
| Risk Control | Classifies AI-generated SQL by risk level; high-risk operations require approval |
| Data Mask | Masks sensitive fields in query results |
| pLLM (Private LLM) | Uses private models to process sensitive data, avoiding sending data to external APIs |
| Input Sanitizer | Filters potential injection attack content from user input |
| Rate Limit | Limits AI request frequency to prevent abuse |
| Audit Log | Records audit logs for all AI-related operations |

Each toggle can be independently enabled or disabled. Changes take effect immediately.

### 2.1 Strict Permission Mode

At the bottom of the page, there is a **Strict Permission Mode** toggle. When enabled, Agent tool calls are subject to stricter permission control; unauthorized operations are rejected or executed in dry-run mode.

The page displays the current denied count and dry-run count, helping administrators evaluate the impact of strict mode.

---

## 3. Data Masking Rules

Go to **AI Center** > **Mask Rules** to manage data masking rules.

### 3.1 Rule Types

**Custom Rules**: Masking rules bound to specific data sources, only effective for query results from that data source.

**Global Rules**: Universal masking rules effective for all data sources, using regex to match column names.

### 3.2 Mask Functions

| Function | Description | Example |
|----------|-------------|---------|
| phone | Phone number masking | `138****1234` |
| email | Email masking | `a***@example.com` |
| id_card | ID card number masking | `110***********1234` |
| bank_card | Bank card number masking | `6222****1234` |
| hash | Hash masking | Replaces original value with a hash |

### 3.3 Managing Custom Rules

1. **Filter**: Use the top dropdowns to filter by data source and database name
2. **Add Rule**: Click **Add Rule** and fill in:
   - Column name (required)
   - Mask function (required)
   - Database name (optional; applies to all databases if left blank)
   - Schema name (optional)
   - Table name (optional)
3. **Enable/Disable**: Toggle the status switch to control whether the rule is active
4. **Edit**: Modify existing rule configuration
5. **Delete**: Remove rules no longer needed
6. **Reset Defaults**: Click **Reset Defaults** to reset all custom rules to system defaults

### 3.4 Managing Global Rules

Global rules use regex to match column names, suitable for universal sensitive fields (e.g., all columns containing `password`).

1. In the global rules area, enter the column name regex
2. Select a mask function
3. Click **Add Global Rule**

### 3.5 AI Recommendations

The system supports AI-powered automatic analysis of database table structures to recommend masking rules.

**Steps**:

1. Click the **AI Recommend** button
2. In the dialog, select:
   - **Agent**: AI Agent for analysis
   - **Data Source**: Target data source
   - **Database**: Target database
   - **Schema**: Target Schema
   - **Custom Instructions** (optional): Additional analysis requirements
3. Click **Generate Mask Rules**
4. The system displays analysis progress ("Analyzing schema", "Scanning columns", "Generating rules")
5. After completion, recommended rules are automatically added to the custom rules list

> **Note**: The AI recommendation process may take 1-3 minutes depending on the number of tables in the database.

---

## 4. Risk Control Configuration

The risk control feature determines whether to allow direct execution or require approval based on the risk level of SQL operations.

### 4.1 Risk Levels

| Level | Description | Examples |
|-------|-------------|----------|
| L1 (Low Risk) | Safe operations, execute directly | SELECT, SHOW, DESCRIBE |
| L2 (Medium Risk) | Cautious operations, recorded in audit log | INSERT, UPDATE (conditional) |
| L3 (High Risk) | Dangerous operations, require approval | DROP TABLE, unconditional DELETE, TRUNCATE |

### 4.2 Per-Environment Configuration

Risk configuration can be set separately by data source environment labels:

- **Development**: Relaxed restrictions, allowing more operations
- **Testing**: Moderate restrictions
- **Production**: Strict restrictions, high-risk operations must be approved

### 4.3 Approval Execution

When AI-generated SQL is classified as high-risk:

1. The system marks the SQL as "pending approval"
2. Administrators see the approval prompt in the AI Assistant interface
3. After reviewing the SQL content, choose "Approve" or "Reject"
4. Only approved SQL is sent to the database for execution

---

## 5. Execution Modes

The AI Assistant supports three execution modes, configured in Agent settings:

| Mode | Description |
|------|-------------|
| Auto | AI automatically judges and executes; suitable for development environments |
| Approval | All write operations require human approval |
| YOLO | Full trust in AI with no restrictions; for testing only |

---

## 6. Best Practices

1. **Enable data masking in production**: Prevent sensitive data from leaking through AI conversations
2. **Configure global rules**: Set global masking for common sensitive fields (passwords, ID cards, phone numbers)
3. **Use AI recommendations**: After creating a new data source, use AI recommendations to quickly generate masking rules
4. **Enable approval mode in production**: Prevent data loss from AI misoperations
5. **Regularly review audit logs**: Check AI operation records to detect abnormal behavior
6. **Enable strict permission mode**: Restrict Agents to only authorized tools and data sources
