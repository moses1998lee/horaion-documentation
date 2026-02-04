# 03 - Tracking Service

> **Audit Trail and Status Management**

---

## Overview

The `NotificationTrackingService` provides the API for creating and updating the status of notifications. It is the interface between the application/workers and the `notification_records` table.

**Interface**: `INotificationTrackingService`

---

## Key Methods

### `create`
Must be called **before** emitting the event to SQS. This ensures we have a record in `PENDING` state. If the SQS message arrives but the record is missing (race condition), the worker will fail safely.

### `markAsSent`
Updates status to `SENT` and records the `providerMessageId`. This ID is critical for processing webhooks later (e.g., getting a "Delivered" webhook from AWS SES).

### `markAsFailed`
Records the error message and error code.
*   **Error Code**: Useful for automated retry logic (e.g., `THROTTLING` might trigger a retry, while `INVALID_EMAIL` should not).

---

## Webhook Support
The service also supports methods for processing asynchronous callbacks from providers:
*   `markAsDelivered`
*   `markAsBounced`
*   `markAsComplained`

{% hint style="danger" %}
**Critical:** `correlationId` must be unique for every notification request. Do not reuse IDs across different notifications.
{% endhint %}
