# API Reference

{% hint style="warning" %}
**Security Notice:**
All shift preference endpoints require authentication. Controller-level Permit checks use the `shift_preference` resource, and service-level checks enforce self-service or department scope before data is queried.
{% endhint %}

## Controller: `ShiftPreferenceController`

**Base Path**: `/api/v1/employees/{employeeId}/shift-preferences`

This controller manages the authenticated employee's own shift preferences.

### 1. Get Employee Shift Preferences

**Endpoint**: `GET /`

Returns active preferences for the employee, grouped by `(employee, date, department)`.

*   **Permissions**: `read:shift_preference`
*   **Access**: The authenticated employee only.
*   **Response**: `200 OK` with `List<ShiftPreferenceResponse>`

```json
{
  "success": true,
  "data": [
    {
      "employeeId": "2b7b4d10-f3d2-4e90-9ce2-2c442f97c000",
      "employeeName": "Jane Tan",
      "date": "2026-07-06",
      "departmentId": "f26c3b5b-0f83-4bb0-a4d2-8f25f72ec000",
      "departmentName": "Front Office",
      "shifts": [
        {
          "shiftId": "77b7f0e6-2101-4f02-9470-2c676f54c000",
          "shiftLabel": "Morning"
        }
      ]
    }
  ],
  "error": null
}
```

### 2. Submit Shift Preferences

**Endpoint**: `POST /`

Creates or updates the employee's preferred shifts for one date in one department. The submitted `shiftIds` become the exact active set for that `(employee, date, department)` group.

*   **Permissions**: `create:shift_preference`
*   **Access**: The authenticated employee only.
*   **Request Body**: `CreateShiftPreferenceRequest`
*   **Response**: `200 OK` with `ShiftPreferenceResponse`

```json
{
  "date": "2026-07-06",
  "departmentId": "f26c3b5b-0f83-4bb0-a4d2-8f25f72ec000",
  "shiftIds": [
    "77b7f0e6-2101-4f02-9470-2c676f54c000",
    "7ac0a87a-9487-4f59-a44b-51b40139c000"
  ]
}
```

{% hint style="info" %}
**Reconciliation:**
If a previously preferred shift is missing from `shiftIds`, that row is soft-deleted. If a new shift is included, a new active row is inserted.
{% endhint %}

### 3. Delete a Date/Department Preference Group

**Endpoint**: `DELETE /{date}?departmentId={departmentId}`

Soft-deletes every active preference row for the employee on the given date and department.

*   **Permissions**: `delete:shift_preference`
*   **Access**: The authenticated employee only.
*   **Path Parameters**:
    *   `date` (`yyyy-MM-dd`): Preference date.
*   **Query Parameters**:
    *   `departmentId` (UUID): Department for the preference group.
*   **Response**: `200 OK`

```http
DELETE /api/v1/employees/{employeeId}/shift-preferences/2026-07-06?departmentId=f26c3b5b-0f83-4bb0-a4d2-8f25f72ec000
Authorization: Bearer {token}
```

## Controller: `DepartmentShiftPreferenceController`

**Base Path**: `/api/v1/departments/{departmentId}/shift-preferences`

This controller provides read-only department visibility for scheduling workflows.

### 1. Get Department Shift Preferences

**Endpoint**: `GET /`

Returns active shift preferences for all employees in the department, grouped by `(employee, date, department)`.

*   **Permissions**: `read:shift_preference`
*   **Access**: System administrators and system owners may read any department; privileged and super-privileged department leads may read only departments they lead.
*   **Response**: `200 OK` with `List<ShiftPreferenceResponse>`

```http
GET /api/v1/departments/{departmentId}/shift-preferences
Authorization: Bearer {token}
```

## Companion Endpoint: `MeController`

**Endpoint**: `GET /api/v1/me/department-assignments`

Returns departments assigned to the authenticated employee. This is a self-scoped helper for account-level flows and does not require the elevated `GET /me/departments` role restriction.

*   **Permissions**: Authenticated.
*   **Response**: `200 OK` with `List<DepartmentResponse>`

## DTO Reference

### `CreateShiftPreferenceRequest`

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `date` | LocalDate | Yes | Date the employee prefers to work. |
| `departmentId` | UUID | Yes | Department that owns the submitted shifts. |
| `shiftIds` | List<UUID> | Yes | Non-empty list of preferred shifts for the date and department. |

### `ShiftPreferenceResponse`

| Field | Type | Description |
| :--- | :--- | :--- |
| `employeeId` | UUID | Employee who owns the preference group. |
| `employeeName` | String | Employee full name. |
| `date` | LocalDate | Preference date. |
| `departmentId` | UUID | Department for the preferred shifts. |
| `departmentName` | String | Department display name. |
| `shifts` | List<PreferredShiftResponse> | Preferred shifts in the group, sorted by shift label. |

### `PreferredShiftResponse`

| Field | Type | Description |
| :--- | :--- | :--- |
| `shiftId` | UUID | Preferred shift ID. |
| `shiftLabel` | String | Preferred shift label. |

## Exception Handling

| Error Code | HTTP Status | Trigger |
| :--- | :--- | :--- |
| `SHIFT_DEPARTMENT_MISMATCH` | `400 Bad Request` | A submitted shift does not belong to the submitted department. |
| `EMPLOYEE_NOT_FOUND` | `404 Not Found` | The employee path ID does not resolve to an employee. |
| `SHIFT_PREFERENCE_NOT_FOUND` | `404 Not Found` | Delete was requested for an empty `(employee, date, department)` group. |
| Shift not found | `404 Not Found` | A submitted shift ID does not resolve to an active shift. |
| Authorization failure | `403 Forbidden` | Permit.io or service-level scope check denies access. |
