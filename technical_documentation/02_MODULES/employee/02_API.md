# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1`
**Scope**: Most endpoints are nested under `/companies/{cid}/branches/{bid}/employees` to enforce strict tenant isolation and logical data partitioning.
{% endhint %}

{% hint style="danger" %}
**Critical:** Access to these endpoints requires the `SCOPE_EMPLOYEE_READ` or `SCOPE_EMPLOYEE_WRITE` permission. Unauthorized access attempts are logged and flagged for security review.
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

Fetches the full profile, including standard fields, active department assignments, and job title details.

*   **Success Response** (`200 OK`):
    ```json
    {
      "id": "e58ed763-928c-4155-bee9-fdbaaadc15f3",
      "firstName": "John",
      "lastName": "Doe",
      "code": "EMP-A1B2",
      "email": "john.doe@company.com",
      "jobTitle": {
        "id": "job-uuid-1",
        "name": "Senior Cashier"
      },
      "departments": [
        {
          "departmentId": "dept-sales-uuid",
          "departmentName": "Sales",
          "isPrimary": true,
          "assignmentType": "PERMANENT"
        },
        {
          "departmentId": "dept-inventory-uuid",
          "departmentName": "Inventory",
          "isPrimary": false,
          "assignmentType": "TEMPORARY"
        }
      ],
      "isActive": true
    }
    ```

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

| Exception | Index Name | Column | Purpose |
| :--- | :--- | :--- |
| `idx_employees_name` | `last_name`, `first_name` | **Composite Index**. Speeds up "Search by Name" which happens on every shift assignment screen. |
| `idx_employees_company_id` | `company_id` | **Multi-tenant filter**. Ensures all queries are scoped to the correct company database partition. |
| `idx_employees_employment_type` | `employment_type` | Filtering (e.g., "Show me all Contractors"). |

{% hint style="success" %}
**Tip:** The `idx_employees_name` index is optimized for *prefix* searches. Use queries like `last_name LIKE 'Smi%'` for maximum performance.
{% endhint %}
