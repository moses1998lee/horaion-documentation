# Exception Handling Architecture

## Overview
This segment introduces a comprehensive exception handling framework with centralized error management, structured error responses, and a service operation executor for consistent error handling across the application.

## Architecture Components

### 1. Base Exception Hierarchy

```mermaid
classDiagram
    class BaseException {
        <<abstract>>
        -HttpStatus httpStatus
        -String errorCode
        -Map~String,Object~ customProperties
        +addProperty(String, Object) BaseException
        +toProblemDetail(String) ProblemDetail
    }
    
    class ExternalServiceException {
        +ExternalServiceException(String, Throwable, Integer, String)
    }
    
    class ResourceNotFoundException {
        +ResourceNotFoundException(String)
    }
    
    class InvalidFileException {
        +InvalidFileException(String, Throwable)
    }
    
    BaseException <|-- ExternalServiceException
    BaseException <|-- ResourceNotFoundException
    RuntimeException <|-- InvalidFileException
```

**Key Classes:**
- **BaseException**: Abstract base class providing consistent error structure with HTTP status, error codes, and custom properties
- **ExternalServiceException**: For external service failures with status code mapping and response body capture
- **ResourceNotFoundException**: Standard 404 resource not found errors
- **InvalidFileException**: File validation and processing errors

### 2. Global Exception Handler

**GlobalExceptionHandler** extends `ResponseEntityExceptionHandler` to provide centralized exception handling:

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
    // Handles BaseException hierarchy
    @ExceptionHandler(BaseException.class)
    public ResponseEntity<ProblemDetail> handleBaseException(...)
    
    // Handles validation errors from @Valid
    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(...)
    
    // Handles constraint violations
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ProblemDetail> handleConstraintViolationException(...)
}
```

**Response Format (RFC 7807 Problem Details):**
```json
{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "Validation failed",
  "instance": "/api/users",
  "errorCode": "VALIDATION_ERROR",
  "errors": [
    {
      "field": "email",
      "message": "must be a valid email"
    }
  ]
}
```

### 3. Service Operation Executor

**ServiceOperationExecutor** provides a unified pattern for executing service operations with consistent error handling:

```mermaid
graph TD
    A[Start Operation] --> B{Execute Operation}
    B --> C[Success]
    B --> D[Exception]
    
    D --> E{Exception Type}
    E --> F[Programmer Error<br/>NullPointer, IllegalState, etc.]
    E --> G[Database Exception]
    E --> H[RestClient Exception]
    E --> I[Validation Exception]
    E --> J[Unexpected Exception]
    
    F --> K[Bubble Up]
    G --> L[Map to User Message]
    H --> M[Create ExternalServiceException]
    I --> N[Re-throw]
    J --> O[Wrap with Context]
    
    K --> P[Return/Throw]
    L --> P
    M --> P
    N --> P
    O --> P
```

**Key Features:**
- **Trace ID Management**: Automatic correlation ID generation/retrieval from MDC
- **Exception Categorization**: 
  - Programmer errors (bubbled up)
  - Database exceptions (mapped to user-friendly messages)
  - External service errors (wrapped with context)
  - Validation errors (preserved)
- **Response Body Truncation**: Limits external service response bodies to 10KB
- **Operation Context**: Includes operation type (RETRIEVE, SAVE, UPDATE, DELETE, OTHER) in logs

**Database Exception Mapping:**
- `OptimisticLockingFailureException` → "Concurrent modification detected"
- `DeadlockLoserDataAccessException` → "Database deadlock detected"
- `EntityNotFoundException` → Operation-specific messages
- `DataIntegrityViolationException` → Constraint-specific messages

### 4. Constants and Enums

**ServiceOperationExecutorConstant:**
```java
public final class ServiceOperationExecutorConstant {
    public static final String TRUNCATION_INDICATOR = "\n\n--- RESPONSE TRUNCATED (exceeded 10KB limit) ---";
    public static final String MDC_REQUEST_ID_KEY = "requestId";
    public static final int MAX_RESPONSE_BODY_SIZE = 10_000;
}
```

**OperationStatus Enum:**
```java
public enum OperationStatus {
    RETRIEVE, SAVE, UPDATE, DELETE, OTHER
}
```

## Design Decisions

### 1. RFC 7807 Compliance
- Uses Spring's `ProblemDetail` for standardized error responses
- Consistent `about:blank` type for application-specific errors
- Structured error details with field-level validation information

### 2. Centralized Error Handling
- Single `GlobalExceptionHandler` for all REST controllers
- Consistent logging with trace IDs
- Structured error responses across all endpoints

### 3. Defensive Programming
- `ServiceOperationExecutor` validates all inputs (non-null checks)
- Programmer errors (NPE, IllegalState) are bubbled up for immediate fixing
- Unexpected exceptions are wrapped with context for debugging

### 4. External Service Integration
- `ExternalServiceException` captures HTTP status, response body, and cause
- Response body truncation prevents memory issues with large error responses
- Status code mapping to appropriate HTTP statuses

## Usage Examples

### Basic Service Operation
```java
public User getUser(String id) {
    return ServiceOperationExecutor.execute(
        () -> userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found")),
        OperationStatus.RETRIEVE,
        (message, cause) -> new RuntimeException("Failed to retrieve user", cause)
    );
}
```

### Void Operation
```java
public void deleteUser(String id) {
    ServiceOperationExecutor.executeVoid(
        () -> userRepository.deleteById(id),
        OperationStatus.DELETE,
        (message, cause) -> new RuntimeException("Failed to delete user", cause)
    );
}
```

### Custom Exception with Properties
```java
throw new ExternalServiceException(
    "Payment service failed",
    cause,
    503,
    "{\"error\": \"service_unavailable\"}"
).addProperty("retryAfter", "300s");
```

## Dependencies Added
- `spring-boot-starter-validation`: For bean validation and constraint violation handling

## Testing Strategy
Comprehensive test coverage includes:
- `GlobalExceptionHandlerTest`: Handler behavior with various exception types
- `BaseExceptionTest`: ProblemDetail creation and property management
- `ExternalServiceExceptionTest`: Status code mapping and property handling
- `ServiceOperationExecutorTest`: Full branch coverage of exception handling logic

## Security Considerations
- Response body truncation prevents exposure of potentially sensitive error details
- Structured error responses avoid information leakage
- Trace IDs in logs enable debugging without exposing internal details to users