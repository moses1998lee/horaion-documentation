# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1`
This module splits endpoints into two controllers: one for the **Employee** (Applicant) and one for the **Manager** (Approver).
{% endhint %}

## Controller: `EmployeeLeaveAvailabilityController`

**Path**: `/employees/{employeeId}/leave-availability`

### 1. Get All Requests

**Endpoint**: `GET .../leave-availability`

Retrieves the history of requests for a specific employee.
*   **Query Parameters**: `range` (Start/End Date) is highly recommended for calendar views.

### 2. Create Request

**Endpoint**: `POST .../leave-availability`

Submits a new "Constraint" (Leave or Availability).

*   **Request Body**: `CreateEmployeeLeaveAvailabilityRequest`
    ```json
    {
      "requestType": "ANNUAL_LEAVE",
      "submissionType": "SELF_SERVICE",
      "startDate": "2024-12-25",
      "endDate": "2024-12-26",
      "notes": "Christmas Holiday"
    }
    ```
*   **Logic**:
    *   Checks for overlapping dates.
    *   If submitted by `SELF_SERVICE` -> Status: **PENDING**.
    *   If submitted by `ON_BEHALF` (e.g., HR) -> Status: **APPROVED**.

### 3. Update Request

**Endpoint**: `PUT .../{id}`

Modifies a **Pending** request.
*   **Restriction**: You generally cannot update a request once it has been `APPROVED` or `REJECTED`. You must cancel (Delete) it and submit a new one.

---

## Controller: `LeaveRequestController`

**Path**: `/leave-requests`

### 1. Approve Request

**Endpoint**: `POST .../{id}/approve`

Manager action to finalize a request.

{% hint style="success" %}
**Tip / Success:**
**Event Trigger**: Approving a request fires a `LeaveRequestApprovedEvent`. This allows downstream systems (like Notification Service) to instantly email the employee: "Your leave has been approved!".
{% endhint %}

### 2. Reject Request

**Endpoint**: `POST .../{id}/reject`

Manager action to deny a request.

*   **Request Body**:
    ```json
    {
      "rejectionReason": "Not enough coverage during Christmas peak."
    }
    ```

## Exception Handling

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `DuplicateLeaveRequestException` | `409 Conflict` | Dates overlap with an existing request. |
| `InvalidDateRangeException` | `400 Bad Request` | End Date is before Start Date. |
| `InvalidRequestStatusException` | `400 Bad Request` | Trying to modify a finalized request. |

### Example Error Response (Conflict)

When an employee tries to double-book themselves (e.g., requesting Sick Leave when they already have Annual Leave approved):

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "DUPLICATE_LEAVE_REQUEST",
    "message": "A leave request already exists for the overlapping period: 2024-12-25 to 2024-12-26."
  }
}
```
