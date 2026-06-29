# Shift Preference Configuration

## Database Configuration

The module is introduced by Flyway migration:

```text
V70__create_employee_shift_preferences.sql
```

### Table

**Table**: `employee_shift_preferences`

| Column | Notes |
| :--- | :--- |
| `employee_id` | Foreign key to `employees`. |
| `shift_id` | Foreign key to `shifts`. |
| `preferred_date` | Plain `DATE` column for efficient schedule-window filtering. |
| `soft_delete` | Active rows use `false`; removed rows use `true`. |
| `deleted_at` | Set when rows are soft-deleted. |

### Indexes

| Index Name | Columns | Purpose |
| :--- | :--- | :--- |
| `uq_employee_shift_preference_emp_shift_date` | `employee_id`, `shift_id`, `preferred_date` where `soft_delete = false` | Ensures at most one active row per employee, shift, and date while allowing re-add after soft delete. |
| `idx_employee_shift_preferences_employee_id` | `employee_id` where `soft_delete = false` | Self-service reads by employee. |
| `idx_employee_shift_preferences_shift_id` | `shift_id` where `soft_delete = false` | Shift joins and maintenance lookups. |
| `idx_employee_shift_preferences_preferred_date` | `preferred_date` where `soft_delete = false` | Planning window queries. |

{% hint style="warning" %}
**Important:**
The database stores one row per `(employee, shift, date)`. The application-level reconciliation groups rows by `(employee, date, department)`. Do not change the unique index to include department; department is derived through `shift.department_id`.
{% endhint %}

## Permit.io Resource

Register the following resource in Permit.io before enabling the endpoints in a strict PDP environment:

| Resource Key | Actions |
| :--- | :--- |
| `shift_preference` | `read`, `create`, `delete` |

Controller annotations:

| Endpoint Group | Annotation |
| :--- | :--- |
| Employee reads | `@PermitCheck(action = "read", resource = "shift_preference")` |
| Employee submit | `@PermitCheck(action = "create", resource = "shift_preference")` |
| Employee delete | `@PermitCheck(action = "delete", resource = "shift_preference")` |
| Department reads | `@PermitCheck(action = "read", resource = "shift_preference")` |

Service-level checks must remain in place because Permit.io validates the high-level action while the service enforces row ownership and department scope.

## Security Context Support

The implementation depends on reusable helpers in `SecurityContextService`:

| Helper | Purpose |
| :--- | :--- |
| `assertOwnEmployeeAccess(...)` | Ensures the path employee matches the authenticated employee for self-service endpoints. |
| `assertDepartmentAccess(...)` | Ensures department reads are limited to system administrators, system owners, or authorized department leads. |
| `validateCompanyAccessUnlessAdmin(...)` | Ensures submitted shifts belong to a company the caller can access unless the caller is an administrator. |

## Application Properties

No module-specific application properties are required.

Required platform configuration remains the shared security stack:

*   AWS Cognito authentication must populate the current employee context.
*   Permit.io PDP must know the `shift_preference` resource.
*   Department assignments and department head relationships must be current for department-level reads.

## Test Coverage

Backend coverage was added in:

```text
src/test/java/com/horaion/app/modules/shiftpreference/services/ShiftPreferenceServiceTest.java
src/test/java/com/horaion/app/modules/me/controllers/MeControllerTest.java
```

Targeted validation command used for the feature branch:

```bash
./mvnw -Dtest=ShiftPreferenceServiceTest,MeControllerTest test
```

The service tests cover preference reconciliation, grouping, delete behavior, shift/department mismatch validation, and access rules.
