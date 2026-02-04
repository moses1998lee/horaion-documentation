# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1/me`
**Scope**: All endpoints in this module are implicit to the **Currently Logged In User**. No IDs are required in the URL.
{% endhint %}

## Controller: `MeController`

### 1. Get My Profile

**Endpoint**: `GET .../me`

The "Boot Request". Called immediately after login to hydrate the application state.

*   **Response**: `MeResponse`
    ```json
    {
      "id": "uuid...",
      "fullName": "John Doe",
      "email": "john@example.com",
      "primaryDepartmentId": "uuid...",
      "role": {
        "name": "Manager",
        "precedence": 10
      },
      "permissions": {
        "schedule": ["read", "update"],
        "employee": ["read"]
      }
    }
    ```

### 2. Get My Leave Requests

**Endpoint**: `GET .../me/leave-requests`

Self-service history of time-off requests.

*   **Pagination**: Supported (`?page=0&size=10`).

### 3. Get Notifications

**Endpoint**: `GET .../me/notifications`

Retrieves the user's alert stream.

*   **Related Endpoints**:
    *   `GET .../notifications/unread/count`: For the "Red Badge" on the UI bell icon.
    *   `PUT .../notifications/{id}/read`: Mark specific item as read.
    *   `PUT .../notifications/read-all`: "Mark All Read".

{% hint style="warning" %}
**Important / Warning:**
**Filters**: The notification stream automatically filters based on the user's **Current Role**. If a notification was sent to "All Managers", and the user is downgraded to "Cashier", they will stop seeing it.
{% endhint %}

### Example Error Responses

#### 404 Not Found (Zombie Account)

This occurs when a user has a valid AWS Cognito Token (successfully validated by API Gateway), but their `Employee` record in the database is missing or disabled.

*   *Scenario*: Employee was fired (Soft Deleted in DB) but their Cognito account wasn't suspended yet.

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "EMPLOYEE_NOT_FOUND",
    "message": "No active employee profile found for current user subject: aws-sub-123"
  }
}
```
