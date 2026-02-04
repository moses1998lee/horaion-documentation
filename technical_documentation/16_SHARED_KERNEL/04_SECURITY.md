# 04 - Shared Security

> **Authentication, Authorization and User Context**

---

## 4.1 Security Architecture
**Package**: `com.horaion.app.shared.security`

Security is implemented using **Spring Security 6** and **OAuth2 Resource Server** standards. The application is stateless, relying on JWTs (JSON Web Tokens) issued by AWS Cognito.

### Security Chain
The HTTP request passes through a chain of filters before reaching the controller:

1.  **CorsFilter**: Handles Cross-Origin requests.
2.  **JwtAuthenticationFilter**: Validates the Bearer token against Cognito's keys.
3.  **PermitFilter** (Optional): Extensible filter for fine-grained policy checks (e.g., Permit.io integration).
4.  **UserContextFilter**: Extracts the user ID from the valid JWT and populates the `UserContext`.

---

## 4.2 Security Configuration
**Package**: `com.horaion.app.shared.security.configurations`

### SecurityConfiguration
This class configures the `SecurityFilterChain`. It sets up:
1.  **CSRF**: Disabled (Stateless API).
2.  **Session Management**: STATELESS.
3.  **Authorization**: Secure by default, with specific exclusions (e.g., `/auth/**`, `/actuator/**`).
4.  **OAuth2 Resource Server**: Configured to decdoe JWTs using Cognito keys.

### Custom Converters
**Package**: `com.horaion.app.shared.security.jwt`

*   `CognitoJwtAuthenticationConverter`: Maps AWS Cognito groups/claims to Spring Security authorities.
*   `CognitoJwtValidator`: Validates custom claims (e.g., audience, issuer).

---

## 4.3 User Context
**Package**: `com.horaion.app.shared.security.context`

### UserContext Object
A thread-local object available throughout the request lifecycle.

```java
public record UserContext(
    UUID userId,
    UUID tenantId,
    List<String> roles,
    String email
) {}
```

### Usage
Developers can inject the context or access it statically to get the current user:

```java
@Service
public class ShiftService {
    public void createShift(Shift shift) {
        UserContext currentUser = SecurityContextHolder.getUserContext();
        log.info("Shift created by user: {}", currentUser.email());
        shift.setCreatedBy(currentUser.userId());
    }
}
```
