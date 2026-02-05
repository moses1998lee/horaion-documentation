# 05 - Messaging Infrastructure

> **Low-Level Communication Adapters**

While the Notification System handles *orchestration*, the Messaging package handles the actual *transmission* of data.

---

## 1. Interfaces

### `IEmailService`
**File**: ``IEmailService.java``

Defines the contract for sending emails.
*   **Inputs**: `EmailRequest` (To, Subject, Body, Template Data).
*   **Output**: `CompletableFuture<MessageResponse>`.

### `INotificationService` (SNS)
**File**: ``INotificationService.java``

Defines the contract for publishing generic messages to a topic (used by the Facade).

---

## 2. Implementations
*(Located in `infrastructure.aws` or sub-modules)*

The actual implementation uses the **AWS SDK v2**.

{% hint style="info" %}
**Note:** By using interfaces, we can easily swap AWS SES for SendGrid or SMTP for local development without changing the core Notification logic.
{% endhint %}
