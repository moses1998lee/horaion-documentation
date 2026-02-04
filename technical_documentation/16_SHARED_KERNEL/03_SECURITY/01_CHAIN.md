# 01 - Security Chain

> **Authentication and Authorization Configuration**

---

## Overview

The `SecurityConfiguration` class establishes the Spring Security Filter Chain. It is designed for a **stateless REST API** architecture relying on JWTs (Json Web Tokens).

**Configuration**: `com.horaion.app.shared.security.configurations.SecurityConfiguration`

---

## Filter Chain Setup

1.  **Session Management**: `SessionCreationPolicy.STATELESS` (No JSESSIONID cookies).
2.  **CSRF**: Disabled (Not needed for stateless APIs).
3.  **CORS**: Enabled (Configured via `corsConfigurationSource`).
4.  **Public Endpoints**:
    *   `/actuator/health`: Health checks.
    *   `/api/v1/auth/public/**`: Login/ForgotPassword endpoints.
    *   `/v3/api-docs/**`, `/swagger-ui/**`: API Documentation.
5.  **Secured Endpoints**: All other endpoints require `authenticated()`.

---

## OAuth2 Resource Server
We use Spring Security 6's built-in OAuth2 Resource Server to validate JWTs issued by **AWS Cognito**.

```java
http.oauth2ResourceServer(oauth2 -> oauth2
    .jwt(jwt -> jwt.jwtAuthenticationConverter(cognitoJwtAuthenticationConverter))
);
```

The validation (signature, expiration, issuer) is handled automatically by Spring Security based on the `spring.security.oauth2.resourceserver.jwt.issuer-uri` property.
