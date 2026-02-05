# 03 - Core Enums

> **Domain-Agnostic Enumerations**

Enums in the Shared Kernel represent fixed sets of values that act as "Configuration as Code". They are used to ensure type safety across Service boundaries.

---

## 1. Operation Status
**File**: [`OperationStatus.java`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/core/enums/OperationStatus.java)

This enum is widely used in **Batch Processing** and **Async Jobs** to indicate the final result of a task.

### Values
| Enum | Usage Scenario |
| :--- | :--- |
| `SUCCESS` | Everything went perfectly. |
| `FAILURE` | A fatal error occurred, and the operation aborted. |
| `PARTIAL_SUCCESS` | Used in bulk uploads (e.g., "Imported 90/100 employees"). |

{% hint style="info" %}
**Note:** When using `PARTIAL_SUCCESS`, your response object should typically include a list of the specific items that failed so the user can correct them.
{% endhint %}

### Why not just use Boolean?
A `boolean success` flag is binary (True/False). It cannot represent the nuance of a "Partial" success where some data was saved, and some was rejected. The Enum allows for this granularity.
