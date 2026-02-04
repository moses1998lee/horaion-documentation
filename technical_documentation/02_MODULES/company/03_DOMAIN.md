# Company Module Domain Logic

## Service: `CompanyService`

The guardian of tenant isolation and business identity.

### Core Flows

#### 1. Creation & Duplicate Prevention
**Goal**: Ensure every company on the platform is a unique legal entity.

**Logic**:
1.  **Check**: `companyRepository.existsByRegistrationNumber(...)`.
    *   {% hint style="warning" %}
    *   **Important / Warning:**
    *   **Strict Uniqueness**: The Registration Number is the primary "business key". Creating a duplicate throws a `DuplicateRegistrationNumberException` immediately.
    *   {% endhint %}
2.  **Persist**: If unique, a new `Company` record is created.
3.  **Metadata**: Timestamps (`createdAt`, `updatedAt`) are auto-managed.

#### 2. Multi-Tenancy Enforcement (Search & List)
**Goal**: Users should only see their own company (unless they are System Admins).

**Logic**:
*   The service checks `securityContextService.isSystemAdministrator()`.
    *   **If Admin**: Call `findAllCandidates()` (Global view).
    *   **If Regular User**:
        1.  Extract `currentCompanyId` from the token.
        2.  Force-filter the query to return **only** that specific company ID.
        3.  Even if they request "page 1 of all companies", they effectively get a page containing just their own single record.

### Validators

#### `CreateCompanyRequest`
*   `name`: Required (Max 255).
*   `registrationNumber`: Required (Max 50).

### Entities

#### `Company` (Database Table: `companies`)

*   **Keys**:
    *   `id` (UUID, PK)
*   **Fields**:
    *   `name`: Display name.
    *   `registrationNumber`: **Unique Index**. The legal identifier.
    *   `hasCompletedOnboarding`: Boolean flag for UI state.

{% hint style="info" %}
**Note:**
The `companies` table is the root of the entire data hierarchy. Most other tables in the database will have a `company_id` column that Foreign Keys back to this table.
{% endhint %}
