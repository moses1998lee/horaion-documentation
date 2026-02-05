# 02 - The Interface Strategy

> **The Contract for Dynamic Options**

**File**: ``FieldOptionProvider.java``

Every provider must implement this interface to be discoverable by the system.

---

## The Method Signature

```java
Map<String, Object> getOptions(
    UUID companyId,     // Context: Which Company?
    UUID branchId,      // Context: Which Branch?
    UUID departmentId,  // Context: Which Department?
    Map<String, Object> sourceConfig // Config: Extra flags (e.g. "includeAll")
);
```

### Parameters in Detail
*   **Context IDs**: These allow the provider to filter data. For example, `EmployeeOptionProvider` will only return employees belonging to `branchId`.
*   **Source Config**: A flexible map passed from the caller. This allows for ad-hoc configuration, such as:
    *   `includeAllOption: true` -> Adds "All" to the list.
    *   `includeEmployeeCode: false` -> Hides the employee code in the label.

### Return Type
The method returns a standardized Map structure that the Frontend understands:

```json
{
  "type": "select",
  "options": [
    { "label": "John Doe", "value": "uuid-1" },
    { "label": "Jane Doe", "value": "uuid-2" }
  ]
}
```

---

## Implementing a New Provider
1.  Create a class implementing `FieldOptionProvider`.
2.  Annotate it with `@Component(name)`.
3.  Inject necessary Repositories.
4.  Implement logic to map Entities -> Options.

{% hint style="warning" %}
**Important:** Your provider name (in `@Component`) must be unique across the entire application. Use constants from `FieldOptionConstants` to avoid typo bugs.
{% endhint %}
