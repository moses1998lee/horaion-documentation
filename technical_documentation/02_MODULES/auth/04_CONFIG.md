# Auth Module Configuration

## Application Properties
The module relies on the global `genesis` (soon to be `horaion`) configuration block.

| Property | Description | Required | Default |
| :--- | :--- | :--- | :--- |
| `aws.cognito.client-id` | App Client ID from AWS Console | Yes | - |
| `aws.cognito.client-secret` | App Client Secret | No* | - |
| `aws.cognito.user-pool-id` | internal User Pool ID | Yes | - |
| `aws.cognito.region` | AWS Region (e.g., `ap-southeast-1`) | Yes | `ap-southeast-1` |

> [!NOTE]
> * `client-secret` is optional in Cognito, but if enabled in AWS, it must be provided here. The `AuthService` automatically detects if it's set and calculates the `SECRET_HASH` accordingly.

## Beans
### `CognitoIdentityProviderClient`
Allows communication with AWS. Configured in `CognitoConfiguration` (Shared Infrastructure).

### `INotificationFacade`
An optional dependency (`Optional<INotificationFacade>`).
*   **Why Optional?**: Allows the Auth module to function (allow logins/registrations) even if the Notification system (SES/SNS) is down or not configured. It gracefully degrades by logging a warning instead of failing the transaction.
