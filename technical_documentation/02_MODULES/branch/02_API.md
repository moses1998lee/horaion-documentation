# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1/companies/{companyId}/branches`
All endpoints require a valid `companyId` in the path. This reinforces our multi-tenant architecture.
{% endhint %}

## Controller: `BranchController`

### 1. Get All Branches

**Endpoint**: `GET /api/v1/companies/{companyId}/branches`

Retrieves a paginated list of branches for a specific company.

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
            "id": "a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11",
            "name": "Main Office",
            "code": "HQ-001",
            "city": "Cape Town",
            "isActive": true
          }
        ],
        "totalPages": 5,
        "totalElements": 50
      }
    }
    ```

### 2. Get Branch by ID

**Endpoint**: `GET /api/v1/companies/{companyId}/branches/{id}`

Fetches a single branch's detailed information.

*   **Success Response** (`200 OK`):
    ```json
    {
      "id": "a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11",
      "name": "Main Office",
      "code": "HQ-001",
      "addressLine1": "123 Long Street",
      "city": "Cape Town",
      "country": "South Africa",
      "timezone": "Africa/Johannesburg",
      "isHeadquarters": true,
      "managerId": "b2f..."
    }
    ```
*   **Error Responses**:
    *   `404 Not Found`: If the branch does not exist or belongs to another company.

### 3. Create Branch

**Endpoint**: `POST /api/v1/companies/{companyId}/branches`

Creates a new physical branch location.

*   **Request Body**: `CreateBranchRequest`
    ```json
    {
      "name": "Durban Central",
      "code": "DBN-001",
      "addressLine1": "45 West Street",
      "city": "Durban",
      "isHeadquarters": false,
      "timezone": "Africa/Johannesburg"
    }
    ```
*   **Success Response** (`201 Created`): Returns the created `BranchResponse`.
*   **Error Responses**:
    *   `409 Conflict`: If a branch with the same `code` already exists in this company.
    *   `400 Bad Request`: Validation errors (e.g., missing name).

### 4. Update Branch

**Endpoint**: `PUT /api/v1/companies/{companyId}/branches/{id}`

Updates details of an existing branch.

*   **Request Body**: `UpdateBranchRequest`
    ```json
    {
      "name": "Durban Central - Renovated",
      "phoneNumber": "+27315551234"
    }
    ```

### 5. Delete Branch (Soft Delete)

**Endpoint**: `DELETE /api/v1/companies/{companyId}/branches/{id}`

Marks a branch as deleted (`softDelete = true`). It does not physically remove the record to preserve historical data.

{% hint style="danger" %}
**Critical:**
**Data Safety**: We rarely hard-delete branches because Shifts and Employees are linked to them. A hard delete would break historical reports.
{% endhint %}

## Exception Handling

Common exceptions mapped in `BranchExceptionHandler`.

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `BranchNotFoundException` | `404 Not Found` | Branch ID does not exist. |
| `DuplicateBranchCodeException` | `409 Conflict` | Branch Code is already used. |
| `CompanyNotFoundException` | `404 Not Found` | Company ID invalid. |
