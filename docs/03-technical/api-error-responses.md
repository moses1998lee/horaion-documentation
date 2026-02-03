# API Error Responses

## Error Response Format

All API errors follow the RFC 7807 Problem Details format with application-specific extensions.

### Standard Error Structure
```json
{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "Validation failed for request parameters",
  "instance": "/api/v1/users",
  "errorCode": "VALIDATION_ERROR",
  "errors": [
    {
      "field": "email",
      "message": "must be a valid email address"
    },
    {
      "field": "age",
      "message": "must be greater than or equal to 18"
    }
  ]
}
```

### Field Descriptions

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `type` | URI | Error type (always "about:blank" for application errors) | "about:blank" |
| `title` | String | Human-readable error title | "Bad Request" |
| `status` | Integer | HTTP status code | `400` |
| `detail` | String | Detailed error message | "Validation failed for request parameters" |
| `instance` | URI | Path where error occurred | "/api/v1/users" |
| `errorCode` | String | Application-specific error code | "VALIDATION_ERROR" |
| `errors` | Array | Field-level validation errors (when applicable) | See above |
| `*` | Any | Additional custom properties | "retryAfter": "300s" |

## Common Error Types

### 1. Validation Errors (400)
**Triggered by:** `@Valid` annotations, `ConstraintViolationException`
```json
{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "Validation failed",
  "instance": "/api/v1/users",
  "errorCode": "VALIDATION_ERROR",
  "errors": [
    {
      "field": "user.email",
      "message": "must be a valid email"
    }
  ]
}
```

### 2. Resource Not Found (404)
**Triggered by:** `ResourceNotFoundException`
```json
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "User with ID '123' not found",
  "instance": "/api/v1/users/123",
  "errorCode": "RESOURCE_NOT_FOUND"
}
```

### 3. External Service Errors
**Triggered by:** `ExternalServiceException`
```json
{
  "type": "about:blank",
  "title": "Service Unavailable",
  "status": 503,
  "detail": "Payment service unavailable",
  "instance": "/api/v1/payments",
  "errorCode": "EXTERNAL_SERVICE_ERROR",
  "httpStatusCode": 503,
  "responseBody": "{\"error\": \"service_down\"}",
  "causeType": "ConnectException"
}
```

### 4. Database Errors
**Triggered by:** Various database exceptions via `ServiceOperationExecutor`
```json
{
  "type": "about:blank",
  "title": "Conflict",
  "status": 409,
  "detail": "Concurrent modification detected - please retry the operation",
  "instance": "/api/v1/orders/456",
  "errorCode": "CONCURRENT_MODIFICATION"
}
```

## Custom Error Properties

### Adding Custom Properties
```java
throw new ExternalServiceException(
    "Service failed",
    cause,
    500,
    responseBody
)
.addProperty("retryAfter", "300s")
.addProperty("serviceName", "payment-service")
.addProperty("requestId", "req-12345");
```

### Property Guidelines
1. **Use descriptive names**: `retryAfter` not `retry`
2. **Include relevant context**: Service names, IDs, timestamps
3. **Avoid sensitive data**: No passwords, tokens, PII
4. **Keep it minimal**: Only include properties useful for debugging

## HTTP Status Code Mapping

### Standard Mappings
| Exception Type | HTTP Status | Typical Scenario |
|----------------|-------------|------------------|
| `ResourceNotFoundException` | 404 NOT FOUND | Entity doesn't exist |
| `ConstraintViolationException` | 400 BAD_REQUEST | Validation failed |
| `MethodArgumentNotValidException` | 400 BAD_REQUEST | Request body invalid |
| `ExternalServiceException` | Mapped from status | External API failure |
| Database deadlock | 409 CONFLICT | Concurrent modification |
| Database constraint | 409 CONFLICT | Unique violation |

### External Service Status Mapping
The `ExternalServiceException` maps HTTP status codes:
- 4xx client errors → Same status (400, 404, etc.)
- 5xx server errors → Same status (500, 503, etc.)
- Unrecognized codes → 500 INTERNAL_SERVER_ERROR

## Error Response Headers

All error responses include:
- `Content-Type: application/problem+json`
- `X-Request-ID`: Trace ID for correlation (from MDC)

## Client Handling Recommendations

### 1. Parse Error Response
```javascript
try {
  const response = await fetch('/api/resource');
  if (!response.ok) {
    const error = await response.json();
    console.error(`Error ${error.status}: ${error.detail}`);
    
    if (error.errors) {
      error.errors.forEach(err => {
        console.error(`Field ${err.field}: ${err.message}`);
      });
    }
  }
} catch (error) {
  // Handle network errors
}
```

### 2. Retry Logic
```javascript
function shouldRetry(error) {
  // Retry on 5xx errors, deadlocks, and service unavailable
  return error.status >= 500 || 
         error.errorCode === 'CONCURRENT_MODIFICATION' ||
         error.errorCode === 'EXTERNAL_SERVICE_ERROR';
}

function getRetryDelay(error) {
  // Use retryAfter property if available
  return error.retryAfter ? parseInt(error.retryAfter) : 1000;
}
```

### 3. User Display
```javascript
function getUserMessage(error) {
  switch (error.errorCode) {
    case 'RESOURCE_NOT_FOUND':
      return 'The requested item was not found.';
    case 'VALIDATION_ERROR':
      return 'Please check your input and try again.';
    case 'EXTERNAL_SERVICE_ERROR':
      return 'Service temporarily unavailable. Please try again later.';
    default:
      return 'An unexpected error occurred.';
  }
}
```

## Testing Error Responses

### Unit Test Example
```java
@Test
void testValidationErrorResponse() {
    // Given invalid request
    UserRequest invalidRequest = new UserRequest("invalid-email", -5); 
    
    // When making request
    ResponseEntity<ProblemDetail> response = restTemplate.postForEntity(
        "/api/users", invalidRequest, ProblemDetail.class);
    
    // Then verify error structure
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
    assertThat(response.getBody().getDetail()).contains("Validation failed");
    assertThat(response.getBody().getProperties()).containsKey("errors");
}
```

### Integration Test Example
```java
@Test
void testExternalServiceErrorPropagation() {
    // Mock external service to fail
    mockServer.expect(requestTo("/external/api"))
        .andRespond(withStatus(HttpStatus.SERVICE_UNAVAILABLE));
    
    // When calling endpoint that uses external service
    ResponseEntity<ProblemDetail> response = restTemplate.getForEntity(
        "/api/data", ProblemDetail.class);
    
    // Then verify error details
    assertThat(response.getStatusCode())
        .isEqualTo(HttpStatus.SERVICE_UNAVAILABLE);
    assertThat(response.getBody().getErrorCode())
        .isEqualTo("EXTERNAL_SERVICE_ERROR");
}
```

## Monitoring and Logging

### Structured Logging
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "logger": "GlobalExceptionHandler",
  "message": "BaseException: External service failed",
  "requestId": "req-12345",
  "exception": "ExternalServiceException",
  "httpStatus": 503,
  "path": "/api/payments",
  "stackTrace": "..."
}
```

### Metrics to Monitor
1. **Error rate by type**: `validation_errors_total`, `external_service_errors_total`
2. **Error rate by endpoint**: `errors_total{path="/api/users"}`
3. **External service status**: `external_service_status{service="payment"}`
4. **Database error types**: `database_errors_total{type="deadlock"}`