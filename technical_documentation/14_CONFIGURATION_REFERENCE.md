# Genesis API - Configuration Deep Dive

> **Genesis Workforce Management Platform - Configuration Reference**

Complete reference for all configuration classes and their purposes.

---

## Configuration Classes Overview

Genesis has **6 configuration classes** that set up critical infrastructure:

| Configuration | Purpose | Key Features |
|---------------|---------|--------------|
| AsyncConfiguration | Async processing | 2 thread pools for different workloads |
| CognitoConfiguration | AWS Cognito client | JWT validation, user pool integration |
| DatabaseConfiguration | Database connection | HikariCP with PostgreSQL optimizations |
| FeignConfiguration | HTTP client | 45-min timeouts, no retries |
| SecurityConfiguration | Spring Security | JWT auth, CORS, endpoint protection |
| WebServerConfiguration | Tomcat server | Unlimited timeouts for long operations |

---

## 1. AsyncConfiguration

**File**: `com.resetrix.genesis.configurations.AsyncConfiguration`

### Purpose
Enables asynchronous method execution with dedicated thread pools.

### Thread Pools

#### Schedule Task Executor
```java
@Bean(name = "scheduleTaskExecutor")
public Executor scheduleTaskExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(5);
    executor.setMaxPoolSize(20);
    executor.setQueueCapacity(100);
    executor.setThreadNamePrefix("Schedule-Async-");
    executor.setKeepAliveSeconds(60);
    return executor;
}
```

**Characteristics**:
- **Core threads**: 5 (always active)
- **Max threads**: 20 (scales under load)
- **Queue**: 100 tasks
- **Thread naming**: `Schedule-Async-{N}`
- **Keep-alive**: 60 seconds for idle threads
- **Rejection policy**: Throws `RuntimeException`

**Usage**:
```java
@Async("scheduleTaskExecutor")
public void generateSchedule(Long scheduleId) {
    // Long-running schedule generation
}
```

#### General Task Executor
```java
@Bean(name = "generalTaskExecutor")
public Executor generalTaskExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(3);
    executor.setMaxPoolSize(10);
    executor.setQueueCapacity(50);
    executor.setThreadNamePrefix("General-Async-");
    executor.setKeepAliveSeconds(60);
    return executor;
}
```

**Characteristics**:
- **Core threads**: 3
- **Max threads**: 10
- **Queue**: 50 tasks
- **Thread naming**: `General-Async-{N}`

**Usage**:
```java
@Async  // Uses default executor (generalTaskExecutor)
public void processAsyncCognitoCreation(String jobId) {
    // Cognito account creation
}
```

### When to Use Which Executor

### When to Use Which Executor

| Scenario | Recommended Executor | Why? |
|---|---|---|
| **Generating a Schedule** | `scheduleTaskExecutor` | 🕒 **Very Long Duration (15-45m)**. These tasks need a separate lane so they don't clog up the system for quick tasks. |
| **Sending Emails** | `generalTaskExecutor` | ⚡ **Fast (1-5s)**. High volume, low latency. |
| **Creating Cognito User** | `generalTaskExecutor` | ⚡ **Fast (1-2s)**. External API call, but relatively quick. |
| **System Cleanup** | `generalTaskExecutor` | 🧹 **Background Maintenance**. Low priority. |

> **Analogy**: Think of `scheduleTaskExecutor` as the "Heavy Cargo Lane" on the highway, and `generalTaskExecutor` as the "Fast Lane". If you put a slow truck in the fast lane, traffic jams ensued.

---

## 2. FeignConfiguration

**File**: `com.resetrix.genesis.configurations.FeignConfiguration`

### Purpose
Configures Feign HTTP clients for external service communication.

### Timeout Configuration

```java
@Bean
public Request.Options feignRequestOptions() {
    int timeoutMs = 2700000;  // 45 minutes
    
    return new Request.Options(
        timeoutMs,  // connectTimeout
        timeoutMs,  // readTimeout
        true        // followRedirects
    );
}
```

**Breakdown**:
- **Connect timeout**: 45 minutes (2,700,000 ms)
- **Read timeout**: 45 minutes (2,700,000 ms)
- **Follow redirects**: Enabled

**Why 45 minutes? (The "Long Polling" Strategy)**
*   **The Problem**: The Optimization Engine solves complex NP-hard problems (Shift assignments with constraints). This isn't instant. It usually takes 15-30 minutes.
*   **The Risk**: If we set a standard timeout (e.g., 30 seconds), the connection would drop while the engine is still thinking. We would lose the result.
*   **The Fix**: We set a massive `2,700,000ms` (45 min) timeout to keep the line open until the engine replies.

> **Warning**: Do not lower this value unless the Engine's performance characteristics change drastically. Premature timeouts will cause "Zombie Schedules" (engine finishes work, but API has already given up).

### Retry Configuration

```java
@Bean
public Retryer feignRetryer() {
    return Retryer.NEVER_RETRY;
}
```

