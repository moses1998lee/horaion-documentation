# API Reference

{% hint style="warning" %}
**Security Notice**: Most endpoints in this controller are **Public** to allow unauthenticated users to Register and Login. However, rate limiting and internal checks are applied.
{% endhint %}

## Controller: `AuthController`

**Base Path**: `/api/v1/auth`

### 1. User Registration

**Endoint**: `POST /register`

Creates a new user account and initiates the verification process.

* **Security**: Restricted by `@PermitCheck` for specific roles, otherwise Public.
*   **Prerequisites**:

    * System Owners must provide a `companyId`.
    * Standard users must provide a `branchId`.

    **Example Request**:

    ```json
    {
      "email": "jane.doe@example.com",
      "password": "StrongPassword123!",
      "firstName": "Jane",
      "lastName": "Doe",
      "phoneNumber": "+15550109999",
      "branchId": "a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11",
      "userGroup": "user"
    }
    ```
*   **Responses**:

    * `201 Created`: Returns `RegisterResponse`.

    **Example Response**:

    ```json
    {
      "userSub": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "codeDeliveryDetails": "DESTINATION: j***@example.com, MEDIA: EMAIL",
      "userConfirmed": false
    }
    ```

#### 2. Confirm Account

* **POST** `/confirm`
* **Summary**: Finalizes registration using the OTP code sent to email.
* **Request Body**: `ConfirmAccountRequest`
  * `email`: User's email.
  * `confirmationCode`: The 6-digit code.
* **Responses**:
  * `200 OK`: Account confirmed.
  * `400 Bad Request`: Invalid/Expired code.

### 3. Login

**Endpoint**: `POST /login`

Authenticates a user against AWS Cognito and retrieves session tokens.

*   **Request Body**: `LoginRequest`

    ```json
    {
      "email": "jane.doe@example.com",
      "password": "StrongPassword123!"
    }
    ```
*   **Success Response** (`200 OK`):

    ```json
    {
      "sub": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "accessToken": "eyJraWQiOiJ...",
      "refreshToken": "eyJjdHkiOiJ...",
      "idToken": "eyJraWQiOiJ...",
      "expiresIn": 3600,
      "type": "Bearer"
    }
    ```
* **Error Responses**:
  * `401 Unauthorized`: Invalid credentials.
  * `403 Forbidden`: Account email is not confirmed.

> \[!CAUTION] **Token Security**: Tokens should be stored securely on the client side (e.g., `HttpOnly` cookies) to prevent XSS attacks.

#### 4. Refresh Token

* **POST** `/refresh`
* **Summary**: Obtains a fresh Access Token using a Refresh Token.
* **Request Body**: `RefreshTokenRequest`
  * `sub`: User ID.
  * `refreshToken`: The token received during login.
* **Responses**:
  * `200 OK`: Returns new `accessToken` and `idToken`.

#### 5. Resend Code

* **POST** `/resend-code`
* **Summary**: Re-triggers the verification email.
* **Request Body**: `ResendCodeRequest`
* **Logic**: If the email doesn't exist locally, it returns a generic specific success message to prevent user enumeration (Security Best Practice).

### Exception Handling

The `AuthExceptionHandler` provides a centralized map of exceptions to HTTP Problem Details.

| Exception                         | HTTP Status               | Description                               |
| --------------------------------- | ------------------------- | ----------------------------------------- |
| `MethodArgumentNotValidException` | `400 Bad Request`         | Validation failure (e.g., invalid email). |
| `InvalidParameterException`       | `400 Bad Request`         | AWS Cognito rejected parameters.          |
| `InvalidPasswordException`        | `400 Bad Request`         | Password complexity not met.              |
| `ExpiredCodeException`            | `400 Bad Request`         | Verification code expired.                |
| `CodeMismatchException`           | `400 Bad Request`         | Incorrect verification code.              |
| `UsernameExistsException`         | `409 Conflict`            | Email already registered.                 |
| `DataIntegrityViolationException` | `409 Conflict`            | Database constraint violation.            |
| `UserNotConfirmedException`       | `403 Forbidden`           | Login attempted for unverified account.   |
| `NotAuthorizedException`          | `401 Unauthorized`        | Invalid credentials (Login).              |
| `UserNotFoundException`           | `404 Not Found`           | Email does not exist.                     |
| `CodeDeliveryFailureException`    | `503 Service Unavailable` | AWS failed to send email/SMS.             |
| `TooManyRequestsException`        | `429 Too Many Requests`   | AWS rate limit exceeded.                  |
| `CryptographicException`          | `500 Internal Error`      | HMAC calculation failed.                  |
| `AuthConfigurationException`      | `500 Internal Error`      | Missing Client Secret/ID.                 |
