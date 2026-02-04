# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1`
**Complexity**: This module has distinct endpoints for **Definitions** (Admin) and **Answers** (User).
{% endhint %}

## Controllers

### 1. Rule Definitions (Admin)

**Endpoint**: `GET /rules/{id}/fields`

Fetches the schema required to render the configuration form on the frontend.

*   **Response**:
    ```json
    {
      "id": "uuid...",
      "name": "Maximum Daily Hours",
      "fields": [
        {
          "name": "max_hours",
          "label": "Max Hours per Shift",
          "type": "NUMBER",
          "isRequired": true,
          "uiConfig": { "min": 0, "max": 24 }
        }
      ]
    }
    ```

### 2. Rule Configuration (User)

**Endpoint**: `POST /departments/{id}/rule-answers`

Saves the specific configuration for a department.

*   **Request Body**:
    ```json
    {
      "ruleId": "uuid...",
      "fieldValues": {
        "max_hours": 8,
        "strict_mode": true
      }
    }
    ```

### 3. Inheritance (Advanced)

**Endpoint**: `GET /branches/{id}/rules`

Fetch rules configured at the Branch level, which may trickle down to Departments.

## Exception Handling

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `DuplicateRuleFieldNameException` | `409 Conflict` | Defining two fields with key `max_hours` in the same rule. |
| `RuleNotFoundException` | `404 Not Found` | Invalid Rule ID. |
| `RuleAnswerNotFoundException` | `404 Not Found` | Trying to update a non-existent configuration. |

### Example Error Responses

#### 400 Bad Request (Type Mismatch)

*   *Scenario*: Sending a String "twenty" to a field defined as `NUMBER`.

```json
{
  "success": false,
  "error": {
    "code": "INVALID_FIELD_TYPE",
    "message": "Field 'max_hours' expects composite type NUMBER, received STRING."
  }
}
```
