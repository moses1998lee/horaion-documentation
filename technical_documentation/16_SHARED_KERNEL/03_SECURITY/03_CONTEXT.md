# 03 - User Context

> **Accessing the Current User**

---

## Overview
In a multi-tenant system, it is critical to know *who* is making the request and *which company* they belong to, to prevent data leaks.

The `SecurityContextService` abstracts `SecurityContextHolder` and provides type-safe access to the current user's profile.

**Service**: `com.horaion.app.shared.security.services.SecurityContextService`

---

## Key Capabilities

### 1. Get Current User
Fetches the `Employee` entity linked to the current JWT's `sub` (Subject ID).
```java
Employee me = securityContextService.getRequiredCurrentEmployee();
```

### 2. Data Boundaries
Enforces that a user can only access resources within their own Company/Branch.

```java
// Throws UnauthorizedResourceAccessException if mismatch
securityContextService.validateCompanyAccess(targetCompanyId, "Shift");
```

### 3. Hierarchy Checks
Contains logic for "Managerial Access" (e.g., Department Leads accessing their reports).
*   `isDepartmentLead()`
*   `isEmployeeInManagedDepartment(targetEmployeeId)`

---

## Usage Pattern

Inject `SecurityContextService` into your Domain Services or Command Handlers to perform ownership validation **before** executing business logic.

```java
public void updateShift(UpdateShiftCommand command) {
    Shift shift = shiftRepository.findById(command.id());
    
    // Security Check: Ensure I am editing a shift in MY company
    securityContextService.validateCompanyAccess(shift.getCompanyId(), "Shift");
    
    // Proceed...
}
```
