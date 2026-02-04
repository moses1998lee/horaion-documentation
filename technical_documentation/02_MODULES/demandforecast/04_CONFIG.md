# Demand Forecast Configuration

## Database Configuration

This module utilizes the PostgreSQL **JSONB** data type extensively.
Ensure your database definition includes the `jsonb` column type for `timeblocks`.

### Indexing Strategy
To ensure the Schedule Engine can retrieve data instantly, we index the following:

| Index Name | Columns | Purpose |
| :--- | :--- | :--- |
| `uq_demand_forecast_department_date` | `department_id`, `forecast_date` | **Uniqueness**. Prevents duplicate plans. |
| `idx_demand_forecasts_forecast_date` | `forecast_date` | **Range Queries**. "Get me the forecast for next week". |

{% hint style="success" %}
**Tip / Success:**
**Partitioning**: If the `demand_forecasts` table grows too large (millions of rows), it is the prime candidate for **Table Partitioning** by Year.
*   `demand_forecasts_2024`
*   `demand_forecasts_2025`
Queries almost always allow filtering by date, making this physically efficient.
{% endhint %}

## Application Properties

No specific external API keys are required.
