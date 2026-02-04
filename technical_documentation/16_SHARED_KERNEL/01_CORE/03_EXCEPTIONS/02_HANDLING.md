# 02 - Handling & ProblemDetail

> **RFC 7807 Standard Error Responses**

---

## Overview

We use Spring 6's native support for **RFC 7807 (Problem Details for HTTP APIs)**. This replaces custom error envelopes with a standardized JSON structure.

**Handler**: `com.horaion.app.shared.core.exceptions.handlers.GlobalExceptionHandler`

---

## The Response Format (RFC 7807)

When an exception is thrown, the handler converts it into a `ProblemDetail` JSON object.

### Example: Validation Error
```json
{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "Validation failed",
  "instance": "/api/v1/employees",
  "errors": [
    {
      "field": "email",
      "message": "must be a well-formed email address"
    }
  ]
}
```

### Example: Business Error (BaseException)
```json
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "User with ID 550e8400-e29b not found",
  "instance": "/api/v1/users/550e8400-e29b",
  "errorCode": "USER_NOT_FOUND",
  "userId": "550e8400-e29b-41d4-a716-446655440000"
}
```

---

## Handler Logic

The `GlobalExceptionHandler` uses `@RestControllerAdvice` to catch exceptions globally.

### 1. Handling `BaseException`
It calls `ex.toProblemDetail()` which automatically populates:
*   Standard fields (`title`, `status`, `detail`).
*   Custom fields (`errorCode`, and any properties added via `addProperty()`).

### 2. Handling `MethodArgumentNotValidException`
Spring Validation errors (e.g., `@NotNull`) are caught here. The handler extracts field errors and maps them to a simplified list under the `errors` property.

{% hint style="info" %}
**Note:** The `type` field is currently set to `about:blank`. In the future, this can be updated to point to a documentation URL for the specific error code.
{% endhint %}
