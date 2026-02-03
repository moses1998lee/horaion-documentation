# Department Module

## Overview

The Department module manages organizational units within branches. Departments support hierarchical structures, budget tracking, and head of department assignments with automatic role management.

**Location**: [`modules/department/`](../../src/main/java/com/horaion/app/modules/department/)

## Key Features

- **Hierarchical Structure**: Parent-child department relationships
- **Budget Tracking**: Cost center codes and budget allocations
- **Head of Department**: Department leadership with automatic Cognito group assignment
- **Employee Assignments**: Many-to-many relationships with employees
- **Rule Scoping**: Business rules are defined at department level

## Data Model

### Department Entity

```java
@Entity
@Table(name = "departments")
public class Department {
    private UUID id;
    private Branch branch;
    private String name;
    private String code;                    // Unique within branch
    private String description;
    private Department parent;              // Self-referential hierarchy
    private Employee headOfDept;
    private String costCenterCode;
    private BigDecimal budget;
    private Boolean isActive;
    private Boolean softDelete;
}
```

## API Endpoints

**Base**: `/api/v1/companies/{companyId}/branches/{branchId}/departments`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/departments` | read:department | List departments by branch (paginated) |
| GET | `/departments/{id}` | read:department | Get department by ID |
| GET | `/departments/code/{code}` | read:department | Get by department code |
| GET | `/departments/{id}/roles` | read:employee-role | Get employee roles in department |
| GET | `/departments/search?name={term}` | read:department | Search by name |
| POST | `/departments` | create:department | Create new department |
| PUT | `/departments/{id}` | update:department | Update department |
| PUT | `/departments/{id}/head` | update:department | Assign department head |
| DELETE | `/departments/{id}` | delete:department | Delete department |

## Special Features

### Head of Department Assignment

When assigning a department head:
1. Employee becomes HOD
2. Automatically added to `PRIVILEGED_SYSTEM_USER` Cognito group
3. Can approve leave requests for department employees

### Hierarchical Queries

Get sub-departments:
```http
GET /api/v1/departments/parent/{parentId}
```

## Related Documentation

- [Branch Module](./03-branch-module.md) - Parent branch container
- [Employee Module](./05-employee-module.md) - Department employees
- [Rule Module](./07-rule-module.md) - Department-scoped rules
- [Schedule Module](./09-schedule-module.md) - Department schedules
- See [`docs_suggestion.md`](../docs_suggestion.md) for comprehensive details
