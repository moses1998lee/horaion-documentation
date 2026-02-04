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

*   **Query Parameters**:

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `page` | `int` | `0` | The page number to retrieve (0-indexed). |
| `size` | `int` | `10` | The number of records per page. |

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
            "isActive": true,
            "type": "HEADQUARTERS"
          }
        ],
        "totalPages": 5,
        "totalElements": 50
      }
    }
    ```

### 2. Get All Branches (Sorted)

**Endpoint**: `GET /api/v1/companies/{companyId}/branches/sorted`

Retrieves a list of branches with server-side sorting.

*   **Query Parameters**:

| Parameter | Type | Default | Example |
| :--- | :--- | :--- | :--- |
| `sort` | `string` | `name` | `name,asc` or `city,desc` |

{% hint style="success" %}
**Tip / Success:**
**Sorting**: You can sort by any top-level field, such as `name`, `code`, `city`, `country`, or `createdAt`.
{% endhint %}

### 3. Get Active Branches

**Endpoint**: `GET /api/v1/companies/{companyId}/branches/active`

Retrieves **only** branches that are currently active (`isActive = true`).
**Use Case**: Populating "Select Branch" dropdowns in forms (you shouldn't let users select a closed branch).

*   **Success Response** (`200 OK`): Returns a list of `BranchResponse`.

### 4. Get Branch by Code

**Endpoint**: `GET /api/v1/companies/{companyId}/branches/code/{branchCode}`

Finds a specific branch using its unique local code. This is useful for integrations where you might know the code (e.g., `NYC-01`) but not the internal UUID.

*   **Path Parameters**:
    *   `branchCode`: The unique string identifier (e.g., `HQ-001`).
*   **Error Responses**:
    *   `404 Not Found`: If no branch with that code exists *in this company*.

### 5. Create Branch

**Endpoint**: `POST /api/v1/companies/{companyId}/branches`

Creates a new physical or virtual branch location.

*   **Request Body**: `CreateBranchRequest`
    ```json
    {
      "name": "Durban Central",
      "code": "DBN-001",
      "type": "RETAIL",
      "addressLine1": "45 West Street",
      "city": "Durban",
      "country": "South Africa",
      "timezone": "Africa/Johannesburg",
      "latitude": -29.8587,
      "longitude": 31.0218,
      "isHeadquarters": false,
      "openingHours": {
        "mon": "08:00-17:00",
        "tue": "08:00-17:00"
      }
    }
    ```
*   **Success Response** (`201 Created`): Returns the created `BranchResponse`.
*   **Error Responses**:
    *   `409 Conflict`: If `code` "DBN-001" already exists for this company.
    *   `400 Bad Request`: If constraints are violated (e.g., name too long).

### 6. Update Branch

**Endpoint**: `PUT /api/v1/companies/{companyId}/branches/{id}`

Updates details of an existing branch.

*   **Request Body**: `UpdateBranchRequest`
    ```json
    {
      "name": "Durban Central - Renovated",
      "phoneNumber": "+27315551234",
      "isActive": false
    }
    ```
{% hint style="info" %}
**Note:**
**Deactivating vs Deleting**: You can set `isActive: false` here to temporarily close a branch (e.g., for renovations). This is different from Soft Delete (permanent closure).
{% endhint %}

### 7. Delete Branch (Soft Delete)

**Endpoint**: `DELETE /api/v1/companies/{companyId}/branches/{id}`

Marks a branch as deleted (`softDelete = true`).

{% hint style="danger" %}
**Critical:**
**Data Safety**: This operation is **irreversible** via the API. Once soft-deleted, the branch will disappear from all standard `GET` requests. It remains in the database solely for reporting integrity (so old payslips don't break).
{% endhint %}

## Exception Handling

Common exceptions mapped in `BranchExceptionHandler`.

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `BranchNotFoundException` | `404 Not Found` | Branch ID does not exist or is soft-deleted. |
| `DuplicateBranchCodeException` | `409 Conflict` | Branch Code is already used in this company. |
| `CompanyNotFoundException` | `404 Not Found` | Company ID invalid. |
