# 07 - Infrastructure Properties

> **Centralized Configuration**

**Package**: `com.horaion.app.shared.infrastructure.properties`

This package holds the `@ConfigurationProperties` classes that map `application.yaml` to Java Beans for all external adapters.

---

## Classes

| Class | Prefix | Description |
| :--- | :--- | :--- |
| ``CognitoProperty`` | `aws.cognito` | Region, UserPoolId, ClientId. |
| ``NotificationProperty`` | `notification` | Enabled toggle, Queue URLs. |
| ``GenesisProperty`` | `genesis` | URLs for internal services. |
| ``SesProperty`` | `aws.ses` | Source email address ("From: noreply@horaion.com"). |

{% hint style="info" %}
**Note:** All these properties use Spring's Validation API (e.g., `@NotNull`) to ensure the application fails to start if critical config is missing, rather than failing at runtime.
{% endhint %}
