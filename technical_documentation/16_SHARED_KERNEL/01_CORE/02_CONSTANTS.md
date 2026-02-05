# 02 - Core Constants

> **The Single Source of Truth for Magic Values**

Hardcoding strings or numbers (e.g., `"ADMIN"`, `45`) throughout the codebase is a major source of bugs. If the value changes, you have to find-and-replace it everywhere. Instead, we use **Constant Classes**.

---

## 1. Role Constants
**File**: ``RoleConstant.java``

This file defines the exact string representations of the User Roles used by Spring Security.

### Defined Roles
| Constant | Value | Description |
| :--- | :--- | :--- |
| `ROLE_ADMIN` | `"ADMIN"` | Full system access. |
| `ROLE_MANAGER` | `"MANAGER"` | Can manage employees and schedules within their branch. |
| `ROLE_EMPLOYEE` | `"EMPLOYEE"` | Read-only access to their own data. |

{% hint style="warning" %}
**Important:** Spring Security is strictly case-sensitive. Always use these constants in your `@PreAuthorize` annotations to avoid typos like `"Admin"` vs `"ADMIN"`.
{% endhint %}

### Example Usage
```java
// ✅ GOOD: Safe refactoring
@PreAuthorize("hasRole(" + RoleConstant.ROLE_ADMIN + ")")
public void deleteSystem() { ... }

// ❌ BAD: Prone to typos
@PreAuthorize("hasRole('ADMIN')")
public void deleteSystem() { ... }
```

---

## 2. Service Executor Constants
**File**: ``ServiceOperationExecutorConstant.java``

These constants are used by the `ServiceOperationExecutor` to ensure logs are formatted identically across thousands of service calls.

*   `LOG_ENTER_PATTERN`: `"Enter: {}"`
*   `LOG_EXIT_PATTERN`: `"Exit: {} ({} ms)"`

{% hint style="success" %}
**Tip:** By standardizing log messages here, we can easily build Dashboards in Kibana/Splunk that parse these exact patterns to visualize performance metrics.
{% endhint %}
