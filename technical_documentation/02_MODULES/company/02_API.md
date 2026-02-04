# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1/companies`
Unlike other modules, some of these endpoints (like `create`) might be accessible to System Admins only.
{% endhint %}

## Controller: `CompanyController`

### 1. Get All Companies

**Endpoint**: `GET /api/v1/companies`

Retrieves a paginated list of companies.
*   **Behavior**:
    *   **System Admin**: Sees ALL companies.
    *   **Company Admin**: Sees ONLY their own company.

*   **Parameters**:
    *   `page`: Page number (default: 0)
    *   `size`: Items per page (default: 10)
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

*   **Parameters**:
    *   `sortBy`: Field to sort by (default: `name`).
    *   `sortDirection`: `ASC` or `DESC`.

### 3. Get Company by ID

**Endpoint**: `GET /api/v1/companies/{id}`

Fetches detailed profile for a specific company.

*   **Success Response** (`200 OK`): `CompanyResponse`.
*   **Error Responses**:
    *   `404 Not Found`: If ID doesn't exist.

### 4. Get Company by Registration Number

**Endpoint**: `GET /api/v1/companies/registration/{registrationNumber}`

Finds a company using its official business registration ID.

*   **Parameters**:
    *   `registrationNumber`: The string ID (e.g., `2023/001`).
*   **Success Response** (`200 OK`): `CompanyResponse`.

### 5. Search Companies

**Endpoint**: `GET /api/v1/companies/search?name={name}`

Performs a case-insensitive fuzzy search on the company name.

*   **Success Response** (`200 OK`): A list of matching `CompanyResponse` objects.

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
    *   `409 Conflict`: If `registrationNumber` is already taken.

### 7. Update Company

**Endpoint**: `PUT /api/v1/companies/{id}`

Updates basic company details (currently just the name).

*   **Request Body**: `UpdateCompanyRequest`
    ```json
    {
      "name": "Horaion Tech Global"
    }
    ```

### 8. Complete Onboarding

**Endpoint**: `POST /api/v1/companies/{id}/complete-onboarding`

Marks the company's initial setup as complete. This might unlock features or remove "Setup Guide" nags in the UI.

{% hint style="success" %}
**Tip / Success:**
This is a dedicated lifecycle event endpoint. Instead of a generic update, this explicit action clearly signals a state change in the company's lifecycle.
{% endhint %}

*   **Success Response** (`200 OK`): Returns updated entity with `hasCompletedOnboarding: true`.

### 9. Delete Company

**Endpoint**: `DELETE /api/v1/companies/{id}`

Permanently removes a company.

{% hint style="danger" %}
**Critical:**
**Cascade Effect**: Deleting a company is a destructive action that should theoretically cascade delete ALL branches, employees, and data associated with it. Use with extreme caution.
{% endhint %}

## Exception Handling

Mapped in `CompanyExceptionHandler`.

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `CompanyNotFoundException` | `404 Not Found` | ID or Reg Number invalid. |
| `DuplicateRegistrationNumberException` | `409 Conflict` | Reg Number already exists. |