**Why no retries?**
- Schedule generation is **not idempotent**
- Retrying could create duplicate schedules
- Engine maintains state between calls
- Better to fail fast and retry manually

### Error Decoder

```java
@Bean
public ErrorDecoder feignErrorDecoder() {
    return (methodKey, response) -> {
        LOGGER.warn("Feign client error for method {}: HTTP {} - {}",
                   methodKey, response.status(), response.reason());
        return new ErrorDecoder.Default().decode(methodKey, response);
    };
}
```

**Logs all Feign errors** with:
- Method name (e.g., `EngineClient#generateSchedule`)
- HTTP status code (e.g., `500`)
- Reason phrase (e.g., `Internal Server Error`)

---

## 3. DatabaseConfiguration

**File**: `com.resetrix.genesis.configurations.DatabaseConfiguration`

### Purpose
Configures HikariCP connection pool with database-specific optimizations.

### Default Settings

```java
// Pool sizing
maximumPoolSize: 10
minimumIdle: 10

// Timeouts
connectionTimeout: 30,000 ms  (30 seconds)
idleTimeout: 600,000 ms       (10 minutes)
maxLifetime: 1,800,000 ms     (30 minutes)

// Behavior
autoCommit: false
readOnly: false
```

### Symptoms of Misconfiguration

| Setting | Symptom if Wrong |
|---|---|
| **`maximumPoolSize` (Too Low)** | **Timeout Exceptions**: `SQLTransientConnectionException: Connection is not available, request timed out after 30000ms`. Requests pile up waiting for a free connection. |
| **`maximumPoolSize` (Too High)** | **Database CPU Spike**: The database spends more time context-switching between connections than executing queries. Performance degrades. |
| **`leakDetectionThreshold` (Enabled)** | **Memory Overhead**: If left on in Production, the detailed stack trace tracking consumes significant heap memory. |
| **`maxLifetime` > Database Timeout** | **Dead Connections**: If Hikari keeps a connection longer than the Database allows (e.g. AWS RDS defaults to 1 hour), the app tries to use a closed connection and fails. *Rule: maxLifetime must be < DB-side timeout.* |

### PostgreSQL Optimizations

**Automatically applied when JDBC URL starts with `jdbc:postgresql:`**:

```java
private void applyPostgreSQLOptimizations(HikariConfig config) {
    // Connection settings
    config.addDataSourceProperty("tcpKeepAlive", "true");
    config.addDataSourceProperty("ApplicationName", "Genesis-Application");
    config.addDataSourceProperty("assumeMinServerVersion", "12.0");
    
    // Performance
    config.addDataSourceProperty("reWriteBatchedInserts", "true");
    
    // Prepared statement caching
    config.addDataSourceProperty("prepareThreshold", "5");
    config.addDataSourceProperty("preparedStatementCacheQueries", "250");
    config.addDataSourceProperty("preparedStatementCacheSizeMiB", "5");
    
    // Disable time stats (performance)
    config.addDataSourceProperty("maintainTimeStats", "false");
}
```

**Benefits**:

| Property | Benefit |
|----------|---------|
| `tcpKeepAlive` | Prevents idle connection drops |
| `ApplicationName` | Identifies app in PostgreSQL logs |
| `assumeMinServerVersion` | Enables PostgreSQL 12+ features |
| `reWriteBatchedInserts` | Optimizes batch INSERT performance |
| `prepareThreshold=5` | Caches statements used 5+ times |
| `preparedStatementCacheQueries=250` | Caches up to 250 statements |
| `preparedStatementCacheSizeMiB=5` | 5MB cache size |
| `maintainTimeStats=false` | Reduces overhead |

### H2 Optimizations

**Applied for in-memory/test databases**:

```java
private void applyH2Optimizations(HikariConfig config) {
    config.addDataSourceProperty("DB_CLOSE_DELAY", "-1");
    config.addDataSourceProperty("DB_CLOSE_ON_EXIT", "FALSE");
    config.addDataSourceProperty("CACHE_SIZE", "65536");
    config.addDataSourceProperty("LOCK_TIMEOUT", "10000");
}
```

### Monitoring

**Micrometer integration** (if MeterRegistry available):

```java
config.setMetricsTrackerFactory(new MicrometerMetricsTrackerFactory(meterRegistry));
```

**Available metrics**:
- `hikaricp.connections.active`
- `hikaricp.connections.idle`
- `hikaricp.connections.pending`
- `hikaricp.connections.timeout`
- `hikaricp.connections.total`
- `hikaricp.connections.max`
- `hikaricp.connections.min`

### Password Resolution

**Priority order**:
1. Environment variable: `DB_PASSWORD`
2. System property: `db.password`
3. Application property: `app.database.password`

