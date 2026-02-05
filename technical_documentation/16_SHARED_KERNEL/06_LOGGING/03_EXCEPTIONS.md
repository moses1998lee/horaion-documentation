# 03 - Logging Exceptions

> **AOP Error Handling**

**File**: [`MethodExecutionException.java`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/logging/exceptions/MethodExecutionException.java)

This is a specific `RuntimeException` used by the AOP Aspect.

### When is it thrown?
The `LogAspect` wraps method execution in a try-catch block.
*   If the service throws a `RuntimeException` (e.g., `ResourceNotFoundException`), it is **rethrown as-is**.
*   If the service throws a `Checked Exception` (Exception) or `Error`:
    *   It is wrapped in `MethodExecutionException` to avoid breaking the runtime flow (since AOP proxies generally prefer unchecked exceptions).

{% hint style="info" %}
**Note:** In 99% of cases, you won't see this exception. It only appears if a method throws a checked exception that wasn't declared, or a fatal `Error` (like OOM) occurs that needs wrapping.
{% endhint %}
