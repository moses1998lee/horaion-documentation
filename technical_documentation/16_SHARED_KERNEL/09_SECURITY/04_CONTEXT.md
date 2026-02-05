# 04 - Developer Guide (Context)

> **Accessing User Information**

**File**: ``AuthorizationHelper.java``

Need to know *who* is making the request? Don't parse the token manually. Use this helper.

---

## Common Recipes

### 1. Get Current User ID (Sub)
Use this to link records to a user in the database.
```java
String userId = AuthorizationHelper.getRequiredUserSub();
// Returns: "a1b2c3d4-..." or throws Exception
```

### 2. Check Role
Avoid hardcoding role strings.
```java
if (AuthorizationHelper.isSystemAdministrator()) {
    // fast path...
}
```

### 3. Get Current Company/Branch
Use the `SecurityContextService` bean for this, as it requires database lookups to resolve the employee profile.

```java
@Autowired
private SecurityContextService securityContext;

public void doSomething() {
    UUID companyId = securityContext.getRequiredCurrentCompanyId();
    // Use companyId...
}
```

---

## Error Handling
The security package automatically handles exceptions:
*   `UnauthorizedResourceAccessException` -> **403 Forbidden** (JSON)
*   `AuthenticationException` -> **401 Unauthorized** (JSON)

{% hint style="danger" %}
**Critical:** Never catch these exceptions and swallow them. Let them bubble up to the `GlobalExceptionHandler` so the API client receives the correct status code.
{% endhint %}
