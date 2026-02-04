# 01 - Notification Overview

> **Reliable, Asynchronous Messaging Engine**

---

## Architecture

The Notification Engine is designed to decouple business logic (e.g., "Shift Created") from message delivery (e.g., "Send Email"). It uses an **Event-Driven Architecture** backed by AWS SQS and a persistent Outbox pattern.

```mermaid
graph LR
    App[Application] -->|1. Emit Event| Queue[AWS SQS]
    App -->|2. Create Record| DB[(Notification DB)]
    Queue -->|3. Consume| Worker[Notification Worker]
    Worker -->|4. Send| AWS[AWS SES/SNS]
    AWS -->|5. Ack| Worker
    Worker -->|6. Update Status| DB
```

---

## Key Components

| Component | Responsibility |
| :--- | :--- |
| **SQS Queues** | Buffers high-volume events (`notification.queues.email`, `notification.queues.sms`). |
| **Workers** | Background listeners (@SqsListener) that process messages. |
| **Tracking Service** | Manage the lifecycle of a notification in the database. |

---

## Data Model
The `NotificationRecord` entity stores the full history of every message sent by the system.

*   **Correlation ID**: Links the notification to the original domain event.
*   **Provider Message ID**: The ID returned by AWS (e.g., SES Message ID), used for tracking delivery receipts/bounces.
*   **Status Lifecycle**: `PENDING` -> `SENT` -> `DELIVERED` (or `FAILED`/`BOUNCED`).

{% hint style="success" %}
**Reliability:** SQS ensures that if the worker fails (e.g., network error), the message remains in the queue and is retried automatically.
{% endhint %}
