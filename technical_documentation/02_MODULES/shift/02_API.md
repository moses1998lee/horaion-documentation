# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1/departments/{departmentId}/shifts`
**Scope**: All shifts are scoped to a specific **Department**.
{% endhint %}

## Shift Management

### 1. Create Shift Template

**Endpoint**: `POST .../shifts`

Defines a new shift pattern and its staffing requirements.

*   **Request Body**:
    ```json
    {
      "label": "Morning Rush",
      "type": "OPENING",
      "startTime": "07:00",
      "endTime": "15:00",
      "daysAppliedTo": [1, 2, 3, 4, 5], // Mon-Fri
      "colorCode": "#FF5733",
      "shiftRoles": [
        "uuid-of-manager-role", // Implicitly required count = 1
        "uuid-of-cashier-role",
        "uuid-of-cashier-role"  // Listed twice = required count 2
      ]
    }
    ```

### 2. Get Active Shifts

**Endpoint**: `GET .../shifts/active`

Returns the "Menu" of currently available shifts for the solver to use.

### 3. Update Shift

**Endpoint**: `PUT .../shifts/{id}`

Updates the definition.

{% hint style="warning" %}
**Important / Warning:**
**Impact**: Changing a Shift Definition (e.g., changing 08:00 to 09:00) **does NOT** automatically update past or already-published schedules. It only affects future schedule generation.
{% endhint %}

## Exception Handling

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `DuplicateShiftLabelException` | `409 Conflict` | Trying to create "Morning" shift when one already exists for this department. |
| `ShiftNotFoundException` | `404 Not Found` | Invalid Shift ID. |

### Example Error Responses

#### 409 Conflict (Duplicate Label)

```json
{
  "success": false,
  "error": {
    "code": "DUPLICATE_SHIFT_LABEL",
    "message": "Shift with label 'Morning Rush' already exists for this department."
  }
}
```
