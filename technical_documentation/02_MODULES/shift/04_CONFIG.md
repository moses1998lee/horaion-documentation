# Configuration & Security

## Application Configuration

The Shift Module does not require specific entries in `application.yml` as it leverages the shared infrastructure (PostgreSQL, Security, Validator).

However, it relies on the Global Exception Handler for standardized error responses.

## Permissions (RBAC)

Access is controlled via the `Permit.io` integration. The following actions must be assigned to user roles for them to interact with this module.

| Resource | Action | Description | Typical Role |
| :--- | :--- | :--- | :--- |
| `shift` | `read` | View shift templates | Manager, Employee, Admin |
| `shift` | `create` | Define new shifts | Manager, Admin |
| `shift` | `update` | Modify existing shifts | Manager, Admin |
| `shift` | `delete` | Soft-delete a shift | Manager, Admin |

{% hint style="info" %}
**Note:**
Permissions are implicitly scoped to the **Department**. A Manager with `read:shift` can only read shifts within their own Department(s).
{% endhint %}

## Database Indexes

To ensure performance during schedule generation (which queries shifts heavily), the following indexes are critical:

```sql
-- Filter by Department (Most common query)
CREATE INDEX idx_shifts_department_id ON shifts(department_id);

-- Filter by Type (Reporting)
CREATE INDEX idx_shifts_shift_type ON shifts(shift_type);

-- Soft Delete awareness (Optional but recommended for large datasets)
-- Note: The current `@Where(clause = "soft_delete = false")` handles this via Hibernate.
```
