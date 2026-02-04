# Demand Forecast Domain Logic

## Service: `DemandForecastService`

This service handles the complex data transformation between simple JSON inputs and the optimized storage format.

### The 48-Slot System (Timeblocks)

The core domain concept is the **Timeblock Index**.
Instead of storing timestamps like `"08:30:00"`, we store integers `0` to `47`.

| Index | Time Range |
| :--- | :--- |
| **0** | 00:00 - 00:30 |
| **1** | 00:30 - 01:00 |
| ... | ... |
| **17** | 08:30 - 09:00 |
| ... | ... |
| **47** | 23:30 - 00:00 |

### Sparse Storage Strategy

**Problem**: Storing 48 rows for every single role for every single day is database bloat.
**Solution**: We use a **Sparse JSONB** structure.

If a shop is closed from 00:00 to 08:00, we simply **do not save** indices 0-15. We only save the indices where `demand > 0`.

**Database JSON Example**:
```json
{
  "16": { "role_uuid_A": 2 },
  "17": { "role_uuid_A": 4, "role_uuid_B": 1 }
}
```

{% hint style="info" %}
**Note:**
This structure makes the database payload extremely small, even for years of data.
{% endhint %}

### TypeScript Helper (Frontend Guide)

For frontend developers integrating with this module, here is a handy conversion cheat sheet:

```typescript
function indexToTime(index: number): string {
  const hours = Math.floor(index / 2);
  const minutes = (index % 2) * 30;
  return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`;
}
// indexToTime(17) -> "08:30"
```

### Entities

#### `DemandForecast` (Database Table: `demand_forecasts`)

*   **Keys**:
    *   `department_id` + `forecast_date`: **Unique Constraint**. A department can only have one active forecast plan per day.
*   **Fields**:
    *   `timeblocks`: **JSONB**. Stores the sparse map.
    *   `forecast_source`: String. E.g., "manual", "ai-prediction-v1".
    *   `is_active`: Boolean.

{% hint style="warning" %}
**Important / Warning:**
**JSONB Indexing**: The `timeblocks` column is stored as JSONB. While flexible, querying *inside* the JSON (e.g., "Find all days where we need Role X at 8am") is slower than standard relational columns. We primarily fetch the *whole* day's object.
{% endhint %}
