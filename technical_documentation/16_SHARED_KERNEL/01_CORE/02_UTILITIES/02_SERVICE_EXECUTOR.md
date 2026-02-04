# 02 - ServiceOperationExecutor

> **The "Safe Runner" Pattern**

---

## Overview

The `ServiceOperationExecutor` is a standard wrapper designed to execute any functional block of code (Supplier) with consistent "plumbing" wrapped around it.

**Class**: `com.horaion.app.shared.core.utils.ServiceOperationExecutor`

It handles:
1.  **Tracing**: Initializes MDC with a `request_id` if missing.
2.  **Logging**: Logs "Executing..." and "Completed..." messages standardized by operation name.
3.  **Exception Translation**: Catches low-level exceptions (SQL, REST) and converts them to domain exceptions (`ExternalServiceException`, `DatabaseException`).

---

## Usage Pattern

Do not write raw `try-catch` blocks in your Services. Use the Executor.

```java
public EmployeeResponseDTO createEmployee(EmployeeRequestDTO request) {
    return ServiceOperationExecutor.execute(() -> {
        // ... Your Business Logic ...
        Employee entity = repository.save(mapper.toEntity(request));
        return mapper.toDTO(entity);
    }, 
    OperationStatus.SAVE, 
    exceptionFactory);
}
```

### Parameters
*   **Operation**: The lambda to execute.
*   **OperationType**: Enum (`SAVE`, `UPDATE`, `DELETE`, `RETRIEVE`) used for logging context.
*   **ExceptionFactory**: Factory to create domain exceptions with proper context.

---

## Exception translation

The Executor automatically maps technical errors to user-friendly messages:

| Source Exception | Translated To |
| :--- | :--- |
| `EntityNotFoundException` | `ResourceNotFoundException` |
| `DataIntegrityViolationException` | `ValidationException` (with constraint details) |
| `RestClientException` | `ExternalServiceException` (with truncated response body) |
| `LockingFailureException` | `DatabaseException` ("Please retry") |

{% hint style="warning" %}
**Important:** If you throw a `RuntimeException` that is a "Strict Programmer Error" (e.g., `NullPointerException`), the Executor will **NOT** wrap it. It bubbles up so you can fix the bug.
{% endhint %}

{% hint style="info" %}
**Debugging:** New trace IDs are generated automatically if one doesn't exist. Check `MDC_REQUEST_ID_KEY` in your logs to trace a request across threads.
{% endhint %}
