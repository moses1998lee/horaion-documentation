# 07 - Infrastructure Properties

> **Centralized Configuration**

**Package**: `com.horaion.app.shared.infrastructure.properties`

This package holds the `@ConfigurationProperties` classes that map `application.yaml` to Java Beans for all external adapters.

---

## Classes

| Class | Prefix | Description |
| :--- | :--- | :--- |
| [`CognitoProperty`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/infrastructure/properties/CognitoProperty.java) | `aws.cognito` | Region, UserPoolId, ClientId. |
| [`NotificationProperty`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/infrastructure/properties/NotificationProperty.java) | `notification` | Enabled toggle, Queue URLs. |
| [`GenesisProperty`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/infrastructure/properties/GenesisProperty.java) | `genesis` | URLs for internal services. |
| [`SesProperty`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/infrastructure/properties/SesProperty.java) | `aws.ses` | Source email address ("From: noreply@horaion.com"). |

{% hint style="info" %}
**Note:** All these properties use Spring's Validation API (e.g., `@NotNull`) to ensure the application fails to start if critical config is missing, rather than failing at runtime.
{% endhint %}
