# Me Module

## Overview

The Me module provides current user profile information, notifications, and permissions. It serves as a self-service endpoint for authenticated users to access their own data without needing to know their employee ID.

**Location**: [`modules/me/`](../../src/main/java/com/horaion/app/modules/me/)

## Key Features

- **User Profile**: Get current employee profile with company/branch/department context
- **In-App Notifications**: User-specific notifications with read/unread tracking
- **Permissions**: Current user's resource permissions
- **Roles**: User's Cognito groups
- **Department Context**: Departments where user is Head of Department (HOD)

## Data Model

### MeResponse Structure

```java
public record MeResponse(
    UUID id,
    UUID companyId,
    UUID branchId,
    UUID departmentId,              // Primary department
    String branchName,
    String employeeCode,
    String firstName,
    String lastName,
    String fullName,
    String emailAddress,
    String phoneNumber,
    LocalDate dateOfBirth,
    LocalDate hireDate,
    EmploymentType employmentType,
    LocalDate probationEndDate,
    Availability availability,
    Map<String, Object> onPlannedLeave,
    Map<String, Object> additionalFields,
    String profilePictureUrl,
    Boolean isActive,
    List<String> roles,                     // Cognito groups
    Map<String, List<String>> permissions   // Resource → Actions
)
```

### UserNotification Entity

```java
@Entity
@Table(name = "user_notifications")
public class UserNotification {
    private UUID id;
    private UUID userId;                    // Employee ID
    private String title;
    private String message;
    private UserNotificationType type;
    private UUID referenceId;               // Related entity ID
    private String referenceType;           // Entity type
    private Boolean isRead;
    private Instant readAt;
    private String actionUrl;
    private Map<String, Object> metadata;   // JSONB
    private Boolean softDelete;
}
```

### Notification Types

```java
public enum UserNotificationType {
    LEAVE_APPROVAL_REQUEST,     // Leave pending approval
    LEAVE_APPROVED,             // Leave approved
    LEAVE_REJECTED,             // Leave rejected
    SYSTEM,                     // System notifications
    SCHEDULE_CHANGE,            // Schedule updated
    SHIFT_ASSIGNMENT            // Shift assigned
}
```

## API Endpoints

### Base URL
```
/api/v1/me
```

### Profile

#### Get Current User Profile

```http
GET /api/v1/me
Authorization: Bearer {token}
```

**Permission**: Authenticated (no additional permission check)

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "employee-uuid",
    "companyId": "company-uuid",
    "branchId": "branch-uuid",
    "departmentId": "department-uuid",
    "branchName": "Main Branch",
    "employeeCode": "EMP001",
    "firstName": "John",
    "lastName": "Doe",
    "fullName": "John Doe",
    "emailAddress": "john.doe@example.com",
    "phoneNumber": "+12025551234",
    "dateOfBirth": "1990-05-15",
    "hireDate": "2023-01-10",
    "employmentType": "FULL_TIME",
    "probationEndDate": null,
    "availability": "AVAILABLE",
    "onPlannedLeave": null,
    "additionalFields": {},
    "profilePictureUrl": null,
    "isActive": true,
    "roles": ["user"],
    "permissions": {
      "me": ["read"],
      "leave-availability": ["read", "create"],
      "schedule": ["read"]
    }
  },
  "error": null
}
```

#### Get My Departments (HOD Only)

```http
GET /api/v1/me/departments
Authorization: Bearer {token}
```

**Permission**: `read:department`

**Access**: Only for privileged-system-user, system-owner, or system-administrator

**Response**: List of departments where current user is head of department

### Notifications

#### Get All Notifications

```http
GET /api/v1/me/notifications?page=0&size=20
Authorization: Bearer {token}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": "notif-uuid",
        "title": "Leave Request Pending Approval",
        "message": "John Doe has requested leave from Jan 1 to Jan 5",
        "notificationType": "LEAVE_APPROVAL_REQUEST",
        "referenceId": "leave-request-uuid",
        "referenceType": "LEAVE_AVAILABILITY",
        "isRead": false,
        "readAt": null,
        "actionUrl": "/leave-requests/123",
        "metadata": {
          "employeeId": "uuid",
          "employeeName": "John Doe"
        },
        "createdAt": "2024-01-15T10:30:00Z"
      }
    ],
    "totalElements": 5
  },
  "error": null
}
```

#### Get Unread Notifications

```http
GET /api/v1/me/notifications/unread?page=0&size=20
Authorization: Bearer {token}
```

#### Get Unread Count

```http
GET /api/v1/me/notifications/unread/count
Authorization: Bearer {token}
```

**Response**:
```json
{
  "count": 5
}
```

#### Mark Notification as Read

```http
PUT /api/v1/me/notifications/{id}/read
Authorization: Bearer {token}
```

**Response**: 204 No Content

#### Mark All as Read

```http
PUT /api/v1/me/notifications/read-all
Authorization: Bearer {token}
```

**Response**: 204 No Content

#### Delete Notification

```http
DELETE /api/v1/me/notifications/{id}
Authorization: Bearer {token}
```

**Response**: 204 No Content (soft delete)

## Security Context

### Automatic User Resolution

The Me endpoints automatically resolve the current user from the JWT token:

```java
public MeResponse getCurrentEmployee() {
    // Extract cognito sub from JWT
    String cognitoSub = AuthorizationHelper.getRequiredUserSub();
    
    // Find employee by cognito sub
    Employee employee = employeeRepository.findByCognitoSub(cognitoSub)
        .orElseThrow(() -> new EmployeeNotFoundException(cognitoSub));
    
    // Extract roles from JWT
    List<String> roles = AuthorizationHelper.getCognitoGroups().orElse(List.of());
    
    // Get permissions from Permit.io
    Map<String, List<String>> permissions = permitService.getUserPermissions();
    
    return MeResponse.from(employee, roles, permissions);
}
```

## Use Cases

### Use Case 1: User Profile Dashboard

```bash
# Get current user profile
GET /api/v1/me
# Returns: employee data, roles, permissions

# Check unread notifications
GET /api/v1/me/notifications/unread/count
# Returns: { count: 5 }

# Get unread notifications
GET /api/v1/me/notifications/unread
# Returns: list of unread notifications
```

### Use Case 2: Department Head Dashboard

```bash
# Get departments I manage
GET /api/v1/me/departments
# Returns: list of departments where user is HOD

# Get pending leave requests for my departments
GET /api/v1/departments/{deptId}/leave-availability/pending
# Returns: pending leaves for approval
```

## Notification Integration

### Event-Driven Notifications

The Me module receives notifications from various system events:

```java
@EventListener
public void onLeaveRequestCreated(LeaveRequestCreatedEvent event) {
    // Get approval recipients
    List<NotificationRecipient> recipients = getApprovalRecipients(event);
    
    // Create in-app notifications
    for (NotificationRecipient recipient : recipients) {
        userNotificationService.create(new CreateUserNotificationRequest(
            recipient.employeeId(),
            "Leave Request Pending Approval",
            buildMessage(event),
            UserNotificationType.LEAVE_APPROVAL_REQUEST,
            event.leaveRequestId(),
            "LEAVE_AVAILABILITY"
        ));
    }
}
```

## Related Documentation

- [Employee Module](./05-employee-module.md) - Employee data structure
- [Leave Availability Module](./10-leave-availability-module.md) - Leave request details
- [Auth Module](./01-auth-module.md) - User authentication
- See [`docs_suggestion.md`](../docs_suggestion.md) for comprehensive details
