# Company Module Configuration

## Database Configuration

The Company module defines the root `companies` table in the PostgreSQL database.

### Critical Constraints
To ensure data integrity at the root level, specific constraints are applied:

| Constraint Type | Column | Purpose |
| :--- | :--- | :--- |
| **Primary Key** | `id` | UUID for global uniqueness. |
| **Unique Index** | `registration_number` | Prevents duplicate legal entities. |
| **Index** | `name` | Optimizes the "Find Company" search bar for Admins. |

{% hint style="success" %}
**Tip / Success:**
**Why UUIDs?**
We use UUIDs (GUIDs) instead of Auto-Increment Integers (`1, 2, 3`) to prevent "ID Enumeration Attacks". You cannot guess that "Company 505" exists just because you are "Company 504".
{% endhint %}

## Application Properties

This module operates on core application logic and does not require specific `application.yml` configurations.

### Inter-Module Dependencies
While it has no config of its own, it **configures** the rest of the system:
*   **Auth Module**: Uses `Company` existence to validate User Registration.
*   **Branch Module**: Uses `company_id` to enforce Multi-tenancy scopes.
