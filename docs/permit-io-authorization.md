# Permit.io Authorization - Knowledge Transfer

This document explains how Permit.io is integrated into the Horaion application for authorization using pure ABAC (Attribute-Based Access Control) without user synchronization.

## Architecture Overview

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Spring Boot    │     │  Permit.io      │     │  Permit.io      │
│  Application    │────▶│  Local PDP      │◀────│  Cloud          │
│                 │     │  (Docker)       │     │  (Policy Sync)  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │
        │ JWT Token
        ▼
┌─────────────────┐
│  AWS Cognito    │
│  (Authentication)│
└─────────────────┘
```

### Key Design Decisions

1. **Pure ABAC** - No RBAC roles are used. Authorization is based solely on user attributes.
2. **No User Sync** - Users are NOT synced to Permit.io. Roles come from JWT claims at check time.
3. **Local PDP** - Uses local Policy Decision Point (Docker container) for authorization decisions.
4. **Cognito Groups as Attributes** - JWT `cognito:groups` claim is passed as `user.groups` attribute.

## Role Hierarchy

| Cognito Group           | Description                    | Precedence | 
|-------------------------|--------------------------------|------------|
| system-administrator    | Full system access             | 1 (Highest)|
| system-owner            | Organization owner access      | 2          |
| privileged-system-user  | Elevated user access           | 3          |
| user                    | Basic user access              | 4 (Lowest) |

## Component Overview

### 1. @PermitCheck Annotation

Location: `src/main/java/com/horaion/app/shared/security/annotations/PermitCheck.java`

Annotation to perform authorization checks on controller methods.

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface PermitCheck {
    String action();           // e.g., "read", "create", "update", "delete"
    String resource();         // e.g., "company", "branch", "department"
    String resourceIdParam() default "";  // Optional: extract ID from method param
    String tenant() default "";           // Optional: tenant override
}
```

**Usage Example:**

```java
@RestController
@RequestMapping("/api/v1/companies")
public class CompanyController {

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @PermitCheck(action = "create", resource = "company")
    public CompanyResponse create(@Valid @RequestBody CreateCompanyRequest request) {
        return companyService.create(request);
    }

    @GetMapping("/{id}")
    @PermitCheck(action = "read", resource = "company")
    public CompanyResponse getById(@PathVariable UUID id) {
        return companyService.getById(id);
    }

    @DeleteMapping("/{id}")
    @PermitCheck(action = "delete", resource = "company", resourceIdParam = "id")
    public void delete(@PathVariable UUID id) {
        companyService.delete(id);
    }
}
```

### 2. PermitCheckAspect

Location: `src/main/java/com/horaion/app/shared/security/aspects/PermitCheckAspect.java`

AOP aspect that intercepts `@PermitCheck` annotated methods and performs authorization checks.

**Processing Flow:**
1. Extract annotation parameters (action, resource, resourceIdParam, tenant)
2. If `resourceIdParam` is specified, extract resource ID from method arguments
3. Build `PermitCheckRequest`
4. Call `PermitAuthorizationService.checkOrThrow()`
5. If authorized, proceed with method execution
6. If denied, throw `PermitAuthorizationException` (returns 403 Forbidden)

### 3. PermitAuthorizationService

Location: `src/main/java/com/horaion/app/shared/security/services/PermitAuthorizationService.java`

Core service that communicates with Permit.io PDP.

**Key Methods:**

- `checkOrThrow(PermitCheckRequest request)` - Checks authorization, throws exception if denied
- `isAllowed(PermitCheckRequest request)` - Returns boolean authorization result

**User Building Process:**

```java
private User buildPermitUser(PermitCheckRequest request) {
    String userKey = AuthorizationHelper.getUserSub().orElse("anonymous");

    HashMap<String, Object> attributes = new HashMap<>();

    // Extract Cognito groups from JWT
    List<String> cognitoGroups = AuthorizationHelper.getCognitoGroups().orElse(List.of());

    // Set groups attribute for ABAC User Sets matching
    // This is the KEY attribute that User Sets use for dynamic role matching
    attributes.put("groups", cognitoGroups);

    // Additional attributes
    attributes.put("role", determinePrimaryRole(permitRoles));
    attributes.put("roles", permitRoles);
    attributes.put("email", jwt.getClaimAsString("email"));
    attributes.put("username", jwt.getClaimAsString("cognito:username"));

    return new User.Builder(userKey)
        .withAttributes(attributes)
        .build();
}
```

### 4. Configuration Properties

Location: `src/main/java/com/horaion/app/shared/infrastructure/properties/PermitProperty.java`

```java
@ConfigurationProperties(prefix = "permit")
public class PermitProperty {
    private boolean enabled = true;           // Enable/disable authorization
    private String apiKey;
    private String pdpAddress = "http://localhost:7766";  // PDP address
    private String defaultTenant = "default"; // Default tenant
    private boolean debugLogging = false;     // Debug mode
}
```

**Environment Variables:**

```bash
PERMIT_ENABLED=true
PERMIT_API_KEY=permit_key_xxxxx
PERMIT_PDP_ADDRESS=http://localhost:7766
PERMIT_DEFAULT_TENANT=default
PERMIT_DEBUG_LOGGING=false
```

## Permit.io Policy Configuration

### Resources

Resources represent the entities that can be protected. Define in Permit.io dashboard:

| Resource   | Actions                        |
|------------|--------------------------------|
| company    | create, read, update, delete   |
| branch     | create, read, update, delete   |
| department | create, read, update, delete   |
| employee   | create, read, update, delete   |
| rule       | create, read, update, delete   |

### User Sets (ABAC Condition Sets)

User Sets define conditions to dynamically match users based on attributes. This is how ABAC works without syncing users.

