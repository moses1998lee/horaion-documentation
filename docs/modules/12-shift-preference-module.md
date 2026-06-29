# Shift Preference Module

## Overview

The Shift Preference module lets employees submit the shifts they prefer to work on specific dates. Preferences are grouped by employee, date, and department so scheduling workflows can understand which shifts an employee wants for a given planning day.

**Location**: [`modules/shiftpreference/`](../../src/main/java/com/horaion/app/modules/shiftpreference/)

## Key Features

- **Self-Service Preferences**: Employees manage only their own preferred shifts
- **Date + Department Grouping**: One response group per employee, date, and department
- **Set Reconciliation**: Submitting a group adds new shifts and soft-deletes omitted shifts
- **Department Visibility**: Department leads, system owners, and administrators can read scoped team preferences
- **Shift Validation**: Submitted shifts must belong to the selected department
- **Soft Delete History**: Removed preferences remain available for audit/history

## Data Model

### ShiftPreference Entity

```java
@Entity
@Table(name = "employee_shift_preferences")
public class ShiftPreference {
    private UUID id;
    private Employee employee;
    private Shift shift;
    private LocalDate preferredDate;
    private Boolean softDelete;
    private Instant deletedAt;
}
```

The database stores one active row per `(employee_id, shift_id, preferred_date)`. The API groups those rows into one object per `(employee, date, department)`.

## API Endpoints

### Employee Preferences

**Base**: `/api/v1/employees/{employeeId}/shift-preferences`

| Method | Endpoint | Permission | Access |
|--------|----------|------------|--------|
| GET | `/` | read:shift_preference | Self |
| POST | `/` | create:shift_preference | Self |
| DELETE | `/{date}?departmentId={departmentId}` | delete:shift_preference | Self |

### Department Preferences

**Base**: `/api/v1/departments/{departmentId}/shift-preferences`

| Method | Endpoint | Permission | Access |
|--------|----------|------------|--------|
| GET | `/` | read:shift_preference | Department lead + Admin |

### Submit Preference Example

```http
POST /api/v1/employees/{employeeId}/shift-preferences
Authorization: Bearer {token}
Content-Type: application/json

{
  "date": "2026-07-06",
  "departmentId": "department-uuid",
  "shiftIds": [
    "morning-shift-uuid",
    "evening-shift-uuid"
  ]
}
```

**Response**:

```json
{
  "success": true,
  "data": {
    "employeeId": "employee-uuid",
    "employeeName": "Jane Tan",
    "date": "2026-07-06",
    "departmentId": "department-uuid",
    "departmentName": "Front Office",
    "shifts": [
      {
        "shiftId": "morning-shift-uuid",
        "shiftLabel": "Morning"
      }
    ]
  },
  "error": null
}
```

## Access Control Matrix

| Role | Manage Own | View Department | View All |
|------|------------|-----------------|----------|
| **system-administrator** | Yes | Yes | Yes |
| **system-owner** | Yes | Yes | Yes |
| **super-privileged-system-user** | Yes | Own lead departments | No |
| **privileged-system-user** | Yes | Own lead departments | No |
| **user** | Yes | No | No |

## Related Documentation

- [Shift Module](./06-shift-module.md) - Shift definitions that employees prefer
- [Employee Module](./05-employee-module.md) - Employee ownership and assignments
- [Department Module](./04-department-module.md) - Department scoping
- [Me Module](./11-me-module.md) - Self-scoped department assignment endpoint
