# Shift Module Configuration

## Database Configuration

### Tables

1.  **`shifts`**: The main definition.
    *   `days_applied_to`: **Integer Array (`integer[]`)**.
        *   Stores the days of week (1=Mon, 7=Sun) this shift is valid for.
        *   *Advantage*: Array columns in Postgres allow extremely fast filtering (e.g., "Find all shifts valid on Mondays" -> `WHERE 1 = ANY(days_applied_to)`).

2.  **`shift_roles`**: The staffing requirements.
    *   Maps `shift_id` <-> `employee_role_id`.

### Indexes

| Index Name | Table | Column | Purpose |
| :--- | :--- | :--- | :--- |
| `idx_shifts_department_id` | `shifts` | `department_id` | Partitioning shifts by department. |
| `uq_shift_label_per_department` | `shifts` | `department_id` + `label` | Enforcing unique names. |

## Application Properties

No specific properties required.

{% hint style="danger" %}
**Critical:**
**Hard Deletes**: Be careful with "Hard Deletes" on Shifts. If you hard-delete a shift template, you may break historical reporting for schedules that referenced it. Always use **Soft Delete** (`is_active = false`).
{% endhint %}
