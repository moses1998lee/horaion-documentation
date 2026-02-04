# Rule Module Configuration

## Database Configuration

The module uses advanced Postgres features.

### JSONB Columns

We heavily utilize `jsonb` for flexibility.

*   `rules`: Meta-data table.
*   `rule_fields`: Schema table.
    *   `source_config`: JSONB. For `SELECT` fields, defines where options come from (e.g., "API:/api/v1/skills").
    *   `ui_config`: JSONB. Rendering hints.
*   `rule_answers`: Data table.
    *   `field_values`: JSONB. The actual saved configuration.

### Indexes

| Index Name | Table | Method | Purpose |
| :--- | :--- | :--- | :--- |
| `idx_rule_answers_rule_id` | `rule_answers` | B-tree | Fast fetch of config for a specific rule. |
| `gin_field_values` | `rule_answers` | **GIN** | (Recommended) Allows querying *inside* the JSON blob. e.g., "Find all stores with max_hours > 40". |

{% hint style="success" %}
**Tip / Success:**
**GIN Indexing**: If you plan to run analytics on rule configurations (e.g. "Which stores are strictest?"), you must add a GIN index to the `field_values` column manually, as JPA does not support it natively.
{% endhint %}
