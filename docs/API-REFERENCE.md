# Horaion API Reference

## Overview

Complete API reference for the Horaion Workforce Management System. All endpoints require Bearer token authentication unless marked as "Public".

**Base URL**: `http://localhost:8080/api/v1` (development)

## Authentication

All authenticated endpoints require:
```http
Authorization: Bearer {accessToken}
```

Get access token via [`POST /auth/login`](#authentication-endpoints)

## Quick Navigation

- [Authentication](#authentication-endpoints) - Register, login, verify
- [Company Management](#company-management) - Organizations
- [Branch Management](#branch-management) - Locations
- [Department Management](#department-management) - Organizational units
- [Employee Management](#employee-management) - Workforce
- [Employee Bulk Import](#employee-bulk-import) - Excel import
- [Employee Roles](#employee-roles) - Job roles
- [Shift Management](#shift-management) - Shift patterns
- [Rule Management](#rule-management) - Business rules
- [Demand Forecast](#demand-forecast) - Workforce planning
- [Schedule Management](#schedule-management) - Schedule generation
- [Leave Management](#leave-management) - Leave requests
- [Current User (Me)](#current-user-me) - User profile
- [Health & Monitoring](#health--monitoring) - System status

---

## Authentication Endpoints

**Base**: `/api/v1/auth` | **All Public** (no authentication required)

### Register User
```http
POST /api/v1/auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecureP@ss1",
  "phoneNumber": "+12025551234",
  "firstName": "John",
  "lastName": "Doe",
  "companyId": "uuid",
  "branchId": "uuid",
  "userGroup": "user"
}
```
**Response**: `{ "userId": "cognito-sub" }`

**Permissions**: `create:account`

**See**: [Auth Module Documentation](./modules/01-auth-module.md)

### Confirm Account
```http
POST /api/v1/auth/confirm
{ "email": "user@example.com", "confirmationCode": "123456" }
```

### Login
```http
POST /api/v1/auth/login
{ "email": "user@example.com", "password": "SecureP@ss1" }
```
**Response**: `{ "sub", "accessToken", "refreshToken", "idToken", "expiresIn", "tokenType" }`

### Resend Verification Code
```http
POST /api/v1/auth/resend-code
{ "email": "user@example.com" }
```

### Refresh Access Token
```http
POST /api/v1/auth/refresh
{ "refreshToken": "..." }
```

---

## Company Management

**Base**: `/api/v1/companies`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/companies` | read:company | List all companies (paginated) |
| GET | `/companies/{id}` | read:company | Get company by ID |
| GET | `/companies/registration/{number}` | read:company | Get by registration number |
| GET | `/companies/search?name={term}` | read:company | Search by name |
| POST | `/companies` | create:company | Create new company |
| PUT | `/companies/{id}` | update:company | Update company |
| DELETE | `/companies/{id}` | delete:company | Delete company |
| POST | `/companies/{id}/complete-onboarding` | update:company | Mark onboarding complete |

**See**: [Company Module Documentation](./modules/02-company-module.md)

---

## Branch Management

**Base**: `/api/v1/companies/{companyId}/branches`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/branches` | read:branch | List branches by company |
| GET | `/branches/{id}` | read:branch | Get branch by ID |
| GET | `/branches/code/{code}` | read:branch | Get by branch code |
| GET | `/branches/search?name={term}` | read:branch | Search by name |
| POST | `/branches` | create:branch | Create new branch |
| PUT | `/branches/{id}` | update:branch | Update branch |
| DELETE | `/branches/{id}` | delete:branch | Delete branch (hard) |
| DELETE | `/branches/{id}/soft` | delete:branch | Soft delete branch |

**Branch Types**: HEADQUARTERS, REGIONAL, SATELLITE, WAREHOUSE, RETAIL, VIRTUAL

**Documentation**: See `docs_suggestion.md` segment on Branch module

---

## Department Management

**Base**: `/api/v1/companies/{companyId}/branches/{branchId}/departments`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/departments` | read:department | List departments by branch |
| GET | `/departments/{id}` | read:department | Get department by ID |
| GET | `/departments/code/{code}` | read:department | Get by department code |
| GET | `/departments/search?name={term}` | read:department | Search by name |
| GET | `/departments/{id}/roles` | read:employee-role | Get employee roles in department |
| POST | `/departments` | create:department | Create new department |
| PUT | `/departments/{id}` | update:department | Update department |
| PUT | `/departments/{id}/head` | update:department | Assign department head |
| DELETE | `/departments/{id}` | delete:department | Delete department |

**Documentation**: See `docs_suggestion.md` segments on Department module

---

## Employee Management

**Base**: `/api/v1`

### Employee CRUD

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/employees?page=0&size=10` | read:employee | List all employees (role-filtered) |
| GET | `/companies/{cId}/branches/{bId}/employees` | read:employee | List by branch |
| GET | `/companies/{cId}/branches/{bId}/employees/{id}` | read:employee | Get employee by ID |

**Access Control**:
- **system-owner**: All employees
- **privileged-system-user**: Department employees only
- **user**: No access (403)

### Department Assignments

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| POST | `/companies/{cId}/branches/{bId}/employees/{eId}/departments` | update:employee | Assign to department |
| GET | `/companies/{cId}/branches/{bId}/employees/{eId}/departments` | read:employee | Get assignments |
| DELETE | `/companies/{cId}/branches/{bId}/employees/{eId}/departments/{dId}` | update:employee | Remove from department |

**See**: [Employee Module Documentation](./modules/05-employee-module.md)

---

## Employee Bulk Import

**Base**: `/api/v1/employees`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| POST | `/employees/upload-excel` | create:employee | Upload Excel file for import |
| GET | `/employees/import/jobs/{jobId}` | read:employee | Get import job status |
| GET | `/employees/import/history` | read:employee | Get import history (paginated) |
| GET | `/employees/excel/download-template` | read:employee | Download Excel template |

**Excel Template Columns**:
- Required: FIRST_NAME, LAST_NAME, PHONE_NUMBER, EMAIL_ADDRESS, BRANCH, DEPARTMENT, ROLE, ROLE_PROFICIENCY, IS_PART_TIME, IS_HEAD_DEPARTMENT
- Optional: EMPLOYEE_CODE, HIRE_DATE, DATE_OF_BIRTH

**Features**:
- Auto-creates missing branches, departments, roles
- Creates Cognito users with temporary passwords
- Sends welcome emails
- Async processing with progress tracking

**See**: [Employee Module - Bulk Import](./modules/05-employee-module.md#bulk-import-feature)

---

## Employee Roles

**Base**: `/api/v1/companies/{companyId}`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/roles` | read:employee-role | Get all company roles (sorted by level) |
| GET | `/departments/{deptId}/roles` | read:employee-role | Get roles used in department |

**Role Levels**: 1-20 (lower = higher seniority)

**See**: [Employee Module - Roles](./modules/05-employee-module.md#employee-role-operations)

---

## Shift Management

**Base**: `/api/v1/companies/{companyId}/branches/{branchId}/departments/{deptId}/shifts`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/shifts` | read:shift | List shifts by department |
| GET | `/shifts/{id}` | read:shift | Get shift by ID |
| GET | `/shifts/types` | N/A | Get shift type options |
| GET | `/departments/{deptId}/shifts/roles` | read:shift | Get roles assigned to shifts |
| POST | `/shifts` | create:shift | Create new shift |
| PUT | `/shifts/{id}` | update:shift | Update shift |
| DELETE | `/shifts/{id}` | delete:shift | Delete shift |

**Shift Types**: MORNING, AFTERNOON, EVENING, NIGHT, SPLIT, FLEXIBLE, ON_CALL

**Documentation**: See `docs_suggestion.md` segments on Shift module

---

## Rule Management

**Base**: `/api/v1`

### Rules

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/departments/{deptId}/rules` | read:rule | List rules by department |
| GET | `/departments/{deptId}/rules/{id}` | read:rule | Get rule with fields |
| GET | `/branches/{branchId}/rules` | read:rule | List rules by branch |
| POST | `/departments/{deptId}/rules` | create:rule | Create rule |
| PUT | `/departments/{deptId}/rules/{id}` | update:rule | Update rule |
| DELETE | `/departments/{deptId}/rules/{id}` | delete:rule | Delete rule |

### Rule Fields

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| POST | `/rules/{ruleId}/fields` | create:rule | Add field to rule |
| POST | `/rules/{ruleId}/fields/from-template` | create:rule | Create field from template |
| PUT | `/rules/{ruleId}/fields/reorder` | update:rule | Reorder fields |

### Rule Answers

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| POST | `/rules/answers` | create:rule | Create rule answer |
| POST | `/rules/answers/bulk` | create:rule | Bulk create answers |
| GET | `/departments/{deptId}/rules/answers` | read:rule | Get answers by department |
| GET | `/rules/{ruleId}/answers` | read:rule | Get answers by rule |

### Rule Field Templates

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/rule-field-templates` | read:rule | List all templates |
| GET | `/rule-field-templates/key/{key}` | read:rule | Get by template key |
| GET | `/rule-field-templates/category/{cat}` | read:rule | Get by category |
| POST | `/rule-field-templates` | create:rule | Create template |

**See**: [Rule Module Documentation](../rule-module.md)

---

## Demand Forecast

**Base**: `/api/v1/companies/{companyId}/departments/{deptId}/demand-forecasts`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/demand-forecasts` | read:demand-forecast | List forecasts (paginated) |
| GET | `/demand-forecasts/{id}` | read:demand-forecast | Get forecast by ID |
| POST | `/demand-forecasts` | create:demand-forecast | Create forecast |
| PUT | `/demand-forecasts/{id}` | update:demand-forecast | Update forecast |
| DELETE | `/demand-forecasts/{id}` | delete:demand-forecast | Delete forecast |

**Documentation**: See `docs_suggestion.md` segments on Demand Forecast module

---

## Schedule Management

**Base**: `/api/v1/companies/{companyId}/departments/{deptId}/schedules`

### Schedule Operations

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| POST | `/schedules` | create:schedule | Create schedule (auto-generates) |
| GET | `/schedules` | read:schedule | List schedules (with filters) |
| GET | `/schedules/{id}` | read:schedule | Get schedule by ID |
| PUT | `/schedules/{id}` | update:schedule | Update schedule |
| POST | `/schedules/{id}/approve` | schedule:approve | Approve schedule |
| POST | `/schedules/{id}/reject` | schedule:reject | Reject schedule |
| PUT | `/schedules/{id}/status` | update:schedule | Update approval status |
| DELETE | `/schedules/{id}` | delete:schedule | Delete schedule |

**Status Values**: DRAFT, PROCESSING, GENERATED, APPROVED, REJECTED, FAILED, PUBLISHED

**See**: [Schedule Module Documentation](./modules/09-schedule-module.md)

---

## Leave Management

**Base**: `/api/v1`

### Employee Leave Requests

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/employees/{eId}/leave-availability` | read:leave-availability | Get employee's leave requests |
| GET | `/employees/{eId}/leave-availability/{id}` | read:leave-availability | Get specific request |
| POST | `/employees/{eId}/leave-availability` | create:leave-availability | Create leave request |
| PUT | `/employees/{eId}/leave-availability/{id}` | update:leave-availability | Update request |
| DELETE | `/employees/{eId}/leave-availability/{id}` | delete:leave-availability | Delete request |

### Department Leave Requests (HOD)

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/departments/{deptId}/leave-availability` | read:leave-availability | Get department leaves (paginated) |
| GET | `/departments/{deptId}/leave-availability/pending` | read:leave-availability | Get pending leaves |

### Admin Leave Requests (System Owners)

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/admin/leave-availability` | read:leave-availability | Get all leaves (paginated) |
| GET | `/admin/leave-availability/pending` | read:leave-availability | Get all pending leaves |

### Leave Approval

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| POST | `/leave-requests/{id}/approve` | approve:leave-availability | Approve leave request |
| POST | `/leave-requests/{id}/reject` | approve:leave-availability | Reject leave request |

**Documentation**: See `docs_suggestion.md` segments on Leave Availability module

---

## Current User (Me)

**Base**: `/api/v1/me`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/me` | Authenticated | Get current user profile |
| GET | `/me/departments` | read:department | Get departments where user is HOD |
| GET | `/me/notifications` | Authenticated | Get user notifications (paginated) |
| GET | `/me/notifications/unread` | Authenticated | Get unread notifications |
| GET | `/me/notifications/unread/count` | Authenticated | Get unread count |
| GET | `/me/notifications/{id}` | Authenticated | Get specific notification |
| PUT | `/me/notifications/{id}/read` | Authenticated | Mark as read |
| PUT | `/me/notifications/read-all` | Authenticated | Mark all as read |
| DELETE | `/me/notifications/{id}` | Authenticated | Delete notification |

**Documentation**: See `docs_suggestion.md` segments on Me module

---

## Health & Monitoring

**Public Endpoints** (no authentication required)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/actuator/health` | Application health status |
| GET | `/actuator/info` | Build and Git information |

---

## Response Formats

### Success Response
```json
{
  "success": true,
  "data": { /* response data */ },
  "error": null
}
```

### Paginated Response
```json
{
  "success": true,
  "data": {
    "content": [ /* array of items */ ],
    "pageable": {
      "pageNumber": 0,
      "pageSize": 10
    },
    "totalElements": 100,
    "totalPages": 10,
    "last": false,
    "first": true
  },
  "error": null
}
```

### Error Response (RFC 7807)
```json
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "Resource not found with id: xxx",
  "instance": "/api/v1/resource/xxx",
  "errors": [
    {
      "field": "fieldName",
      "message": "Field-specific error message"
    }
  ]
}
```

---

## Common HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET, PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error, invalid input |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Duplicate resource, constraint violation |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | External service unavailable |

---

## Common Query Parameters

### Pagination
- `page` - Page number (0-indexed, default: 0)
- `size` - Items per page (default: 10-20)
- `sort` - Sort field and direction (e.g., "name,asc", "createdAt,desc")

### Filtering
- `status` - Filter by status enum
- `name` - Search by name (partial match)
- `startDate` / `endDate` - Date range filters

---

## Authorization Model

### User Roles

| Role | Access Level |
|------|-------------|
| **system-owner** | Full access to all resources across all companies |
| **system-administrator** | Cross-company administrative access |
| **privileged-system-user** | Department-level management (HOD) |
| **user** | Personal data access only |

### Resource Permissions

Standard actions: `create`, `read`, `update`, `delete`, `approve`

Each endpoint is annotated with required permission:
```java
@PermitCheck(action = "create", resource = "company")
```

---

## Rate Limiting

- **Auth endpoints**: Subject to AWS Cognito limits
- **Bulk operations**: Limited to 100 items per request
- **General API**: No enforced limits (monitor usage)

---

## API Versioning

Current version: **v1**

All endpoints use `/api/v1` prefix. Future breaking changes will use `/api/v2`.

---

## Additional Resources

### Module Documentation
- **[Modules Overview](./modules/00-modules-overview.md)** - Architecture and patterns
- **[Auth Module](./modules/01-auth-module.md)** - Authentication details
- **[Company Module](./modules/02-company-module.md)** - Company management
- **[Employee Module](./modules/05-employee-module.md)** - Employee management and import
- **[Schedule Module](./modules/09-schedule-module.md)** - Scheduling and engine integration
- **[Rule Module](./rule-module.md)** - Business rules engine

### Technical Documentation
- **[Exception Handling](./02-design/exception-handling.md)** - Error handling architecture
- **[Security Design](./02-design/security.md)** - Security architecture
- **[Database Migrations](./04-operations/database-migrations.md)** - Schema evolution
- **[CI/CD Pipeline](./03-technical/ci-cd-pipeline.md)** - Deployment automation

### Operations
- **[Docker Deployment](./04-operations/deployment/docker.md)** - Container deployment
- **[Environment Variables](./04-operations/configuration/environment-variables.md)** - Configuration
- **[AWS Infrastructure](./04-operations/aws-infrastructure.md)** - AWS setup automation

### Development
- **[Development Workflow](./06-contributing/development-workflow.md)** - Development process
- **[Postman Collections](../postman/)** - API testing collections
- **[Setup Scripts](../scripts/)** - Automation scripts

---

## Support

For implementation details, consult the module-specific documentation. For questions about:
- **Authentication**: See [Auth Module](./modules/01-auth-module.md)
- **Bulk Import**: See [Employee Module - Import](./modules/05-employee-module.md#bulk-import-feature)
- **Scheduling**: See [Schedule Module](./modules/09-schedule-module.md)
- **Business Rules**: See [Rule Module](./rule-module.md)
- **Existing Documentation**: See [`docs_suggestion.md`](./docs_suggestion.md) for comprehensive segment-by-segment documentation
