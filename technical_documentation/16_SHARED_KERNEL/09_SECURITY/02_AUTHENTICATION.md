# 02 - Authentication & Roles

> **Identity Management via Cognito**

**File**: ``CognitoJwtAuthenticationConverter.java``

We treat AWS Cognito as the Identity Provider (IdP). The application never sees a password; it only sees a signed JWT.

---

## The Conversion Process
When a request arrives, we don't just "trust" the token. We convert it into a Spring Security `Authentication` object.

### Role Mapping
Cognito Groups are mapped to Spring Security Roles to adhere to Java standards (`ROLE_` prefix).

| Cognito Group | Spring Security Role | Constant |
| :--- | :--- | :--- |
| `system-administrator` | `ROLE_SYSTEM_ADMINISTRATOR` | `RoleConstant.ROLE_SYSTEM_ADMINISTRATOR` |
| `system-owner` | `ROLE_SYSTEM_OWNER` | `RoleConstant.ROLE_SYSTEM_OWNER` |
| `user` | `ROLE_USER` | `RoleConstant.ROLE_USER` |

{% hint style="warning" %}
**Important:** If a user is in multiple groups, the application logic (via `AuthorizationHelper`) often determines a **Primary Role** based on precedence (Admin > Owner > User).
{% endhint %}

---

## Security Filter Chain
**File**: ``SecurityConfiguration.java``

This class defines which endpoints are public and which are private.

```java
// Logic inside SecurityConfiguration
authorize
    .requestMatchers(SecurityConstant.getPublicEndpoints()).permitAll() // Public
    .requestMatchers(HttpMethod.OPTIONS, "/**").permitAll()             // CORS Pre-flight
    .anyRequest().authenticated();                                      // Everything else
```

### Public Endpoints
Defined in `SecurityConstant`, these typically include:
*   `/actuator/health` (Health Checks)
*   `/auth/login` (Initial Login)
*   `/public/**` (Marketing pages, if any)