```java
private String getResolvedPassword() {
    if (StringUtils.hasText(System.getenv("DB_PASSWORD"))) {
        return System.getenv("DB_PASSWORD");
    }
    if (StringUtils.hasText(System.getProperty("db.password"))) {
        return System.getProperty("db.password");
    }
    return databaseProperties.getPassword();
}
```

**Security**: Passwords masked in logs using regex.

### Pool Naming

**Format**: `Genesis-HikariCP-{profile}`

**Examples**:
- `Genesis-HikariCP-dev`
- `Genesis-HikariCP-prod`
- `Genesis-HikariCP` (no profile)

---

## 4. WebServerConfiguration

**File**: `com.resetrix.genesis.configurations.WebServerConfiguration`

### Purpose
Configures Tomcat embedded server with extended timeouts for long-running operations.

### Async Support

```java
@Override
public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
    configurer.setDefaultTimeout(-1L);  // -1 = unlimited
}
```

**Effect**: Spring MVC async requests never timeout.

### Tomcat Connector Settings

```java
factory.addConnectorCustomizers(connector -> {
    // Timeouts (0 = unlimited)
    connector.setProperty("connectionTimeout", "0");
    connector.setProperty("keepAliveTimeout", "0");
    connector.setProperty("asyncTimeout", "0");
    
    // Connection limits
    connector.setProperty("maxConnections", "8192");
    connector.setProperty("acceptCount", "100");
    
    // Thread pool
    connector.setProperty("maxThreads", "200");
    connector.setProperty("minSpareThreads", "10");
});
```

**Configuration breakdown**:

| Property | Value | Purpose |
|----------|-------|---------|
| `connectionTimeout` | 0 (unlimited) | No timeout for connections |
| `keepAliveTimeout` | 0 (unlimited) | Keep connections alive indefinitely |
| `asyncTimeout` | 0 (unlimited) | No timeout for async requests |
| `maxConnections` | 8192 | Maximum concurrent connections |
| `acceptCount` | 100 | Queue length for incoming connections |
| `maxThreads` | 200 | Maximum request processing threads |
| `minSpareThreads` | 10 | Minimum idle threads |

**Why unlimited timeouts?**
- Schedule generation: 15-30 minutes
- Feign calls to engine: up to 45 minutes
- Prevents premature connection termination
- Client controls timeout, not server

---

## 5. CognitoConfiguration

**File**: `com.resetrix.genesis.configurations.CognitoConfiguration`

### Purpose
Configures AWS Cognito client for user authentication and management.

### Bean Configuration

```java
@Bean
public CognitoIdentityProviderClient cognitoClient(CognitoProperty cognitoProperty) {
    return CognitoIdentityProviderClient.builder()
        .region(Region.of(cognitoProperty.getRegion()))
        .build();
}
```

**Uses AWS SDK v2** for Cognito operations:
- User creation
- User deletion
- User attribute updates
- Password resets

### Properties

**From `application.yaml`**:
```yaml
app:
  cognito:
    region: us-east-1
    user-pool-id: ${COGNITO_USER_POOL_ID}
    client-id: ${COGNITO_CLIENT_ID}
```

---

## 6. SecurityConfiguration

**File**: `com.resetrix.genesis.configurations.SecurityConfiguration`

### Purpose
Configures Spring Security with JWT authentication and endpoint protection.

### Key Features

1. **JWT Token Validation**
   - Validates JWT signature using JWKS from Cognito
   - Extracts user claims (sub, email, custom:companyId)
   - Converts to Spring Security authorities

2. **CORS Configuration**
   - Allows cross-origin requests
   - Configurable allowed origins

3. **Endpoint Protection**
   - Public endpoints: `/api/v1/auth/**`
   - Protected endpoints: All others require authentication

4. **IP Whitelisting**
   - Engine callback endpoints restricted by IP
   - Configured via `app.security.engine-ip-whitelist`

---

## Configuration Properties

### Database Properties

**Prefix**: `app.database`

```yaml
app:
  database:
    enabled: true
    url: jdbc:postgresql://localhost:5432/genesis
    username: genesis_user
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 10
      minimum-idle: 10
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      leak-detection-threshold: 0
```

### Cognito Properties

**Prefix**: `app.cognito`

```yaml
app:
  cognito:
    region: us-east-1
    user-pool-id: ${COGNITO_USER_POOL_ID}
    client-id: ${COGNITO_CLIENT_ID}
```

### Security Properties

**Prefix**: `app.security`

```yaml
app:
  security:
    engine-ip-whitelist: 192.168.1.100,10.0.0.50
```

---

## Summary

All 6 configuration classes work together to provide:

✅ **Async Processing**: 2 dedicated thread pools  
✅ **External Integration**: Feign with 45-min timeouts  
✅ **Database**: HikariCP with PostgreSQL optimizations  
✅ **Web Server**: Unlimited timeouts for long operations  
✅ **Authentication**: AWS Cognito with JWT validation  
✅ **Security**: Endpoint protection and IP whitelisting  

All configurations are production-ready and verified from actual codebase.
