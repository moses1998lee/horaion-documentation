# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1/companies/{cid}/departments/{did}/schedules`
**Async Pattern**: This module relies heavily on HTTP `202 Accepted` for generation endpoints.
{% endhint %}

## Controller: `ScheduleController`

### 1. Create & Generate

**Endpoint**: `POST .../schedules`

Creates the schedule container and immediately triggers the generation job.

*   **Request Body**:
    ```json
    {
      "startDate": "2024-12-25",
      "endDate": "2024-12-31",
      "startTime": "08:00",
      "endTime": "22:00",
      "shiftIds": ["..."], // Optimization targets
      "blockSizeMinutes": 15 // Granularity
    }
    ```
*   **Response**: `200 OK` (Job Submitted)
    ```json
    {
      "jobId": "abc-123",
      "status": "PROCESSING",
      "message": "Optimization started."
    }
    ```

### 2. Get Schedule Details

**Endpoint**: `GET .../schedules/{id}`

Poll this endpoint to check if `jobStatus` has changed from `PROCESSING` to `COMPLETED`.

#### Polling Response Example (`PROCESSING`)

```json
{
  "id": "abc-123",
  "jobStatus": "PROCESSING",
  "approvalStatus": "PENDING",
  "totalTimeBlocks": 672,
  "progress": 0.45,  // Hypothetical progress indicator
  "shifts": []       // Empty until completed
}
```

### 3. Approve Schedule

**Endpoint**: `POST .../schedules/{id}/approve`

Finalizes the schedule. This makes the shifts visible to employees in their mobile apps.

*   **Logic**:
    *   Validates that `jobStatus` is `COMPLETED`.
    *   Sets `approvalStatus` to `APPROVED`.
    *   Locks the schedule from further re-generation.

---

## Controller: `ScheduleCallbackController`

**Path**: `/api/v1/schedules/{id}/callback`

{% hint style="warning" %}
**Important / Warning:**
**Public Endpoint**: This endpoint is technically public (no JWT required) to allow the external Engine to call it.
**Security**: It relies on the obscure/unguessable `scheduleId` UUID and potentially IP allow-listing at the WAF level.
{% endhint %}

### 1. Receive Results

**Endpoint**: `POST .../callback`

Receives the optimized shift assignments.

*   **Payload**: Large JSON containing `[employee_id, shift_id, start_time, end_time]`.
*   **Action**:
    1.  Parses the JSON.
    2.  Writes thousands of `ScheduleShift` records to the DB.
    3.  Updates `jobStatus` to `COMPLETED`.

## Exception Handling

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `InvalidScheduleStatusException` | `400 Bad Request` | Trying to regenerate an `APPROVED` schedule. |
| `ScheduleNotFoundException` | `404 Not Found` | Invalid ID. |
| `EngineApiException` | `502 Bad Gateway` | The external optimization engine is down or rejected the payload. |

### Example Error (Engine Failure)

```json
{
  "success": false,
  "error": {
    "code": "ENGINE_CONNECTION_ERROR",
    "message": "Failed to submit job to optimization engine. Please try again later."
  }
}
```
