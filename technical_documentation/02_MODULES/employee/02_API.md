# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1`
**Scope**: Most endpoints are nested under `/companies/{cid}/branches/{bid}/employees` to enforce strict tenant isolation.
{% endhint %}

## Controller: `EmployeeController`

### 1. Get All Employees (Branch Scoped)

**Endpoint**: `GET .../employees`

Retrieves a paginated list of employees for specific branch.

*   **Query Parameters**: `page`, `size`.
*   **Security**:
    *   **System Owner**: Sees all.
    *   **Manager**: Sees only employees in their allowed departments.

### 2. Get Employee by ID

**Endpoint**: `GET .../employees/{id}`

Fetches the full profile, including:
*   Standard fields (Name, Email).
*   Active Department Assignments.
*   Job Title details.

### 3. Assign to Department

**Endpoint**: `POST .../employees/{id}/departments`

Links an employee to a functional unit.

*   **Request Body**: `AssignEmployeeToDepartmentRequest`
    ```json
    {
      "departmentId": "uuid-for-sales",
      "isPrimary": true,
      "assignmentType": "PERMANENT",
      "effectiveFrom": "2024-01-01"
    }
    ```
    {% hint style="success" %}
    **Tip / Success:**
    **Primary Swap**: If you assign a new `Primary` department, the system usually auto-downgrades the *old* primary department to `Secondary`.
    {% endhint %}

### 4. Assign Job Title

**Endpoint**: `PUT /api/v1/employees/{id}/job-title`

Promotes or changes an employee's rank.

*   **Side Effect**: This triggers a call to AWS Cognito to update the user's Group (e.g., adding them to `MANAGERS` group).

### 5. Remove from Department

**Endpoint**: `DELETE .../employees/{id}/departments/{deptId}`

Ends an assignment.
*   **Logic**: This is a **Soft Delete** on the assignment table. It sets `effectiveUntil = NOW()` and `isActive = false`, preserving the history that "Alice worked in Sales from Jan to March".

### 6. Get Department Assignments

**Endpoint**: `GET .../employees/{id}/departments`

Returns the history (or current list) of where this person works.

## Exception Handling

Mapped in `EmployeeExceptionHandler`.

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `EntityNotFoundException` | `404 Not Found` | Employee or Department ID invalid. |
| `DuplicateDepartmentAssignmentException` | `409 Conflict` | Employee is already active in this department. |
