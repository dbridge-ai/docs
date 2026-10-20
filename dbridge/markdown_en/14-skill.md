# Skill Management

Skills are reusable AI prompt templates that define how specific tasks are executed. Skills can bind tools, define input variables, and be called by Agents.

## Features

- Create and manage AI Skills
- Prompt templates with input variables
- Tool binding
- Version management and rollback
- Test runs
- Import/Export
- AI-assisted generation

## Skill List

The "Skill Management" page displays all Skills in a card grid:

| Info | Description |
|:-----|:------------|
| Name | Skill name |
| Icon | Skill icon |
| Category | general / analysis / optimization / migration |
| Description | Skill capability description |

## Creating a Skill

Click the "New" button and fill in the Skill configuration:

### Basic Information

| Field | Description |
|:------|:------------|
| Name | Skill name |
| Slug | Unique identifier (URL-friendly) |
| Description | Capability description |
| Icon | Select an icon |
| Category | general / analysis / optimization / migration |
| Enabled | Whether the Skill is enabled |

### Prompt Configuration

| Field | Description |
|:------|:------------|
| Prompt Template | Skill's Prompt template with variable interpolation |
| System Prompt | System-level instructions |
| Input Variables | Variable list used in the template |
| Output Format | markdown / json / yaml / text |

**Prompt Template Example**:

```
Please analyze the performance issues of the following SQL query:

SQL: {{sql}}
Database Type: {{db_type}}

Please provide:
1. Performance issue diagnosis
2. Optimization suggestions
3. Optimized SQL
```

### Tool Binding

Select MCP tools that can be called during Skill execution.

### Script Files & Resources

- **Script Files** — Associated SQL script files
- **Resources** — Additional resource files

## Categories

| Category | Description | Examples |
|:---------|:------------|:---------|
| general | General skills | SQL explanation, code generation |
| analysis | Analysis skills | Data insights, trend analysis |
| optimization | Optimization skills | SQL optimization, index suggestions |
| migration | Migration skills | DDL conversion, data migration |

## Test Run

After creating or editing a Skill, click "Test Run" to verify the results:

1. Fill in the input variable values
2. Click "Run"
3. View execution results:
   - Tool call traces
   - Output content
   - Execution duration

## Version Management

Every save creates a new version:

1. Click the "Version History" button
2. View all historical versions
3. Rollback to any historical version

> Version management ensures safe rollback when modifying Skills.

## Import / Export

### Exporting Skills

Export Skills as a ZIP file containing configuration and associated resources:

1. Select the Skills to export
2. Click "Export"
3. Download the ZIP file

### Importing Skills

Import Skills from a ZIP file:

1. Click the "Import" button
2. Select the ZIP file
3. Confirm import

> Import/export is ideal for sharing Skill configurations between teams.

## AI Generation

The system supports automatically generating Skills via AI:

1. Click the "AI Generate" button
2. Enter a Skill description
3. AI automatically generates the prompt template, input variables, and output format

## Built-in Skills

The system comes with several pre-configured Skills:

| Skill | Category | Description |
|:------|:---------|:------------|
| SQL Explain | general | Translate SQL into natural language |
| SQL Optimize | optimization | Analyze performance and provide optimization suggestions |
| Data Insights | analysis | Analyze query results and generate insights |
| DDL Convert | migration | Convert DDL between different database dialects |

Built-in Skills cannot be edited or deleted, but you can create custom Skills based on them.
