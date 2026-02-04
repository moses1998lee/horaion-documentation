# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1/companies`
**Access Level**: Many of these endpoints (like `GET /`) behave differently depending on who is calling them.
*   **System Admin**: Sees everything (Global View).
*   **Company Admin**: Sees only their own company (Tenant View).
{% endhint %}

## Controller: `CompanyController`

### 1. Get All Companies

**Endpoint**: `GET /api/v1/companies`

Retrieves a paginated list of companies.

*   **Query Parameters**:

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `page` | `int` | `0` | Page number. |
| `size` | `int` | `10` | Items per page. |

*   **Success Response** (`200 OK`):
    ```json
    {
      "success": true,
      "data": {
        "content": [
          {
            "id": "e58ed763-928c-4155-bee9-fdbaaadc15f3",
            "name": "Acme Corp",
            "registrationNumber": "2023/123456/07",
            "hasCompletedOnboarding": true
          }
        ],
        "totalPages": 1,
        "totalElements": 1
      }
    }
    ```

### 2. Get All Companies (Sorted)

**Endpoint**: `GET /api/v1/companies/sorted`

Retrieves companies with flexible server-side sorting.

*   **Query Parameters**:

| Parameter | Type | Default | Example |
| :--- | :--- | :--- | :--- |
| `sortBy` | `string` | `name` | `name`, `registrationNumber` |
| `sortDirection` | `string` | `ASC` | `ASC`, `DESC` |

### 3. Get Company by ID

**Endpoint**: `GET /api/v1/companies/{id}`

Fetches detailed profile for a specific company.

*   **Success Response** (`200 OK`): `CompanyResponse`.
*   **Error Responses**:
    *   `404 Not Found`: If ID doesn't exist.

### 4. Get Company by Registration Number

**Endpoint**: `GET /api/v1/companies/registration/{registrationNumber}`

Finds a company using its official business registration ID.

*   **Success Response** (`200 OK`): `CompanyResponse`.

### 5. Search Companies

**Endpoint**: `GET /api/v1/companies/search?name={name}`

Performs a case-insensitive fuzzy search.
*   **System Admins**: Search across ALL companies.
*   **Company Users**: Only find their own company (if it matches the name).

### 6. Create Company

**Endpoint**: `POST /api/v1/companies`

Onboards a new company onto the platform.

*   **Request Body**: `CreateCompanyRequest`
    ```json
    {
      "name": "Horaion Technologies",
      "registrationNumber": "2024/999999/07"
    }
    ```
*   **Success Response** (`201 Created`): `CompanyResponse`.
*   **Error Responses**:
    *   `409 Conflict`: If `registrationNumber` matches an existing active company.

### 7. Update Company

**Endpoint**: `PUT /api/v1/companies/{id}`

Updates basic company details.

*   **Request Body**: `UpdateCompanyRequest`
    ```json
    {
      "name": "Horaion Tech Global"
    }
    ```

### 8. Complete Onboarding

**Endpoint**: `POST /api/v1/companies/{id}/complete-onboarding`

Finalizes the setup process. This is usually called after the user has defined their first Branch and Department.

{% hint style="success" %}
**Tip / Success:**
**Lifecycle Application**: This endpoint flips the `hasCompletedOnboarding` flag to `true`. The UI uses this flag to decide whether to show the "Setup Wizard" or the main Dashboard.
{% endhint %}

*   **Success Response** (`200 OK`): Returns updated entity.

### 9. Delete Company

**Endpoint**: `DELETE /api/v1/companies/{id}`

Permanently removes a company.

{% hint style="danger" %}
**Critical:**
**Cascade Effect**: Deleting a company is a **destructive root action**.
It will wipe:
*   All Branches
*   All Departments
*   All Employees
*   All Shifts & Schedules
**This cannot be undone.**
{% endhint %}

## Exception Handling

Mapped in `CompanyExceptionHandler`.

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `CompanyNotFoundException` | `404 Not Found` | ID or Reg Number invalid. |
| `DuplicateRegistrationNumberException` | `409 Conflict` | Reg Number already exists. |
