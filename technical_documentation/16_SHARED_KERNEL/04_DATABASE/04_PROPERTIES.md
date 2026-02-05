# 04 - Database Properties

> **Configuration as Code**

We use Spring's `@ConfigurationProperties` to map `application.yaml` values into strongly-typed Java objects. This provides validation and IDE auto-completion.

---

## 1. DatabaseProperty
**File**: ``DatabaseProperty.java``

**Prefix**: `spring.datasource`

| Field | Description |
| :--- | :--- |
| `url` | Standard JDBC URL. |
| `username` | Database user. |
| `password` | Database password. |
| `hikari` | Nested properties for fine-tuning the pool. |

---

## 2. HikariProperty
**File**: ``HikariProperty.java``

**Prefix**: `spring.datasource.hikari`

These settings allow you to override the [Default Constants](02_CONSTANTS.md).

### YAML Example
```yaml
spring:
  datasource:
    url: jdbc:postgresql://db-prod:5432/horaion
    username: app_user
    password: secure_password
    hikari:
      maximum-pool-size: 50   # Override default (10) for high-traffic
      connection-timeout: 60000 # Wait 60s for connection
```

{% hint style="warning" %}
**Warning:** Be careful when increasing `maximum-pool-size`. PostgreSQL has a limit on concurrent connections (often 100). If you have 5 Application Instances each with a pool size of 50, you will exhaust the database limit (5 * 50 = 250 connections).
{% endhint %}

{% hint style="info" %}
**Note:** For most Horaion deployments, the default `maximum-pool-size` of 10 is sufficient. Increase it only if you observe connection acquisition timeouts in your logs.
{% endhint %}
