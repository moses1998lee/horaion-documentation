# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1/companies/{companyId}/job-titles`
**Scope**: Job Titles are defined at the **Company** level, not the Branch level. This allows a "Store Manager" role to be reused across hundreds of branches.
{% endhint %}

## Controller: `JobTitleController`

### 1. Get All Job Titles

**Endpoint**: `GET .../job-titles`

Retrieves a paginated list of available roles.

*   **Use Case**: Populating the "Select Job Title" dropdown on the Employee Onboarding screen.

### 2. Create Job Title

**Endpoint**: `POST .../job-titles`

Defines a new role and provisions security context.

*   **Request Body**:
    ```json
    {
      "name": "District Manager",
      "description": "Oversees clustered branches."
    }
    ```

*   **Side Effects**:
    1.  **DB**: Inserts row into `job_titles`.
    2.  **AWS**: Calls `cognito:CreateGroup` with name `District Manager`.

{% hint style="warning" %}
**Important / Warning:**
**Uniqueness**: The `name` must be unique within the Company. You cannot have two "Manager" titles.
{% endhint %}

### 3. Get Job Title by ID

**Endpoint**: `GET .../job-titles/{id}`

fetches details for a specific role.

## Limitations

*   **No Update/Delete**: As observed in the current version (`v1`), there are no endpoints to Update or Delete a Job Title.
    *   *Workaround*: To "Remove" a title, you currently must contact the System Administrator to perform a soft-delete at the database level, as the API does not expose this to prevent accidental permission lockout.

### Example Error Responses

#### 409 Conflict (Duplicate Name)

Tried to create "Store Manager" when it already exists.

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "JOB_TITLE_ALREADY_EXISTS",
    "message": "Job title 'Store Manager' already exists for company 12345."
  }
}
```

#### 403 Forbidden (Cross-Tenant)

Tried to fetch Job Titles for a Company you do not belong to.

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ACCESS_DENIED",
    "message": "User does not have permission to access company 99999."
  }
}
```
