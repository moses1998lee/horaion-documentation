# 01 - AWS Infrastructure

> **Cloud Provider Integrations**

The application leverages **Amazon Web Services (AWS)** for critical infrastructure components such as Identity Management and Email Delivery. The `aws` package contains the wrappers and clients used to communicate with these services.

---

## 1. Cognito (Identity Management)

**Configuration**: ``CognitoProperty.java``

We use **Amazon Cognito** as our Identity Provider (IdP). This handles User Sign-up, Sign-in, and Password recovery.

### Key Components

*   **User Pool**: Stores all user directories.
*   **App Client**: The credentials used by the Backend to talk to the User Pool.

### Property Mapping
`application.yaml` -> `CognitoProperty.java`

```yaml
aws:
  cognito:
    region: ap-southeast-1
    user-pool-id: ap-southeast-1_xxxxxx
    client-id: xxxxxxxxxx
```

{% hint style="info" %}
**Note**: The backend verifies JWT tokens issued by Cognito. It constructs the **Issuer URI** dynamically using the `region` and `userPoolId` (e.g., `https://cognito-idp.us-east-1.amazonaws.com/us-east-1_AbCdEf`).
{% endhint %}

---

## 2. SES (Simple Email Service)

**Package**: `messaging` (See [05_MESSAGING.md](05_MESSAGING.md))

While the client wrappers live here, the actual usage is abstracted via the [Messaging Service](05_MESSAGING.md).
