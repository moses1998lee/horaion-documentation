# Demand Forecast Configuration

## Database Configuration

This module utilizes the PostgreSQL **JSONB** data type extensively.
Ensure your database dialect is configured for PostgreSQL.

## Application Properties

No specific external API keys are required.

{% hint style="success" %}
**Tip / Success:**
**Performance Tuning**: If the `demand_forecasts` table grows too large (millions of rows), consider partitioning the table by `forecast_date` (e.g., yearly partitions), as queries are almost always date-range based.
{% endhint %}
