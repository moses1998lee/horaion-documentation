# Auth Module Domain Logic

## Service: `AuthService`
The core business logic resides here, orchestrating interactions between AWS Cognito and the local database.

### Core Flows

### 1. Registration (`register`)
**Pattern**: Distributed Transaction with Compensating Action
**Goal**: Sync identity (AWS) with business data (Local DB).

#### Process Flow
1.  **Identity Creation (AWS)**:
    *   The system invokes `Cognito.SignUp`.
    *   *Rationale*: Fail fast if the password is weak or email exists.
2.  **Identity Capture**:
    *   Success returns a `Subject ID` (`sub`). This is the immutable link between the two systems.
3.  **Local Persistence**:
    *   An `Employee` record is created keyed by the `sub`.
4.  **Rolllback Mechanism (Crucial)**:
    *   {% hint style="warning" %}
    *   **Important / Warning:**
    *   **Failure Scenario**: If the local database fails to save the Employee, the system enters an inconsistent state.
    *   {% endhint %}
    *   **Resolution**: The service catches the exception and immediately deletes the user from AWS ("Compensating Transaction") to restore consistency.
5.  **Notification**:
    *   Upon final success, a welcome email is queued.

### 2. Secret Hash Calculation
**Requirement**: AWS Cognito Security Compliance.

{% hint style="success" %}
**Tip / Success:**
AWS Cognito requires a `SECRET_HASH` to verify the client's authenticity. This prevents unauthorized apps from spamming the User Pool.
{% endhint %}

*   **Algorithm**: `HMAC-SHA256`
*   **Formula**: `Base64(HmacSHA256("Client Secret", "Username" + "Client ID"))`

### Validators
### Validators
Custom validators implemented:

#### `RegisterRequestValidator`
Enforces conditional requirements based on the user's role. `validators/RegisterRequestValidator.java`.

*   **Logic**:
    *   **System Owner (`SYSTEM_OWNER`)**:
        *   ✅ Must provide `companyId`.
        *   ❌ Cannot use `branchId`.
    *   **Regular User**:
        *   ✅ Must provide `branchId`.

#### `E164PhoneValidator`
Enforces E.164 compliance for AWS SNS compatibility.
*   **Example**: `+14155552671`
*   **Reference**: `ValidationConstant.E164_PATTERN`

## Utilities

### `ExceptionWrappingExecutor`
A functional wrapper helper (`utils/ExceptionWrappingExecutor.java`) designed to handle checked exceptions in lambda expressions, specifically for the `AuthService`'s functional calls.

**Why it exists**:
Java's Functional Interfaces (like `Supplier`) do not support checked exceptions. This utility wraps code blocks (like AWS SDK calls) to:

{% hint style="info" %}
**Note:**
**Wrapper Logic**: checked `Exception`s are wrapped into a runtime `CryptographicException` (usually for HMAC errors) or `AuthOperationException`. RuntimeExceptions (e.g., `SdkClientException`) pass through unmodified.
{% endhint %}

**Usage Pattern**:
```java
return executor.run(() -> {
    return cognitoClient.signUp(...);
});
```

## Constants
Defined in `AuthConstant.java`:
*   `MIN_JWT_PARTS = 2`: Structural check before parsing.
*   `ATTR_EMAIL`: Maps to Cognito attribute `email`.
*   `isValidBasicEmail()`: A custom "regex-free" validator to prevent ReDoS (Regular Expression Denial of Service) attacks.
