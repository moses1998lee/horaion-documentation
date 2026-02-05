# 03 - Standard Implementations

> **Available Providers Registry**

**File**: ``FieldOptionConstants.java``

We maintain a registry of constant names for all standard providers.

---

## Registry

| Provider Name | Constant Key | Details |
| :--- | :--- | :--- |
| **Employee** | `PROVIDER_EMPLOYEE` | Returns Active Employees. Filters by Branch/Dept. Supports `includeAllOption`. |
| **Role** | `PROVIDER_ROLE` | Returns System Roles (Admin, Manager, etc.). |
| **Shift** | `PROVIDER_SHIFT` | Returns Shift Templates available for the branch. |
| **Role List** | `PROVIDER_ROLE_LIST` | Returns a simplified list of roles for basic filtering. |

### Example Usage (Employee)
**Class**: ``EmployeeOptionProvider``

This provider demonstrates best practices:
1.  **Defaults**: If no `branchId` is provided, it attempts to return *all* employees (be careful with this!).
2.  **Formatting**: It formats the label as `FirstName LastName (Code)`.
3.  **Configurable**: You can pass `includeEmployeeCode: false` to hide the code.

```java
// Logic inside EmployeeOptionProvider
if (includeAll) {
    options.add(Map.of("value", "all", "label", "All Employees"));
}
```

### JSON Response Structure
The Frontend receives this exact structure:

```json
{
  "type": "select",
  "option": [
    {
      "value": "all",
      "label": "All Employees"
    },
    {
      "value": "a1b2c3d4-...",
      "label": "John Doe (EMP001)"
    }
  ]
}
```

{% hint style="info" %}
**Note:** When creating new providers, always add the constant to `FieldOptionConstants.java` first. Do not use magic strings in your `@Component` annotation.
{% endhint %}
