# 02 - AWS SES & SNS

> **Cloud Messaging Services**

---

## Simple Email Service (SES)

The `SesService` wraps the **AWS SDK 2.x Async Client** (`SesV2AsyncClient`) to send emails. It handles:
*   Standard Text/HTML emails.
*   **Templated Emails** (Stored in AWS).
*   Configuration Sets (for open/click tracking).

**Service**: `com.horaion.app.shared.infrastructure.aws.ses.services.SesService`

### Usage

```java
@Autowired
private IEmailService emailService;

public void notifyUser(String email) {
    EmailRequest request = new EmailRequest.Builder()
        .to(email)
        .templateName("WelcomeEmail")
        .templateData(Map.of("name", "John"))
        .build();

    // Returns a CompletableFuture
    emailService.sendTemplatedEmail(request)
        .thenAccept(response -> log.info("Sent: " + response.messageId()));
}
```

{% hint style="success" %}
**Best Practice:** Always use **Templated Emails**. Do not hardcode HTML strings in your Java code. Templates are managed via Terraform/CLI and ensure consistent branding across the platform.
{% endhint %}

---

## Simple Notification Service (SNS)

The `SnsService` provides a generic interface for publishing messages to SNS Topics. This is the backbone of our Event-Driven Architecture.

**Service**: `com.horaion.app.shared.infrastructure.aws.sns.services.SnsService`

### Features
*   **FIFO Support**: Can set `MessageGroupId` and `DeduplicationId` for strict ordering.
*   **Attributes**: Supports custom message attributes for filtering subscription policies.

### Automatic Configuration
The service is conditionally enabled via `spring.cloud.aws.sns.enabled=true`. In local development, you can disable this to avoid connecting to real AWS topics, or use LocalStack.

{% hint style="warning" %}
**Important:** When publishing domain events, ensure your `Topic ARN` is correctly configured in `application.yml`. A wrong ARN will cause `TopicNotFoundException`.
{% endhint %}
