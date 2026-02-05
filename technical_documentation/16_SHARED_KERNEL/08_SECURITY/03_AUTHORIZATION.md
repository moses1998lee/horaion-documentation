# 03 - Authorization & Policy

> **Who can do what?**

We split authorization into two distinct layers: **Data Ownership** (Internal) and **Action Permissibility** (External Policy).

---

## 1. Data Ownership (SecurityContextService)
**File**: ``SecurityContextService.java``

This service protects **Data Privacy**. It answers questions like:
*   *"Is this user allowed to see the data of Company X?"*
*   *"Is this Manager allowed to see the Leave Availability of Employee Y?"*

### The Hierarchy Logic
The service enforces a strict hierarchy:
1.  **System Admin**: Context-free access (can see everything).
2.  **Super Privileged**: Can manage other Privileged users.
3.  **Privileged**: Can manage Regular users (but not other Privileged users).
4.  **User**: Can only see their own data.

```java
// Example: Validating Access
public void validateLeaveAvailabilityAccess(UUID employeeId) {
    if (isSystemAdministrator()) return; // Pass
    if (isCurrentEmployee(employeeId)) return; // Pass (Own Data)
    
    // Complex check: Is the requester a Manager of the target employee?
    if (isDepartmentLeadOf(employeeId)) { ... }
}
```

---

## 2. Policy Enforcement (PermitAuthorizationService)
**File**: ``PermitAuthorizationService.java``

This service protects **Actions**. It delegates decisions to **Permit.io**, allowing us to change rules without redeploying code.
*   *"Can a `Manager` `create` a `Shift`?"*
*   *"Can a `User` `approve` a `TimeOffRequest`?"*

### How it works
1.  **Check**: Code calls `permitAuthorizationService.checkOrThrow(...)`.
2.  **Build**: The service builds a lightweight `User` object (Attributes: `role`, `email`, `groups`).
3.  **Delegate**: It asks the Permit PDP (Policy Decision Point).
4.  **Enforce**: If denied, it throws `PermitAuthorizationException` (403 Forbidden).

{% hint style="success" %}
**Tip:** Use `SecurityContextService` for **Tenant/Ownership** checks (hard business rules). Use `PermitAuthorizationService` for **Feature/Action** checks (flexible policy).
{% endhint %}
