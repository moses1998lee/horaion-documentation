# Branch Module Configuration

## Database Configuration

The **Branch Module** relies on the core PostgreSQL database. It introduces the `branches` table.

### Indexes & Performance
To ensure fast lookups—especially when filtering by location or hierarchy—we explicitly index the following columns:

| Index Name | Column | Purpose |
| :--- | :--- | :--- |
| `idx_branches_company_id` | `company_id` | **Critical**: Every query filters by Company ID first (Multi-tenancy). |
| `idx_branches_branch_code` | `branch_code` | User search (e.g., look up "HQ-01"). |
| `idx_branches_city` | `city` | Reporting and filtering (e.g., "Show all Cape Town stores"). |
| `idx_branches_country` | `country` | Regional reporting. |

{% hint style="danger" %}
**Critical for Multi-Tenancy**: The `idx_branches_company_id` index is the most important for performance. Every SQL query in the Branch module (and most others) includes a `WHERE company_id = ?` clause to ensure data isolation. Without this index, system performance would degrade exponentially as the number of clients grows.
{% endhint %}

{% hint style="success" %}
**Tip / Success:**
**Multi-Tenant Performance**: The composite uniqueness constraint `(company_id, branch_code)` also acts as a covering index for those two fields, speeding up existence checks during creation.
{% endhint %}

## Application Properties

This module operates on standard defaults and does not require unique `application.yml` entries.
However, it heavily influences the configuration of *other* modules:

*   **Scheduling**: Relies on the `timezone` field stored in this module.
*   **Geofencing**: Relies on `latitude`/`longitude` stored here.
