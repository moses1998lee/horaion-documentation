# 02 - Shared Core

> **Standardized Responses, Exceptions, and Utilities**

---

## 2.1 Standard API Responses

To ensure consistency across the entire API surface, every endpoint MUST return data wrapped in standard "Envelope" objects.

### ApiResponse `<T>`
**Class**: `com.horaion.app.shared.core.responses.ApiResponse`

This generic wrapper is used for single-item responses or simple success messages.

```json
{
  "success": true,
  "data": {
    "id": "123",
    "name": "Project Horaion"
  },
  "message": "Operation successful",
  "timestamp": "2023-10-27T10:00:00Z",
  "path": "/api/v1/projects"
}
```

**Usage**:
```java
// Logic Controller
@GetMapping("/{id}")
public ApiResponse<ProjectDTO> getProject(@PathVariable UUID id) {
    ProjectDTO project = service.getById(id);
    return ApiResponse.success(project, "Project retrieved", "/api/v1/projects/" + id);
}
```

### PagedResponse `<T>`
**Class**: `com.horaion.app.shared.core.responses.PagedResponse`

This wrapper is mandatory for all paginated endpoints. It standardizes the pagination metadata block.

```json
{
  "content": [
    { "id": 1, "name": "Item A" },
    { "id": 2, "name": "Item B" }
  ],
  "page": {
    "number": 0,
    "size": 10,
    "totalElements": 100,
    "totalPages": 10,
    "first": true,
    "last": false,
    "hasNext": true,
    "hasPrevious": false
  }
}
```

**Factory Methods**:
*   `PagedResponse.of(Page<T> page)`: Wraps a Spring Data Page object.
*   `PagedResponse.of(Page<T> page, Function<T, R> mapper)`: Wraps and converts content (e.g., Entity -> DTO).

---

## 2.2 Global Exception Handling

The application uses **RFC 7807 Problem Details** for error responses, leveraging Spring Boot 3's native support. This differs from the success response envelope.

### Success vs. Error
*   **Success**: Returns `ApiResponse<T>` (Custom Envelope).
*   **Error**: Returns `ProblemDetail` (Standard JSON).

### GlobalExceptionHandler
**Class**: `com.horaion.app.shared.core.exceptions.handlers.GlobalExceptionHandler`

This `@RestControllerAdvice` catches exceptions and converts them into `ProblemDetail` objects.

**Example Error Response**:
```json
{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "Invalid value for field 'email'. Expected type: String",
  "instance": "/api/v1/employees",
  "errors": [
    {
      "field": "email",
      "message": "must be a well-formed email address"
    }
  ]
}
```

### Hierarchy
All custom exceptions extend `BaseException`, which carries the `HttpStatus`.

| Exception Type | HTTP Status | Use Case |
| :--- | :--- | :--- |
| `ResourceNotFoundException` | 404 | Entity not found. |
| `ValidationException` | 400 | Business rule violation. |
| `PermitAuthorizationException` | 403 | RBAC denial. |

---

## 2.3 Constants & Enums

### AppConstants
**Class**: `com.horaion.app.shared.core.constants.AppConstants`

Central repository for system-wide constants.

*   `DEFAULT_PAGE_NUMBER`: "0"
*   `DEFAULT_PAGE_SIZE`: "10"
*   `MAX_PAGE_SIZE`: "50"
*   `API_VERSION_V1`: "/api/v1"

Using these ensures that if we decide to change the default page size, we do it in one place.
