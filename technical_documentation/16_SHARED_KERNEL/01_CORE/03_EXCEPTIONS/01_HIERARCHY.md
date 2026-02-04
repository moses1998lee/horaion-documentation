# 01 - Exception Hierarchy

> **Standardized Runtime Exceptions with BaseException**

---

## The BaseException
The root of all custom application logic errors is `BaseException`. It forces developers to define:
1.  **HTTP Status**: The correct REST status code (e.g., 404, 409).
2.  **Error Code**: A string for client-side i18n mapping (e.g., `USER_NOT_FOUND`).
3.  **Message**: A developer-friendly English debug message.

**Class**: `com.horaion.app.shared.core.exceptions.types.BaseException`

```java
public class UserNotFoundException extends BaseException {
    public UserNotFoundException(UUID userId) {
        super(
            "User with ID " + userId + " not found", // Debug Message
            HttpStatus.NOT_FOUND,                    // 404
            "USER_NOT_FOUND"                         // Error Code
        );
        addProperty("userId", userId);               // Extra Context
    }
}
```

---

## Key Subclasses

| Exception | Status | Usage |
| :--- | :--- | :--- |
| **ResourceNotFoundException** | 404 | Entity lookup failed. |
| **ExternalServiceException** | 502 | A call to AWS/Coinbase/etc failed. |
| **InvalidFileException** | 400 | Malformed upload. |

---

## Custom Context Properties
`BaseException` allows injecting dynamic properties into the final JSON response using `addProperty(key, value)`.

{% hint style="success" %}
**Tip:** Always add context! Telling the client *which* ID failed (e.g., `userId: 123`) is infinitely more helpful than a generic "User not found" message.
{% endhint %}

{% hint style="danger" %}
**Critical:** Do not extend `RuntimeException` directly for business logic errors. If you do, the `GlobalExceptionHandler` will treat it as a generic 500 "Internal Server Error" and obfuscate the message. Always extend `BaseException`.
{% endhint %}
