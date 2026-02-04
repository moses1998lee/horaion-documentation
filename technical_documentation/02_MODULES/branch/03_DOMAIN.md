# Branch Module Domain Logic

## Service: `BranchService`

This service encapsulates the business rules for managing branches.

### Core Flows

#### 1. Branch Creation
**Goal**: Establish a new location with a unique identifier.

**Process Flow**:
1.  **Validation**:
    *   Checks if the parent `Company` exists.
    *   {% hint style="warning" %}
    *   **Important / Warning:**
    *   **Uniqueness Check**: Verifies that `{companyId, branchCode}` is unique. You cannot have two branches with code `HQ-001` in the same company.
    *   {% endhint %}
2.  **Defaults**:
    *   Sets default `country` to "South Africa" if missing.
    *   Sets default `timezone` to "Africa/Johannesburg" if missing.
3.  **Persistence**: Saves the entity to the database.

#### 2. Soft Deletion
**Goal**: "Remove" a branch without breaking history.

**Why?**
If you delete a Branch that had 1000 shifts last year, all those shifts would have a dangling reference (pointing to nothing). This breaks reporting.

**Logic**:
*   Instead of `DELETE FROM`, we set `softDelete = true` and `deletedAt = NOW()`.
*   The branch is hidden from standard API calls but remains in the database for analytics.

### Validators

#### `CreateBranchRequest`
*   `name`: Required, max 255 chars.
*   `code`: Required, max 20 chars.
*   `timezone`: Max 50 chars.

### Entities

#### `Branch` (Database Table: `branches`)
*   **Keys**:
    *   `id` (UUID, PK)
    *   `company_id` (FK to Companies)
    *   `manager_id` (FK to Employees, optional)
*   **Enums**:
    *   `type`: `HEADQUARTERS`, `REGIONAL`, `SATELLITE`, `WAREHOUSE`, `RETAIL`, `VIRTUAL`.
*   **JSON Fields**:
    *   `opening_hours`: Stores complex weekly schedules (e.g., `{"mon": "09:00-17:00"}`).
    *   `services`: List of services offered at this location.

{% hint style="info" %}
**Note:**
We use **JSONB** for `opening_hours` to allow flexible schemas. Some branches might split shifts (09:00-12:00, 14:00-18:00), while others are 24/7. Validating this structure happens in the application layer.
{% endhint %}
