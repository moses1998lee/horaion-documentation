# Auth Module Configuration

## Application Properties
The module relies on the global `horaion` configuration block.

| Property | Description | Required | Default |
| :--- | :--- | :--- | :--- |
| `aws.cognito.client-id` | App Client ID from AWS Console | Yes | - |
| `aws.cognito.client-secret` | App Client Secret | No* | - |
| `aws.cognito.user-pool-id` | internal User Pool ID | Yes | - |
| `aws.cognito.region` | AWS Region (e.g., `ap-southeast-1`) | Yes | `ap-southeast-1` |

{% hint style="info" %}
**Note:**
* `client-secret` is optional in Cognito, but if enabled in AWS, it must be provided here. The `AuthService` automatically detects if it's set and calculates the `SECRET_HASH` accordingly.
{% endhint %}

## Beans
### `CognitoIdentityProviderClient`
Allows communication with AWS. Configured in `CognitoConfiguration` (Shared Infrastructure).

### `INotificationFacade`
An optional dependency (`Optional<INotificationFacade>`).
*   **Why Optional?**: Allows the Auth module to function (allow logins/registrations) even if the Notification system (SES/SNS) is down or not configured. It gracefully degrades by logging a warning instead of failing the transaction.

{% hint style="success" %}
**Success:** Graceful degradation of the `NotificationFacade` ensures that critical authentication flows remain functional even during downstream service interruptions, prioritizing user access over non-critical communications.
{% endhint %}
