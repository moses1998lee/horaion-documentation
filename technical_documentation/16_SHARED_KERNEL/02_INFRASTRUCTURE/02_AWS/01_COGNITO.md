# 01 - AWS Cognito

> **User Identity and Group Management**

---

## Overview

The `CognitoGroupService` provides a wrapper around the **Amazon Cognito Identity Provider Client**. It allows the application to perform administrative actions that standard users cannot do themselves, such as managing group memberships.

**Service**: `com.horaion.app.shared.infrastructure.aws.cognito.CognitoGroupService`

---

## Capabilities

| Action | Description | AWS Permission Required |
| :--- | :--- | :--- |
| **Add User to Group** | Assigns a user to a group (e.g., `SYSTEM_ADMINISTRATOR`). | `cognito-idp:AdminAddUserToGroup` |
| **Remove User from Group** | Revokes group membership. | `cognito-idp:AdminRemoveUserFromGroup` |
| **List User Groups** | Returns all groups a user belongs to (with precedence). | `cognito-idp:AdminListGroupsForUser` |
| **Create Group** | Creates a new group in the User Pool. | `cognito-idp:CreateGroup` |

---

## Privacy & Logging

The service automatically masks sensitive information in logs.

*   **Input**: `john.doe@example.com`
*   **Log Output**: `j***@example.com`

{% hint style="success" %}
**Tip:** This service uses the `CognitoProperty` bean to resolve the correct User Pool ID for the current environment (Staging vs. Production).
{% endhint %}

{% hint style="danger" %}
**Critical:** These operations require AWS credentials with high privilege. They are typically used only by the `User Management` module or during system initialization.
{% endhint %}
