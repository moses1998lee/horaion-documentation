# API Reference

{% hint style="warning" %}
**Security Notice:**
All endpoints in this module are protected by **Role Based Access Control (RBAC)**.
You must have the appropriate permissions (e.g., `create:shift`, `read:shift`) to access these resources. Access is also strictly scoped to the `departmentId` provided in the path.
{% endhint %}

## Controller: `ShiftController`

**Base Path**: `/api/v1/departments/{departmentId}/shifts`

This controller manages the lifecycle of Shift Templates.

### 1. Retrieve Shifts (Paginated)

**Endpoint**: `GET /`

Retrieves a paginated list of all shifts for a specific department.

*   **Permissions**: `read:shift`
*   **Parameters**:
    *   `page` (query, int): Page number (default: 0)
    *   `size` (query, int): Items per page (default: 10)
*   **Response**: `200 OK` with `Page<ShiftResponse>`

### 2. Create Shift

**Endpoint**: `POST /`

Creates a new Shift Template definition.

*   **Permissions**: `create:shift`
*   **Request Body**: `CreateShiftRequest`

    ```json
    {
      "label": "Morning Standard",
      "type": "MORNING",
      "startTime": "09:00:00",
      "endTime": "17:00:00",
      "daysAppliedTo": [1, 2, 3, 4, 5],
      "colorCode": "#FF5733",
      "shiftRoles": [
        "a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11", 
        "b1ffcd22-8d1a-5fe9-cc7e-7cc8ce491b22"
      ]
    }
    ```

    {% hint style="info" %}
    **Note on daysAppliedTo**:
    1 = Monday, 7 = Sunday. The array `[1, 2, 3, 4, 5]` represents a standard Monday-Friday work week.
    {% endhint %}

*   **Response**: `201 Created`

### 3. Get Shift by ID

**Endpoint**: `GET /{id}`

Retrieves full details for a single shift template.

*   **Permissions**: `read:shift`
*   **Response**: `200 OK`

    ```json
    {
      "id": "c2ggde00-0e1b-6ff0-dd8f-8dd9df502c33",
      "label": "Morning Standard",
      "shiftType": "MORNING",
      "startTime": "09:00:00",
      "endTime": "17:00:00",
      "shiftRoles": ["..."],
      "isActive": true
    }
    ```

### 4. Update Shift

**Endpoint**: `PUT /{id}`

Updates an existing shift.

*   **Permissions**: `update:shift`
*   **Request Body**: `UpdateShiftRequest` (Same structure as Create)
*   **Response**: `200 OK`

### 5. Delete Shift

**Endpoint**: `DELETE /{id}`

Performs a **HARD DELETE** on the shift.

{% hint style="danger" %}
**Critical Warning:**
This operation is **irreversible**. It permanently removes the Shift template and its associated Role constraints from the database.
Ensure the shift is not actively being used in upcoming schedules before deletion.
{% endhint %}

*   **Permissions**: `delete:shift`
*   **Response**: `204 No Content`

### 6. Utility Endpoints

| Endpoint | Method | Description |
| :--- | :--- | :--- |
| `/active` | `GET` | Returns a simple list (not paginated) of all **Active** shifts. Useful for dropdowns. |
| `/roles` | `GET` | Returns a distinct list of all Employee Roles currently used in this department's shifts. |
| `/list` | `GET` | Returns a simple list of all shifts (including inactive). |
