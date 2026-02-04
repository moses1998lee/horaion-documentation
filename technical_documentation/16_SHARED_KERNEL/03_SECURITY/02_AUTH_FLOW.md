# 02 - Auth Flow & Roles

> **From Token to Permissions**

---

## 1. Authentication (Who are you?)
When a request arrives with a `Authorization: Bearer <token>` header:
1.  Spring Security validates the token signature against AWS Cognito keys.
2.  The `CognitoJwtAuthenticationConverter` extracts the `cognito:groups` claim.
3.  Groups are mapped to Spring Security **Roles**.

**Converter**: `com.horaion.app.shared.security.jwt.CognitoJwtAuthenticationConverter`

### Role Mapping
| Cognito Group | Spring Security Role |
| :--- | :--- |
| `system-administrator` | `ROLE_SYSTEM_ADMINISTRATOR` |
| `system-owner` | `ROLE_SYSTEM_OWNER` |
| `user` | `ROLE_USER` |

---

## 2. Authorization (What can you do?)
We use a dual-layer authorization strategy.

### Layer A: Role-Based (High Level)
Used for coarse-grained access (e.g., "Only admins can see specific endpoints").
```java
@PreAuthorize("hasRole('SYSTEM_ADMINISTRATOR')")
public void deleteDatabase() { ... }
```

### Layer B: Attribute-Based (Fine Grained) w/ Permit.io
We use `Permit.io` for complex policy checks (e.g., "Can a Manager edit a Shift in their own Department?").

**Aspect**: `com.horaion.app.shared.security.aspects.PermitCheckAspect`

```java
@PermitCheck(action = "update", resource = "shift") // Intercepted by Aspect
public void updateShift(UUID shiftId) { ... }
```
The Aspect delegates to `PermitAuthorizationService` to check the policy against the PDP (Policy Decision Point).
