# 05 - Core Helpers

> **Utility Classes for cleaner Service Logic**

Helper classes are designed to encapsulate repetitive, low-level logic. If you find yourself writing the same 3 lines of code in multiple services, it belongs here.

---

## 1. Repository Helper
**File**: ``RepositoryHelper.java``

This is the most frequent helper you will use. It solves the common boilerplate of "Find by ID, and if empty, throw 404".

### The "Find or Throw" Pattern

**⛔ Bad Code (Repetitive)**
```java
// You have to write this block 100 times across the app
Employee emp = employeeRepository.findById(id)
    .orElseThrow(() -> new ResourceNotFoundException("Employee not found with ID: " + id));
```

**✅ Good Code (Clean)**
```java
// One liner, type-safe, and consistent error message
Employee emp = RepositoryHelper.findByIdOrThrow(employeeRepository, id, Employee.class);
```

{% hint style="info" %}
**Note:** This helper automatically generates a standardized error message based on the Class name (e.g., "Employee not found...").
{% endhint %}

---

## 2. Validation Helper
**File**: ``ValidationHelper.java``

Centralizes common business validation checks.

### Capabilities
*   **Null Checks**: `ensureNotNull(obj, "Field Name")` - Throws `BadRequestException` if null.
*   **String Checks**: `ensureNotBlank(str, "Field Name")`.
*   **Date Logic**: `ensureEndDateAfterStartDate(start, end)`.

{% hint style="warning" %}
**Warning:** Do not confuse this with Jakarta Bean Validation (`@NotNull`).
*   Use `@NotNull` on DTOs for **Input Validation** (Controller layer).
*   Use `ValidationHelper` for **Business Validation** (Service layer) where complex logic depends on calculated values.
{% endhint %}
