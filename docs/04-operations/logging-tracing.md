# Logging and Tracing Configuration

## Trace ID Management

### MDC Integration
The exception handling system integrates with SLF4J's Mapped Diagnostic Context (MDC) for distributed tracing:

```java
// Automatic trace ID management in ServiceOperationExecutor
private static String getOrCreateTraceId() {
    String existingTraceId = MDC.get(MDC_REQUEST_ID_KEY);
    
    if (existingTraceId != null && !existingTraceId.isBlank()) {
        return existingTraceId;
    }
    
    String newTraceId = randomUUID().toString();
    MDC.put(MDC_REQUEST_ID_KEY, newTraceId);
    return newTraceId;
}
```

### Configuration Constants
```properties
# Defined in ServiceOperationExecutorConstant
mdc.request.id.key=requestId
max.response.body.size=10000
response.truncation.indicator=

--- RESPONSE TRUNCATED (exceeded 10KB limit) ---
```

## Logging Patterns

### Structured Logging Format
```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "ERROR",
  "logger": "com.horaion.app.shared.core.utils.ServiceOperationExecutor",
  "thread": "http-nio-8080-exec-1",
  "message": "[trace-id-123] UPDATE operation failed with database exception: Constraint violation",
  "requestId": "trace-id-123",
  "operationType": "UPDATE",
  "exception": "DataIntegrityViolationException",
  "stackTrace": "..."
}
```

### Log Levels by Exception Type

| Exception Type | Log Level | Reason |
|----------------|-----------|---------|
| `BaseException` | ERROR | Application errors requiring attention |
| `ResourceNotFoundException` | WARN | Expected business condition |
| Validation errors | WARN | Client input issues |
| Database deadlocks | ERROR | Concurrency issues |
| External service errors | ERROR | Integration failures |
| Programmer errors | ERROR | Code defects |

### Operation-Specific Logging
```java
// Entry log
LOGGER.debug("[{}] Executing {} operation", traceId, operationType.name().toLowerCase());

// Success log
LOGGER.debug("[{}] {} operation completed successfully", traceId, operationType.name().toLowerCase());

// Error logs with context
LOGGER.error("[{}] {} operation failed with database exception: {}", 
    traceId, operationType, ex.getMessage(), ex);
```

## Response Body Truncation

### Configuration
```java
public static final int MAX_RESPONSE_BODY_SIZE = 10_000;
public static final String TRUNCATION_INDICATOR = 
    "\n\n--- RESPONSE TRUNCATED (exceeded 10KB limit) ---";
```

### Implementation
```java
private static String truncateResponseBody(String responseBody, String traceId) {
    if (responseBody.length() > MAX_RESPONSE_BODY_SIZE) {
        String truncated = responseBody.substring(0, MAX
```