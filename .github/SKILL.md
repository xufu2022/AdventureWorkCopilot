---
name: sql-test-generator
description: Generates xUnit tests for SQL stored procedures and functions
version: 1.0.0
triggers:
  - "/generate-sp"
  - "/generate-function"
  - "/generate-all"
  - "generate test for stored procedure"
  - "create xunit test for"
---

# SQL Test Generator Skill

## Paths
- DatabaseProject path: `../DatabaseProject`
- TestProject path: `../TestProject`
- Templates path: `.copilot/templates/`
- Connection string key: `ConnectionStrings:DefaultConnection`

## Execution Logic

### Parse Request
Extract the database object name from user input.

### Locate Object
Search order:
1. `DatabaseProject/StoredProcedures/{name}.sql`
2. `DatabaseProject/Functions/{name}.sql`

If not found: Return error "Object not found in DatabaseProject"

### Determine Operation Type

**READ Pattern:**
- SQL contains SELECT
- SQL does NOT contain INSERT, UPDATE, or DELETE

**WRITE Pattern:**
- SQL contains INSERT, UPDATE, or DELETE

**FUNCTION Pattern:**
- SQL contains CREATE FUNCTION

### Apply Template

Copy from appropriate template file:
- READ → `.copilot/templates/read-sp-template.md`
- WRITE → `.copilot/templates/write-sp-template.md`
- FUNCTION → `.copilot/templates/function-template.md`

Replace placeholders:
- `[SP_NAME]` → Stored procedure name
- `[FUNCTION_NAME]` → Function name
- `[PROJECT_NAMESPACE]` → "TestProject"

### Output Files

**For Stored Procedure:**
- Test class: `TestProject/StoredProcedures/{name}_Tests.cs`
- JSON data: `TestProject/TestCases/{name}_Tests.json` (if READ)

**For Function:**
- Test class: `TestProject/Functions/{name}_Tests.cs`

### Generate JSON Test Cases (READ only)

Create file with minimum 3 test cases:

```json
{
  "testCases": [
    {
      "id": 1,
      "name": "ValidInput_ReturnsData",
      "description": "Returns data when valid input provided",
      "parameters": {
        "@UserId": 1
      },
      "expectedRowCount": 1
    },
    {
      "id": 2,
      "name": "InvalidInput_ReturnsEmpty",
      "description": "Returns empty set when invalid input provided",
      "parameters": {
        "@UserId": 99999
      },
      "expectedRowCount": 0
    },
    {
      "id": 3,
      "name": "NullParameter_HandlesGracefully",
      "description": "Handles NULL parameter without error",
      "parameters": {
        "@UserId": null
      },
      "expectedRowCount": 0
    }
  ]
}