# Schedule Module Configuration

## Database Configuration

The **Schedule Module** manages the `schedules` and `schedule_shifts` tables.

### Indexes & Performance

This table is high-traffic.

| Index Name | Column | Purpose |
| :--- | :--- | :--- |
| `idx_schedules_department_id` | `department_id` | View schedules for a specific store. |
| `idx_schedules_dates` | `start_date`, `end_date` | Date range filtering. |
| `idx_schedules_job_status` | `job_status` | Finding "Stuck" jobs (e.g., Processing > 1 hour). |

## Application Properties

Specific configuration using the `schedule` prefix.

| Property | Default | Description |
| :--- | :--- | :--- |
| `schedule.enabled` | `true` | Master switch for the module. |
| `schedule.engine-url` | (Required) | URL of the Python Optimization Engine. |
| `schedule.callback-base-url` | (Required) | Public URL of *this* API (e.g., `https://api.horaion.com`). Used to construct the callback hook. |

### External Engine Integration

The engine is a separate microservice.
*   **Protocol**: HTTP / REST.
*   **Auth**: Currently internal-network trust or Shared Secret (check `EngineClient` implementation).

{% hint style="danger" %}
**Critical:**
**Callback URL**: In non-production environments (e.g., localhost), the `callback-base-url` must be reachable by the Engine. You may need tools like **ngrok** to expose your local API to an external engine.
{% endhint %}
