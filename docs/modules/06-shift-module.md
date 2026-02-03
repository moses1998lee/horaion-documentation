# Shift Module

## Overview

The Shift module manages shift patterns and scheduling requirements. It defines shift types, time blocks, role requirements, and days of operation for workforce scheduling.

**Location**: [`modules/shift/`](../../src/main/java/com/horaion/app/modules/shift/)

## Key Features

- **Multiple Shift Types**: MORNING, AFTERNOON, EVENING, NIGHT, SPLIT, FLEXIBLE, ON_CALL
- **Role Requirements**: Associate employee roles with shifts
- **Day Patterns**: Specify which days shifts apply to (1=Monday, 7=Sunday)
- **Time Blocks**: Define start/end times for shift coverage
- **Color Coding**: Visual identification with hex color codes

## Data Model

### Shift Entity

```java
@Entity
@Table(name = "shifts")
public class Shift {
    private UUID id;
    private Department department;
    private String label;
    private ShiftType type;
    private LocalTime startTime;
    private LocalTime endTime;
    private List<Integer> daysAppliedTo;    // 1-7 (Monday-Sunday)
    private String colorCode;               // Hex color (#4CAF50)
    private List<ShiftRole> shiftRoles;     // Associated employee roles
    private Boolean isActive;
}
```

### Shift Types

```java
public enum ShiftType {
    MORNING,    // Morning shift (typically 08:00-16:00)
    AFTERNOON,  // Afternoon shift (typically 12:00-20:00)
    EVENING,    // Evening shift (typically 16:00-00:00)
    NIGHT,      // Night shift (typically 00:00-08:00)
    SPLIT,      // Split shift (multiple periods)
    FLEXIBLE,   // Flexible shift (variable hours)
    ON_CALL     // On-call shift (as needed)
}
```

## API Endpoints

**Base**: `/api/v1/companies/{companyId}/branches/{branchId}/departments/{deptId}/shifts`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/shifts` | read:shift | List shifts by department |
| GET | `/shifts/{id}` | read:shift | Get shift by ID |
| GET | `/shifts/types` | N/A | Get shift type options |
| GET | `/departments/{deptId}/shifts/roles` | read:shift | Get roles in shifts |
| POST | `/shifts` | create:shift | Create shift |
| PUT | `/shifts/{id}` | update:shift | Update shift |
| DELETE | `/shifts/{id}` | delete:shift | Delete shift |

### Get Shift Type Options

```http
GET /api/v1/shifts/types
```

**Response**:
```json
[
  { "value": "MORNING", "label": "Morning" },
  { "value": "AFTERNOON", "label": "Afternoon" },
  { "value": "EVENING", "label": "Evening" },
  { "value": "NIGHT", "label": "Night" },
  { "value": "SPLIT", "label": "Split" },
  { "value": "FLEXIBLE", "label": "Flexible" },
  { "value": "ON_CALL", "label": "On call" }
]
```

### Create Shift Example

```http
POST /api/v1/companies/{companyId}/branches/{branchId}/departments/{deptId}/shifts
Authorization: Bearer {token}
Content-Type: application/json

{
  "label": "Morning Shift",
  "type": "MORNING",
  "startTime": "08:00:00",
  "endTime": "16:00:00",
  "daysAppliedTo": [1, 2, 3, 4, 5],
  "colorCode": "#4CAF50",
  "shiftRoles": [
    "role-uuid-1",
    "role-uuid-2"
  ]
}
```

## Related Documentation

- [Department Module](./04-department-module.md) - Shift container
- [Schedule Module](./09-schedule-module.md) - Uses shifts for scheduling
- [Employee Module](./05-employee-module.md) - Role assignments for shifts
- See [`docs_suggestion.md`](../docs_suggestion.md) for comprehensive details
