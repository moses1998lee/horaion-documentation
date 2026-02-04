# 02 - Notification Workers

> **Background Consumers**

---

## Overview

Workers are `@Component` classes annotated with `@SqsListener`. They sit in the background, polling AWS SQS queues for `NotificationEvent` messages.

**Package**: `com.horaion.app.shared.infrastructure.notification.workers`

---

## EmailNotificationWorker
Listens to `${notification.queues.email}`.

### Workflow
1.  **Receive**: deserializes the event.
2.  **Validate**: Checks if type is `EMAIL`.
3.  **Send**: Calls `IEmailService.sendEmail()` (Async).
4.  **Update**:
    *   **Success**: Calls `trackingService.markAsSent()` with the SES Message ID.
    *   **Failure**: Calls `trackingService.markAsFailed()` with the error code.
5.  **Acknowledge**: Manually acknowledges the SQS message to remove it from the queue.

{% hint style="info" %}
**Manual Acknowledgement:** We use `SqsListenerAcknowledgementMode.MANUAL` to ensure we don't delete the message from the queue until we are sure we have attempted delivery or recorded a failure.
{% endhint %}

---

## SmsNotificationWorker
Listens to `${notification.queues.sms}`.

{% hint style="warning" %}
**Implementation Gap:** Currently, the SMS worker is a **placeholder**. It processes the event but immediately logs a warning and marks the notification as `FAILED` with error code `SMS_NOT_IMPLEMENTED`. Integration with Twilio or AWS SNS SMS is pending.
{% endhint %}
