# Code Examples & Common Workflows

> **Horaion Workforce Management Platform - API Design**

Practical examples to help you understand how to work with the Horaion API.

---

## Table of Contents
- [Authentication Examples](#authentication-examples)
- [Employee Management](#employee-management)
- [Schedule Generation](#schedule-generation)
- [Error Handling](#error-handling)
- [Common Workflows](#common-workflows)

---

## 7.1 Authentication Examples {#id-7.1-authentication-examples}

### Register a New User

**What**: Create a new user account with email verification.

**Why**: Required before users can log in and access the system.

**How**:

```bash
# Step 1: Register user
curl -X POST http://localhost:8080/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "emailAddress": "john.doe@example.com",
    "password": "SecurePass123!",
    "firstName": "John",
    "lastName": "Doe",
    "phoneNumber": "+6591234567",
    "departmentId": 1,
    "employeeRoleId": 2
  }'

# Response:
{
  "success": true,
  "message": "User registered successfully. Please check your email for verification code.",
  "timestamp": "2026-02-02T13:00:00"
}
```

**What Happens**:
1. User is created in AWS Cognito (status: UNCONFIRMED)
2. Employee record is created in database
3. Verification email is sent to user's email address

---

### Confirm Email

**What**: Verify email address with the code sent via email.

**Why**: Required to activate the account before login.

**How**:

```bash
curl -X POST http://localhost:8080/auth/confirm \
  -H "Content-Type: application/json" \
  -d '{
    "emailAddress": "john.doe@example.com",
    "confirmationCode": "123456"
  }'

# Response:
{
  "success": true,
  "message": "Email confirmed successfully. You can now log in.",
  "timestamp": "2026-02-02T13:05:00"
}
```

---

### Login

**What**: Authenticate and receive JWT tokens.

**Why**: Tokens are required for all subsequent API calls.

**How**:

```bash
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "emailAddress": "john.doe@example.com",
    "password": "SecurePass123!"
  }'

# Response:
{
  "success": true,
  "data": {
    "accessToken": "eyJraWQiOiJ...",
    "idToken": "eyJraWQiOiJ...",
    "refreshToken": "eyJjdHkiOiJ...",
    "expiresIn": 3600,
    "tokenType": "Bearer"
  },
  "message": "Login successful",
  "timestamp": "2026-02-02T13:10:00"
}
```

**Important**: Save the `accessToken` - you'll need it for all API requests.

---

### Making Authenticated Requests

**What**: Use the access token to call protected endpoints.

**How**:

```bash
# Store token in variable
TOKEN="eyJraWQiOiJ..."

# Make authenticated request
curl -X GET http://localhost:8080/api/v1/employees \
  -H "Authorization: Bearer $TOKEN"
```

---

## 7.2 Employee Management {#id-7.2-employee-management}

### Create a Single Employee

**What**: Add a new employee to the system.

**Why**: Employees need to be in the system before they can be scheduled.

**How**:

```bash
curl -X POST http://localhost:8080/api/v1/employees \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "Jane",
    "lastName": "Smith",
    "emailAddress": "jane.smith@example.com",
    "phoneNumber": "+6598765432",
    "departmentId": 1,
    "employeeRoleId": 3,
    "employeeCode": "EMP001",
    "isActive": true,
    "additionalFields": {
      "skills": ["cashier", "inventory"],
      "preferredShift": "morning"
    }
  }'

# Response:
{
  "success": true,
  "data": {
    "id": 42,
    "firstName": "Jane",
    "lastName": "Smith",
    "emailAddress": "jane.smith@example.com",
    "employeeCode": "EMP001",
    "departmentId": 1,
    "employeeRoleId": 3,
    "isActive": true,
    "createdAt": "2026-02-02T13:15:00"
  },
  "message": "Employee created successfully",
  "timestamp": "2026-02-02T13:15:00"
}
```

**Note**: The employee is created but does NOT have a Cognito account yet. See next example.

---

### Create Cognito Account for Employee

**What**: Create an AWS Cognito account so the employee can log in.

**Why**: Separates employee data creation from authentication setup.

**How**:

```bash
curl -X POST http://localhost:8080/api/v1/employees/42/create-cognito \
  -H "Authorization: Bearer $TOKEN" \
  -d "webhookUrl=https://your-app.com/webhook"

# Response (Immediate):
{
  "success": true,
  "message": "Cognito account creation initiated. You will be notified via webhook.",
  "timestamp": "2026-02-02T13:20:00"
}

# Webhook Callback (Later):
POST https://your-app.com/webhook
{
  "employeeId": 42,
  "status": "SUCCESS",
  "cognitoSub": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "message": "Cognito account created successfully"
}
```

**What Happens**:
1. API returns immediately (202 Accepted)
2. Cognito account creation runs asynchronously
3. When complete, webhook is called with result
4. Employee record is updated with `cognito_sub`

---

### Bulk Import Employees from Excel

**What**: Upload an Excel file to create multiple employees at once.

**Why**: Faster than creating employees one by one.

**How**:

```bash
# Step 1: Download template
curl -X GET http://localhost:8080/api/v1/employees/template \
  -H "Authorization: Bearer $TOKEN" \
  -o employee_template.xlsx

# Step 2: Fill in the template with employee data

# Step 3: Upload the file
curl -X POST http://localhost:8080/api/v1/employees/upload \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@employees.xlsx" \
  -F "webhookUrl=https://your-app.com/webhook"

# Response:
{
  "success": true,
  "message": "Employee import initiated. Processing 50 employees.",
  "timestamp": "2026-02-02T13:25:00"
}
```

**What Happens**:
1. File is validated
2. Employees are created in database
3. Cognito accounts are created asynchronously
4. Webhook is called for each employee with status

---

### Get Employees with Pagination

**What**: Retrieve a list of employees with pagination and sorting.

**Why**: Prevents loading too much data at once.

**How**:

```bash
curl -X GET "http://localhost:8080/api/v1/employees?page=0&size=20&sortBy=lastName&sortDirection=ASC" \
  -H "Authorization: Bearer $TOKEN"

# Response:
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "firstName": "Alice",
        "lastName": "Anderson",
        "emailAddress": "alice@example.com",
        "employeeCode": "EMP001"
      },
      // ... 19 more employees
    ],
    "page": 0,
    "size": 20,
    "totalElements": 145,
    "totalPages": 8,
    "last": false
  },
  "message": "Employees retrieved successfully",
  "timestamp": "2026-02-02T13:30:00"
}
```

**Parameters**:
- `page`: Page number (0-indexed)
- `size`: Items per page
- `sortBy`: Field to sort by (firstName, lastName, emailAddress, etc.)
- `sortDirection`: ASC or DESC

---

## 7.3 Schedule Generation {#id-7.3-schedule-generation}

### Create and Generate a Schedule

**What**: Create a schedule request and trigger optimization.

**Why**: Generates optimal work schedules based on constraints.

**How**:

```bash
curl -X POST http://localhost:8080/api/v1/schedules \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "departmentId": 1,
    "scheduleName": "Week 5 Schedule",
    "startDate": "2026-02-03",
    "endDate": "2026-02-09",
    "webhookUrl": "https://your-app.com/webhook"
  }'

# Response (Immediate):
{
  "success": true,
  "data": {
    "id": 123,
    "scheduleName": "Week 5 Schedule",
    "departmentId": 1,
    "startDate": "2026-02-03",
    "endDate": "2026-02-09",
    "status": "PENDING"
  },
  "message": "Schedule created. Optimization in progress.",
  "timestamp": "2026-02-02T13:35:00"
}
```

**What Happens**:
1. Schedule record is created (status: PENDING)
2. System gathers constraints (employees, shifts, rules, demand forecasts)
3. Request is sent to external optimization engine (async)
4. Status changes to PROCESSING
5. When complete, status changes to COMPLETED
6. Webhook is called with result

---

### Check Schedule Status

**What**: Monitor the progress of schedule generation.

**How**:

```bash
curl -X GET http://localhost:8080/api/v1/schedules/123 \
  -H "Authorization: Bearer $TOKEN"

# Response (While Processing):
{
  "success": true,
  "data": {
    "id": 123,
    "scheduleName": "Week 5 Schedule",
    "status": "PROCESSING",
    "startDate": "2026-02-03",
    "endDate": "2026-02-09"
  },
  "timestamp": "2026-02-02T13:40:00"
}

# Response (When Complete):
{
  "success": true,
  "data": {
    "id": 123,
    "scheduleName": "Week 5 Schedule",
    "status": "COMPLETED",
    "startDate": "2026-02-03",
    "endDate": "2026-02-09",
    "scheduleData": {
      "assignments": [
        {
          "employeeId": 1,
          "date": "2026-02-03",
          "shiftId": 5,
          "timeBlocks": ["09:00-13:00", "14:00-18:00"]
        }
        // ... more assignments
      ]
    }
  },
  "timestamp": "2026-02-02T14:00:00"
}
```

**Status Values**:
- `PENDING`: Created, not yet sent to engine
- `PROCESSING`: Sent to engine, waiting for result
- `COMPLETED`: Engine returned result
- `FAILED`: Engine error or timeout
- `APPROVED`: Manager approved
- `REJECTED`: Manager rejected

---

### Approve a Schedule

**What**: Approve a completed schedule to make it active.

**Why**: Requires manager approval before schedules go live.

**How**:

```bash
curl -X POST http://localhost:8080/api/v1/schedules/123/approve \
  -H "Authorization: Bearer $TOKEN" \
  -d "approverId=5"

# Response:
{
  "success": true,
  "data": {
    "id": 123,
    "status": "APPROVED",
    "approvedBy": 5,
    "approvedAt": "2026-02-02T14:05:00"
  },
  "message": "Schedule approved successfully",
  "timestamp": "2026-02-02T14:05:00"
}
```

---

## 7.4 Error Handling {#id-7.4-error-handling}

### Understanding Error Responses

All errors follow RFC 7807 Problem Details format:

```json
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "Employee with ID 999 not found",
  "instance": "/api/v1/employees/999"
}
```

**Fields**:
- `type`: URI reference identifying the problem type
- `title`: Short, human-readable summary
- `status`: HTTP status code
- `detail`: Detailed explanation
- `instance`: URI of the specific occurrence

### Common Error Scenarios

#### Resource Not Found (404)

```bash
curl -X GET http://localhost:8080/api/v1/employees/999 \
  -H "Authorization: Bearer $TOKEN"

# Response:
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "Employee with ID 999 not found",
  "instance": "/api/v1/employees/999"
}
```

#### Validation Error (400)

```bash
curl -X POST http://localhost:8080/api/v1/employees \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "",
    "emailAddress": "invalid-email"
  }'

# Response:
{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "Validation failed: firstName must not be blank, emailAddress must be a valid email",
  "instance": "/api/v1/employees"
}
```

#### Unauthorized (401)

```bash
curl -X GET http://localhost:8080/api/v1/employees
# (No Authorization header)

# Response:
{
  "type": "about:blank",
  "title": "Unauthorized",
  "status": 401,
  "detail": "Full authentication is required to access this resource",
  "instance": "/api/v1/employees"
}
```

---

## 7.5 Common Workflows {#id-7.5-common-workflows}

### Workflow 1: Onboarding a New Employee

**Goal**: Add a new employee and set up their login.

**Steps**:

```bash
# Create employee record
curl -X POST http://localhost:8080/api/v1/employees \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "Bob",
    "lastName": "Johnson",
    "emailAddress": "bob@example.com",
    "phoneNumber": "+6591111111",
    "departmentId": 1,
    "employeeRoleId": 2,
    "employeeCode": "EMP042"
  }'
# Save the returned employee ID (e.g., 42)

# Create Cognito account
curl -X POST http://localhost:8080/api/v1/employees/42/create-cognito \
  -H "Authorization: Bearer $TOKEN"

# Employee receives welcome email with temporary password

# Employee logs in and changes password
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "emailAddress": "bob@example.com",
    "password": "TemporaryPass123!"
  }'

curl -X POST http://localhost:8080/auth/change-password \
  -H "Authorization: Bearer $EMPLOYEE_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "oldPassword": "TemporaryPass123!",
    "newPassword": "MyNewSecurePass456!"
  }'
```

---

### Workflow 2: Creating a Weekly Schedule

**Goal**: Generate an optimized schedule for the upcoming week.

**Steps**:

```bash
# Ensure all prerequisites are set up
# - Employees exist in the department
# - Shifts are defined
# - Rules are configured
# - Demand forecasts are created

# Create schedule request
curl -X POST http://localhost:8080/api/v1/schedules \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "departmentId": 1,
    "scheduleName": "Week 6 Schedule",
    "startDate": "2026-02-10",
    "endDate": "2026-02-16",
    "webhookUrl": "https://your-app.com/webhook"
  }'
# Save the schedule ID (e.g., 456)

# Monitor status (or wait for webhook)
curl -X GET http://localhost:8080/api/v1/schedules/456 \
  -H "Authorization: Bearer $TOKEN"

# Review the generated schedule
curl -X GET http://localhost:8080/api/v1/schedules/456/data \
  -H "Authorization: Bearer $TOKEN"

# Approve the schedule
curl -X POST http://localhost:8080/api/v1/schedules/456/approve \
  -H "Authorization: Bearer $TOKEN" \
  -d "approverId=5"
```

---

### Workflow 3: Bulk Employee Import

**Goal**: Import 100 employees from an Excel file.

**Steps**:

```bash
# Download template
curl -X GET http://localhost:8080/api/v1/employees/template \
  -H "Authorization: Bearer $TOKEN" \
  -o template.xlsx

# Fill in employee data in Excel
# Columns: firstName, lastName, emailAddress, phoneNumber, employeeCode, departmentId, employeeRoleId

# Upload file
curl -X POST http://localhost:8080/api/v1/employees/upload \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@employees_batch.xlsx" \
  -F "webhookUrl=https://your-app.com/webhook"

# Monitor webhook for completion
# Each employee creation will trigger a webhook callback

# Verify import
curl -X GET "http://localhost:8080/api/v1/employees?page=0&size=100" \
  -H "Authorization: Bearer $TOKEN"
```

---

## 7.6 Tips for Developers {#id-7.6-tips-for-developers}

### Always Check Response Status

```javascript
// Good practice
const response = await fetch('/api/v1/employees', {
  headers: { 'Authorization': `Bearer ${token}` }
});

if (!response.ok) {
  const error = await response.json();
  console.error('Error:', error.detail);
  // Handle error appropriately
}

const data = await response.json();
```

### Use Pagination for Large Datasets

```bash
# Don't fetch all employees at once
# Bad: GET /api/v1/employees (could return thousands)

# Good: Use pagination
GET /api/v1/employees?page=0&size=50
```

### Handle Async Operations

```javascript
// Schedule generation is async
const createResponse = await createSchedule(data);
const scheduleId = createResponse.data.id;

// Poll for completion or use webhook
const checkStatus = async () => {
  const status = await getSchedule(scheduleId);
  if (status.data.status === 'COMPLETED') {
    // Schedule is ready
  } else if (status.data.status === 'PROCESSING') {
    // Still processing, check again later
    setTimeout(checkStatus, 5000);
  }
};
```

### Store Tokens Securely

```javascript
// Good: Store in httpOnly cookie or secure storage
// Bad: Store in localStorage (vulnerable to XSS)

// Refresh tokens before expiry
if (tokenExpiresIn < 300) { // 5 minutes
  await refreshToken();
}
```

---

## Next Steps

- [Backend Strategy](04_BACKEND_SPECIFICATIONS.md) - Detailed module documentation
- [System Architecture](02_SYSTEM_ARCHITECTURE.md) - High-level design overview
