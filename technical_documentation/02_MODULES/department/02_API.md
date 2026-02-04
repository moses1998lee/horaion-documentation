# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1/companies/{companyId}/branches/{branchId}/departments`
All endpoints require a valid `companyId` AND `branchId` in the path. This strict scoping ensures data isolation.
{% endhint %}

## Controller: `DepartmentController`

### 1. Get All Departments

**Endpoint**: `GET .../departments`

Retrieves a paginated list of departments for a specific branch.

*   **Query Parameters**:

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `page` | `int` | `0` | Page number. |
| `size` | `int` | `10` | Items per page. |

### 2. Get Department by ID

**Endpoint**: `GET .../departments/{id}`

Fetches detailed information about a department, including its recursive parent (if any).
*   **Response includes**: `parentId`, `parentName`, `headOfDeptId`, `headOfDeptName`.

### 3. Get Active Departments

**Endpoint**: `GET .../departments/active`

Retrieves all departments where `isActive = true`.
*   **Use Case**: Populating "Select Department" lists in the Scheduler UI.
*   **Note**: This endpoint is usually unpaginated as the list of departments is rarely massive per branch.

### 4. Get Roles in Department

**Endpoint**: `GET .../departments/{id}/roles`

Returns a list of all Employee Roles (e.g., "Cashier", "Manager") that are associated with this department.
*   **Use Case**: When scheduling a shift for the "Bakery" department, you only want to see "Bakers", not "Accountants".

### 5. Create Department

**Endpoint**: `POST .../departments`

Creates a new functional unit.

*   **Request Body**: `CreateDepartmentRequest`
    ```json
    {
      "name": "Butchery",
      "code": "BTC-01",
      "description": "Meat processing and sales",
      "parentId": "optional-uuid-of-parent",
      "costCenter": "CC-900",
      "budget": 50000.00
    }
    ```
*   **Success Response** (`201 Created`): `DepartmentResponse`.
*   **Error Responses**:
    *   `409 Conflict`: If `code` "BTC-01" already exists in this Branch.

### 6. Assign Head of Department

**Endpoint**: `PUT .../departments/{id}/head`

Promotes an employee to be the manager of this department.

{% hint style="warning" %}
**Important / Warning:**
**Privilege Escalation**: This action triggers a security event. The assigned employee is added to the `PRIVILEGED_SYSTEM_USER` group in AWS Cognito, granting them access to Manager-level dashboards.
{% endhint %}

*   **Request Body**: `AssignHeadOfDepartmentRequest`
    ```json
    {
      "employeeId": "uuid-of-new-manager"
    }
    ```

### 7. Delete Department

**Endpoint**: `DELETE .../departments/{id}`

Permanently removes a department.

{% hint style="danger" %}
**Critical:**
**Constraint Violation**: You likely cannot delete a department if it still has active Employees or Shifts assigned to it. You must reassign those resources first.
{% endhint %}

## Exception Handling

Mapped in `DepartmentExceptionHandler`.

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `DepartmentNotFoundException` | `404 Not Found` | ID invalid. |
| `DuplicateDepartmentCodeException` | `409 Conflict` | Code already used in this branch. |
