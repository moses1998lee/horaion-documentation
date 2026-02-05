# 02 - Application Properties

> **The Single Source of Configuration Truth**

---

## 2.1 File Structure

**File**: `src/main/resources/application.yaml`

The application uses a **single YAML file** strategy with dynamic environment variable injection. This follows the **12-Factor App** methodology, where configuration is stored in the environment.

{% hint style="warning" %}
**Important:** Do not hardcode secrets (passwords, keys) in this file. Always use the `${ENV_VAR}` syntax to inject them at runtime.
{% endhint %}

---

## 2.2 Core Sections

### Server & Management
Controls the embedded Tomcat server and Actuator endpoints.
```yaml
server:
  port: ${SERVER_PORT:8080} # Default: 8080
management:
  endpoints:
    web:
      exposure:
        include: health,info # Only expose safe endpoints
```

### CORS (Security)
Controls browser access policies.
```yaml
cors:
  enabled: ${CORS_ENABLED:true}
  allowed-origins: ${CORS_ALLOWED_ORIGINS:http://localhost:5173,http://localhost:3000}
```
*   **Dev**: Opens for `localhost:5173` (Vite) and `localhost:3000` (React).
*   **Prod**: Should be restricted to the specific domain.

### Database (Datasource)
Controls the connection to PostgreSQL.
```yaml
spring:
  datasource:
    url: jdbc:postgresql://${POSTGRES_DB_HOST}:${POSTGRES_DB_PORT}/${POSTGRES_DB_NAME}
    password: ${POSTGRES_DB_PASS}
    hikari:
      maximum-pool-size: ${DB_POOL_SIZE:10} # Don't go higher than DB CPU/2
```

### AWS Integration
Centralized configuration for all AWS services.
```yaml
spring.cloud.aws:
  region:
    static: ${AWS_REGION:ap-southeast-1}
  credentials:
    access-key: ${AWS_ACCESS_KEY_ID:}
    secret-key: ${AWS_SECRET_ACCESS_KEY:}
```
*   **Note**: The colons `id:` with empty values typically mean the SDK will fall back to the Default Credential Provider Chain (Instance Profile) if these are not set.

---

## 2.3 Custom Properties (`app.*`)

We map these custom keys to Java `@ConfigurationProperties` classes (e.g., `DatabaseProperties`, `CognitoProperty`).

| Property Key | Java Class | Purpose |
| :--- | :--- | :--- |
| `app.database.*` | `DatabaseProperties` | Custom Hikari tuning flags. |
| `app.cognito.*` | `CognitoProperty` | User Pool ID and Client ID. |
| `app.security.*` | `SecurityProperties` | IP Whitelists for callbacks. |
| `app.logging.*` | `LoggingProperties` | Slow SQL detection thresholds. |

---

## 2.4 External Modules

### Permit.io (Authorization)
```yaml
permit:
  pdp-address: ${PERMIT_PDP_ADDRESS:http://localhost:7766}
  api-key: ${PERMIT_API_KEY:}
```

### Schedule Engine
```yaml
schedule:
  engine:
    base-url: ${SCHEDULE_ENGINE_BASE_URL}
    read-timeout-ms: 300000 # 5 Minutes
```

{% hint style="danger" %}
**Critical:** The `read-timeout-ms` for the Engine is extremely high because the optimization process is computationally intensive. Lowering this will cause "Timeout" errors even if the engine successfully generates a schedule later.
{% endhint %}
