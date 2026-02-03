# Logging Aspect API Documentation

## Overview
The `LogAspect` provides cross-cutting logging functionality through Spring AOP. It intercepts method executions in services and controllers to provide consistent logging and metrics.

## Core Components

### 1. LogAspect Class
**Package**: `com.horaion.app.shared.logging.aspects`

**Annotations**:
- `@Aspect`: Marks as Spring AOP aspect
- `@Component`: Spring bean
- `@ConditionalOnProperty`: Conditionally enabled via configuration

**Dependencies**:
- `MeterRegistry`: For metrics collection
- `LogAspectProperties`: Configuration properties

### 2. Pointcut Definitions

#### Service Methods
```java
@Pointcut("execution(public * com.horaion.app..services..*(..))")
public void serviceMethods()
```
Intercepts all public methods in any service class under `com.horaion.app` package.

#### Controller Methods
```java
@Pointcut("within(@org.springframework.web.bind.annotation.RestController *) || " +
          "within(@org.springframework.stereotype.Controller *)")
public void controllerMethods()
```
Intercepts all methods in classes annotated with `@RestController` or `@Controller`.

### 3. Advice Method
```java
@Around("serviceMethods() || controllerMethods()")
public Object logExecution(ProceedingJoinPoint joinPoint) throws Throwable
```
The main advice that wraps method execution with logging and metrics.

## Method Execution Flow

### Success Path
1. Generate/retrieve request ID
2. Extract user context
3. Capture HTTP context (if available)
4. Execute method
5. Record success metrics
6. Log execution time
7. Clean up MDC context
8. Return result

### Failure Path
1. Steps 1-4 same as success
2. Catch exception
3. Record failure metrics
4. Log error with duration
5. Handle exception type:
   - `Error`: Re-throw directly
   - `RuntimeException`: Re-throw directly
   - Checked exception: Wrap in `MethodExecutionException`
6. Clean up MDC context
7. Throw exception

## Configuration Properties

### LogAspectProperties Record
```java
public record LogAspectProperties(
    boolean enabled,
    long slowExecutionThresholdMs
)
```

**Defaults**:
- `enabled`: `true` (via `matchIfMissing = true`)
- `slowExecutionThresholdMs`: 1000ms (from `AspectConstant.SLOW_EXECUTION_THRESHOLD_MS`)

### YAML Configuration
```yaml
app:
  logging:
    aspect:
      enabled: true
      slow-execution-threshold-ms: 1000
```

## Constants

### AspectConstant Class
```java
public final class AspectConstant {
    public static final long SLOW_EXECUTION_THRESHOLD_MS = 1000;
    
    public static final String MDC_REQUEST_ID = "requestId";
    public static final String MDC_METHOD = "method";
    public static final String MDC_USER_ID = "userId";
    public static final String MDC_HTTP_METHOD = "httpMethod";
    public static final String MDC_ENDPOINT = "endpoint";
    
    public static final String STATUS_TAG = "status";
}
```

## Exception Handling

### MethodExecutionException
Wraps checked exceptions thrown by intercepted methods.

**Constructor**:
```java
public MethodExecutionException(String methodName,
                                long executionTimeMs,
                                Throwable cause)
```

**Message Format**:
```
Execution failed for method {methodName} after {executionTimeMs} ms
```

## Metrics Integration

### Timer Registration
```java
meterRegistry
    .timer("app.method.execution", "method", method, STATUS_TAG, status)
    .record(durationMs, TimeUnit.MILLISECONDS);
```

**Tags**:
- `method`: Method signature (e.g., `UserService.createUser()`)
- `status`: `success` or `failure`

## Log Messages

### Success Messages
**Normal execution**:
```
Method {method} executed successfully in {duration} ms
```

**Slow execution** (≥ threshold):
```
Method {method} executed slowly in {duration} ms
```

### Error Messages
```
Method {method} failed in {duration} ms with {exceptionType}: {message}
```

## MDC (Mapped Diagnostic Context) Management

### Context Variables Set
| Variable | When Set | When Cleared |
|----------|----------|--------------|
| `requestId` | If not already present | If was set by this aspect |
| `method` | Always | Always |
| `userId` | If authenticated user exists | If was set |
| `httpMethod` | If HTTP request available | If was set |
| `endpoint` | If HTTP request available | If was set |

### Request ID Generation
```java
randomUUID()
    .toString()
    .replace("-", "")
    .substring(0, REQUEST_ID_LENGTH)
    .toUpperCase();
```
Generates 8-character uppercase ID (e.g., `A1B2C3D4`).