**Example User Sets:**

| User Set Name           | Condition                                           |
|-------------------------|-----------------------------------------------------|
| system-administrators   | `user.groups array_contains "system-administrator"` |
| system-owners           | `user.groups array_contains "system-owner"`         |
| privileged-system-users | `user.groups array_contains "privileged-system-user"`|
| basic-users             | `user.groups array_contains "user"`                 |

**Creating a User Set via Permit.io API:**

```bash
curl -X POST "https://api.permit.io/v2/schema/{project_id}/{env_id}/condition_sets" \
  -H "Authorization: Bearer permit_key_xxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "key": "system-owners",
    "name": "System Owners",
    "type": "userset",
    "conditions": {
      "allOf": [
        {
          "allOf": [
            {
              "user.groups": {
                "array_contains": "system-owner"
              }
            }
          ]
        }
      ]
    }
  }'
```

### Set Rules (Permissions)

Set Rules grant permissions to User Sets. They link User Sets to resource actions.

**Example Set Rules:**

| User Set              | Resource   | Actions                       |
|-----------------------|------------|-------------------------------|
| system-administrators | *          | create, read, update, delete  |
| system-owners         | company    | create, read, update, delete  |
| system-owners         | branch     | create, read, update, delete  |
| system-owners         | department | create, read, update, delete  |
| privileged-system-users| employee  | create, read, update          |
| basic-users           | *          | read                          |

**Creating a Set Rule via Permit.io API:**

```bash
curl -X POST "https://api.permit.io/v2/facts/{project_id}/{env_id}/set_rules" \
  -H "Authorization: Bearer permit_key_xxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "permission": "company:create",
    "user_set": "system-owners",
    "is_role": false
  }'
```

## Local PDP Setup

### Running the PDP Container

```bash
docker run -d \
  --name permit-pdp \
  -p 7766:7000 \
  -e PDP_API_KEY=permit_key_xxxxx \
  -e PDP_DEBUG=true \
  permitio/pdp-v2:latest
```

### Verifying PDP is Running

```bash
# Health check
curl http://localhost:7766/health

# Test authorization
curl -X POST http://localhost:7766/allowed \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "key": "test-user",
      "attributes": {
        "groups": ["system-owner"]
      }
    },
    "action": "create",
    "resource": {
      "type": "company"
    }
  }'
```

### Expected Response

```json
{
  "allow": true,
  "result": true
}
```

If you see `allow: true` but `result: false`, it means ABAC condition passed but no matching Set Rule was found.

## Troubleshooting

### Common Issues

#### 1. "Invalid API key" Error

**Symptom:** PDP logs show "Authentication failed: Invalid API key"

**Solution:** Ensure `PERMIT_API_KEY` environment variable is set correctly:

```bash
export PERMIT_API_KEY=permit_key_xxxxx
```

#### 2. Authorization Always Denied

**Symptom:** All requests return 403 Forbidden

**Checklist:**
1. Verify PDP is running: `curl http://localhost:7766/health`
2. Check if `permit.enabled=true` in configuration
3. Verify User Sets exist with correct conditions
4. Verify Set Rules grant permissions to User Sets
5. Check JWT contains `cognito:groups` claim

#### 3. ABAC `allow: true` but `result: false`

**Symptom:** PDP returns `{ "allow": true, "result": false }`

**Cause:** The ABAC condition (User Set) matched, but no Set Rule grants the permission.

**Solution:** Create a Set Rule linking the User Set to the resource action.

#### 4. Cognito Groups Not Recognized

**Symptom:** User has correct groups but authorization fails

**Debug Steps:**
1. Decode JWT and verify `cognito:groups` claim exists
2. Enable debug logging: `permit.debug-logging=true`
3. Check if groups are passed correctly in authorization request

```bash
# Decode JWT (base64)
echo "eyJhbG..." | cut -d'.' -f2 | base64 -d | jq .
```

### Debug Logging

Enable debug logging to see detailed authorization flow:

```properties
permit.debug-logging=true
logging.level.com.horaion.app.shared.security=DEBUG
```

## API Quick Reference

### Check Authorization Programmatically

```java
@Autowired
private PermitAuthorizationService permitService;

public void someMethod() {
    // Check and throw exception if denied
    permitService.checkOrThrow(PermitCheckRequest.of("create", "company"));

    // Check and get boolean result
    boolean allowed = permitService.isAllowed(PermitCheckRequest.of("read", "company"));

    // Check with resource ID
    permitService.checkOrThrow(PermitCheckRequest.of("update", "company", companyId.toString()));
}
```

### Constants Reference

Location: `src/main/java/com/horaion/app/shared/security/constants/PermitConstant.java`

```java
// User attribute keys
ATTR_USER_ROLE = "role"
ATTR_USER_ROLES = "roles"
ATTR_USER_GROUPS = "groups"
ATTR_USER_EMAIL = "email"
ATTR_USER_USERNAME = "username"

// Cognito group names
COGNITO_GROUP_SYSTEM_ADMINISTRATOR = "system-administrator"
COGNITO_GROUP_SYSTEM_OWNER = "system-owner"
COGNITO_GROUP_PRIVILEGED_SYSTEM_USER = "privileged-system-user"
COGNITO_GROUP_USER = "user"

// Default tenant
DEFAULT_TENANT = "default"
```

## Summary

1. **@PermitCheck** annotation on controller methods triggers authorization
2. **PermitCheckAspect** intercepts and builds authorization request
3. **PermitAuthorizationService** extracts Cognito groups from JWT and calls PDP
4. **PDP** evaluates User Set conditions against user attributes
5. **Set Rules** determine if matched User Sets have permission for the action

**No user sync is needed.** Authorization is purely based on attributes passed at check time.