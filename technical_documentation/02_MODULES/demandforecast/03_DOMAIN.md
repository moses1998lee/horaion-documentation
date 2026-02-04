# Demand Forecast Domain Logic

## Service: `DemandForecastService`

This service handles the complex data transformation between simple JSON inputs and the optimized storage format.

### The 48-Slot System (Timeblocks)

The core domain concept is the **Timeblock Index**.
Instead of storing verbose timestamps like `"08:30:00"`, we store simple integers `0` to `47`.

| Index | Time Range | Formula |
| :--- | :--- | :--- |
| **0** | 00:00 - 00:30 | `(0 * 2) + 0` |
| **1** | 00:30 - 01:00 | `(0 * 2) + 1` |
| ... | ... | |
| **17** | 08:30 - 09:00 | `(8 * 2) + 1` |
| **34** | 17:00 - 17:30 | `(17 * 2) + 0` |
| **47** | 23:30 - 00:00 | `(23 * 2) + 1` |

### Sparse Storage Strategy

**Problem**: Storing 48 rows for every single role for every single day creates millions of empty rows.
**Solution**: We use a **Sparse JSONB** structure.

**Visualizing the Data**:
Imagine a spreadsheet where rows 0-15 (Midnight to 8 AM) are completely blank. We simply don't save them.

**Database JSON Example**:
```json
{
  "16": { "role_uuid_A": 2 },
  "17": { "role_uuid_A": 4, "role_uuid_B": 1 },
  "18": { "role_uuid_A": 4, "role_uuid_B": 1 }
}
```

{% hint style="success" %}
**Tip / Success:**
**Performance**: This structure keeps the database payload extremely small (often < 1KB per day), making date-range queries (e.g., "Get me the whole month") lightning fast.
{% endhint %}

### Frontend Integration Guide

For developers building the UI, you don't want to calculate `(index * 2) % 60` in your head. Use these standard TypeScript helpers.

#### TypeScript Utilities

```typescript
/**
 * Converts a 0-47 index to "HH:mm" string
 * Example: 17 -> "08:30"
 */
function indexToTime(index: number): string {
  const hours = Math.floor(index / 2);
  const minutes = (index % 2) * 30;
  return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`;
}

/**
 * Converts Hour/Minute to 0-47 index
 * Example: (8, 30) -> 17
 */
function timeToIndex(hours: number, minutes: number): number {
  return (hours * 2) + (minutes === 30 ? 1 : 0);
}
```

### Entities

#### `DemandForecast` (Database Table: `demand_forecasts`)

*   **Keys**:
    *   `department_id` + `forecast_date`: **Unique Constraint**.
    *   *Rule*: A department can only have **one** active forecast plan per physical day.
*   **Fields**:
    *   `timeblocks`: **JSONB**. Stores the sparse map shown above.
    *   `forecast_source`: String. E.g., `manual`, `import-csv`, `ai-v1`.
    *   `is_active`: Boolean.

{% hint style="warning" %}
**Important / Warning:**
**JSONB Indexing**: While PostgreSQL allows querying *inside* JSONB, it is slower than relational columns. We optimized this module to strictly fetch by `department_id` and `date`. We do **not** support queries like "Find all days where we need Role X at 8am".
{% endhint %}
