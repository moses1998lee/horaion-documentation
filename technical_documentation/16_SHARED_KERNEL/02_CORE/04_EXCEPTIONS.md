# 04 - Core Exceptions

> **Global Error Handling Strategy**

A robust API must have a predictable error strategy. We do **not** return random stack traces to the client. Instead, we conform to the **RFC 7807 Problem Details** standard.

---

## Error Handling Flow

The following diagram illustrates how an exception travels from the depth of the code to the user.

```mermaid
flowchart TD
    Code[Service Code] -->|Throw Exception| Catch(GlobalExceptionHandler)
    
    Catch -->|Analyze| Type{Exception Type?}
    
    Type -->|ResourceNotFound| 404[Build 404 Not Found]
    Type -->|BadRequest| 400[Build 400 Bad Request]
    Type -->|Unauthorized| 401[Build 401 Unauthorized]
    Type -->|runtimeException| 500[Build 500 Server Error]
    
    404 --> Response[Standard JSON Response]
    400 --> Response
    401 --> Response
    500 --> Response
```

> **Diagram Explanation**: The Horaion error handling chain ensures no raw stack traces ever leak to the client:
> 1.  **Interception**: The `GlobalExceptionHandler` (using `@RestControllerAdvice`) functions as a prioritized interceptor for all controller-thrown exceptions.
> 2.  **Normalization**: It maps varied Java exceptions to a clean set of HTTP status codes (404, 400, etc.) based on our RFC-compliant standard.
> 3.  **Serialization**: The final output is a standardized `ProblemDetail` JSON, ensuring that FE developers only need to learn one error structure for the entire platform.

### Flow Description
1.  **Throw**: A method (e.g., `EmployeeService`) detects an error condition and throws a Java Exception.
2.  **Catch**: The `GlobalExceptionHandler` interceptor catches the exception before it reaches the Tomcat container. It acts as a safety net.
3.  **Analyze**: The handler checks the *class* of the exception.
    *   If it's one of our `BaseException` children, it uses the specific status code defined there.
    *   If it's an unknown `RuntimeException` (like a NullPointerException), it defaults to a **500 Internal Server Error** to protect the system's internal state.
4.  **Response**: Finally, it constructs the standardized JSON payload so the Frontend always gets a consistent structure, regardless of what went wrong.

---

## 1. The Hierarchy (`BaseException`)
**File**: ``BaseException.java``

All custom exceptions in Horaion extend this abstract class. This allows us to enforce that every custom exception *must* provide a user-friendly message.

### Common Subclasses

| Exception | HTTP Status | Use Case |
| :--- | :--- | :--- |
| `ResourceNotFoundException` | **404** | "Employee with ID 5 does not exist." |
| `BadRequestException` | **400** | "Start date cannot be after End date." |
| `ConflictException` | **409** | "Email address is already in use." |
| `ForbiddenException` | **403** | "Manager cannot view Admin logs." |

{% hint style="success" %}
**Tip:** Always prefer throwing a specific exception (e.g., `ResourceNotFoundException`) over a generic `RuntimeException`. This helps the `GlobalExceptionHandler` pick the right HTTP Status Code.
{% endhint %}

---

## 2. Global Exception Handler
**File**: ``GlobalExceptionHandler.java``

This class uses Spring's `@RestControllerAdvice` to act as a "Catch-All" net.

### The Problem Detail Format
When an error occurs, the client receives a structured JSON:

```json
{
  "type": "about:blank",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "Shift template with UUID 'abc-123' not found.",
  "instance": "/api/v1/shifts/abc-123",
  "timestamp": "2023-10-27T10:00:00Z"
}
```

{% hint style="danger" %}
**Critical:** Never swallow exceptions in your Service code with an empty `catch` block. If you catch an exception, either handle it properly or rethrow it wrapped in a `BaseException`.
{% endhint %}
