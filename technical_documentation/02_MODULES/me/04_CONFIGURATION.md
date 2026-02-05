# Me Module Configuration

## Database Configuration

This module is **Read-Only** on the Core tables (`employees`, `departments`) and **Read-Write** on the `user_notifications` table.

### Indexes & Performance

Since this module queries by `Cognito Sub` (Subject ID), this index is paramount.

| Index Name | Table | Column | Purpose |
| :--- | :--- | :--- | :--- |
| `idx_employee_cognito_sub` | `employees` | `cognito_sub` | **Critical**: Fast lookup of employee during login. |

## Application Properties

No specific properties, but it relies on:

*   `horaion.security.cognito.user-pool-id`: To resolve group definitions.
*   `horaion.notification.enabled`: If false, the notification endpoints return empty lists.

### Caching

{% hint style="info" %}
**Note:**
**Caching Strategy**: The `MeResponse` is a prime candidate for short-lived caching (e.g., 5 minutes) via `Redis`, as user permissions and profile data change rarely.
{% endhint %}
