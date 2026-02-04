# Domain Model

## Entities

### 1. Shift
**Table**: `shifts`

The core aggregate root representing a shift template.

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `id` | UUID | Yes | Primary Key |
| `department_id` | UUID | Yes | Foreign Key to Department |
| `label` | VARCHAR(100) | Yes | Human-readable name (e.g., "Morning A") |
| `shift_type` | Enum | Yes | Classification (see below) |
| `start_time` | TIME | Yes | Start of the shift (e.g., `09:00:00`) |
| `end_time` | TIME | Yes | End of the shift (e.g., `17:00:00`) |
| `days_applied_to`| INT[] | Yes | Array of integers (1=Mon) indicating valid days |
| `color_code` | VARCHAR(10) | Yes | Hex code for UI display (e.g., `#FF0000`) |
| `is_active` | BOOLEAN | Yes | Default: `true`. |
| `soft_delete` | BOOLEAN | Yes | Default: `false`. Handling deletion logic. |

**Constraints**:
*   `uq_shift_label_per_department`: A shift label must be unique within a Department.

### 2. ShiftRole
**Table**: `shift_roles`

A association entity defining staffing requirements.

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `id` | UUID | Yes | Primary Key |
| `shift_id` | UUID | Yes | Parent Shift |
| `employee_role_id`| UUID | Yes | Reference to global Employee Role |
| `required_count` | INTEGER | Yes | How many people of this role are needed. (**Default: 1**) |

**Constraints**:
*   `uq_shift_role`: A specific Role can only be added once per Shift.
*   **Implementation Note**: While the database supports variable counts (e.g., 2x Baristas), the current `ShiftService` implementation hardcodes this to **1** upon creation.

## Enums

### ShiftType
Standardized categories for reporting and high-level logic.

```java
public enum ShiftType {
    MORNING,
    AFTERNOON,
    EVENING,
    NIGHT,
    SPLIT,    // A shift with a significant break in the middle
    FLEXIBLE, // No fixed start/end time
    ON_CALL   // Employee must be available but not necessarily on site
}
```

## Validation Rules

Requests are validated using `jakarta.validation` annotations.

{% hint style="danger" %}
**Critical:**
**Color Code Validation**: The system strictly enforces Hex format (`^#[0-9A-Fa-f]{6}$`). Invalid colors will cause a `400 Bad Request`.
{% endhint %}

| Field | Rule | Message |
| :--- | :--- | :--- |
| `label` | Not Blank, Max 100 | "Label must not exceed 100 characters." |
| `daysAppliedTo` | Not Empty, Size 1-7 | "Days applied to must have between 1 and 7 elements." |
| `colorCode` | Regex | "Color code must be a valid hex color." |
