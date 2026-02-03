# Frequently Asked Questions (FAQ)

Common questions and answers about the Genesis platform.

---

## General Questions

### What is Genesis?
Genesis is an enterprise workforce management platform that helps organizations manage employees, create optimized work schedules, and forecast staffing demand. It's built with Spring Boot and follows modern architectural patterns.

### Who should use this documentation?
- **New Developers**: Start with [README.md](README.md) and [EXAMPLES.md](EXAMPLES.md)
- **Architects**: Focus on [ARCHITECTURE.md](ARCHITECTURE.md)
- **DevOps**: See [OPERATIONS.md](OPERATIONS.md)
- **API Consumers**: Reference [TECHNICAL.md](TECHNICAL.md) and [MODULES.md](MODULES.md)

### What technologies do I need to know?
- **Java 21**: Modern Java with records, pattern matching
- **Spring Boot 3.4**: Dependency injection, REST controllers, JPA
- **PostgreSQL**: Relational database, SQL
- **AWS Services**: Cognito (auth), SES (email), SNS/SQS (messaging)
- **REST APIs**: HTTP methods, JSON, authentication

---

## Architecture Questions

### Why Vertical Slice Architecture instead of traditional layers?
**Traditional Layered Architecture** organizes code by technical concerns (all controllers together, all services together, etc.). This creates:
- ❌ Tight coupling across layers
- ❌ Difficult to find all code for a feature
- ❌ Changes ripple across many files

**Vertical Slice Architecture** organizes code by business features. Each module contains everything needed for that feature:
- ✅ All related code in one place
- ✅ Easy to understand complete features
- ✅ Modules can evolve independently

**Example**: The Employee module contains `EmployeeController`, `EmployeeService`, `EmployeeRepository`, `Employee` entity, and DTOs all in `modules/employee/`.

### What is the difference between a Module and a Package?
- **Package**: A Java namespace (e.g., `com.resetrix.genesis.modules.employee`)
- **Module**: A business feature containing multiple packages (controller, service, repository, entity, dto)

Think of a module as a self-contained feature slice.

### How does authentication work?
1. User registers via `/auth/register` → Creates Cognito user + Employee record
2. User confirms email via `/auth/confirm` → Activates account
3. User logs in via `/auth/login` → Receives JWT tokens
4. User includes token in requests: `Authorization: Bearer <token>`
5. API validates JWT signature and extracts user context

See [ARCHITECTURE.md - Authentication Flow](ARCHITECTURE.md#authentication-flow-aws-cognito--jwt) for detailed diagram.

---

## Development Questions

### How do I add a new API endpoint?
1. Create a method in the appropriate controller
2. Add business logic in the service layer
3. Use the repository to access data
4. Return a DTO wrapped in `ApiResponseDTO`

**Example**:
```java
@RestController
@RequestMapping("/api/v1/employees")
public class EmployeeController {
    
    @GetMapping("/{id}")
    public ResponseEntity<ApiResponseDTO<EmployeeResponseDTO>> getEmployee(@PathVariable Long id) {
        EmployeeResponseDTO employee = employeeService.getEmployeeById(id);
        return ResponseEntity.ok(new ApiResponseDTO<>(true, employee, "Employee retrieved successfully"));
    }
}
```

### How do I add a new database table?
1. Create a Flyway migration file: `V{next_version}__{description}.sql`
2. Write the SQL to create the table
3. Create a JPA entity class
4. Create a repository interface extending `JpaRepository`
5. Run `./mvnw flyway:migrate`

**Example Migration** (`V12__add_notifications_table.sql`):
```sql
CREATE TABLE notifications (
    id BIGSERIAL PRIMARY KEY,
    employee_id BIGINT NOT NULL REFERENCES employees(id),
    message TEXT NOT NULL,
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### How do I handle errors properly?
Throw domain-specific exceptions and let `GlobalExceptionHandler` convert them to RFC 7807 Problem Details.

**Example**:
```java
// In service layer
if (employee == null) {
    throw new ResourceNotFoundException("Employee with ID " + id + " not found");
}

// GlobalExceptionHandler automatically converts to:
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "Employee with ID 999 not found",
  "instance": "/api/v1/employees/999"
}
```

### How does logging work?
The `LogAspect` automatically logs all service and controller methods using AOP. You don't need to add logging code manually.

**What gets logged**:
- Method entry and exit
- Execution duration
- Request ID (for tracing)
- User ID (from JWT)
- Exceptions with stack traces

**Example log output**:
```
2026-02-02 13:00:00 [A1B2C3D4] [user-123] - Entering EmployeeService.createEmployee()
2026-02-02 13:00:01 [A1B2C3D4] [user-123] - Exiting EmployeeService.createEmployee() - Duration: 1234ms
```

---

## Data Model Questions

### What is the organizational hierarchy?
```
Company (Tenant)
└── Branch (Location)
    └── Department (Scheduling Unit)
        └── Employee (Staff Member)
```

Each level has a specific purpose:
- **Company**: Multi-tenant isolation
- **Branch**: Physical/logical location
- **Department**: Scheduling scope
- **Employee**: Individual staff member

### Why use JSONB columns?
JSONB provides schema flexibility without database migrations.

**Use cases**:
- `Employee.additional_fields`: Custom attributes per company
- `Employee.on_planned_leave`: Dynamic leave dates
- `Schedule.schedule_data`: Complex optimization results
- `Shift.time_blocks`: Variable shift structures

**Example**:
```json
{
  "additional_fields": {
    "skills": ["cashier", "inventory", "customer_service"],
    "certifications": ["food_safety", "first_aid"],
    "preferred_shifts": ["morning", "afternoon"]
  }
}
```

### What is soft delete?
Instead of permanently deleting records, we mark them as inactive.

**Benefits**:
- Audit trail preserved
- Can restore deleted records
- Historical data intact

**Implementation**:
```sql
-- Instead of: DELETE FROM employees WHERE id = 42;
UPDATE employees SET is_active = FALSE, deleted_at = NOW() WHERE id = 42;
```

---

## API Questions

### Why do all responses use `ApiResponseDTO`?
Provides a consistent structure for all API responses:

```json
{
  "success": true,
  "data": { /* actual data */ },
  "message": "Operation successful",
  "timestamp": "2026-02-02T13:00:00"
}
```

**Benefits**:
- Clients know what to expect
- Easy to add metadata (timestamps, messages)
- Consistent error handling

### How does pagination work?
Use `page` and `size` query parameters:

```bash
GET /api/v1/employees?page=0&size=20&sortBy=lastName&sortDirection=ASC
```

**Response**:
```json
{
  "content": [ /* 20 employees */ ],
  "page": 0,
  "size": 20,
  "totalElements": 145,
  "totalPages": 8,
  "last": false
}
```

**Page numbers are zero-indexed**: First page is `page=0`.

### What is the difference between `accessToken` and `idToken`?
- **Access Token**: Used for API authorization (`Authorization: Bearer <accessToken>`)
- **ID Token**: Contains user profile information (name, email, etc.)
- **Refresh Token**: Used to get new access tokens when they expire

**Use the access token** for all API requests.

---

## Schedule Generation Questions

### Why is schedule generation async?
Schedule optimization can take 5-45 minutes depending on complexity (number of employees, shifts, constraints). Running synchronously would:
- ❌ Block the HTTP connection
- ❌ Risk timeouts
- ❌ Prevent other requests

**Async approach**:
1. API returns immediately (202 Accepted)
2. Processing happens in background
3. Webhook notifies when complete
4. Client can poll status endpoint

### What happens if schedule generation fails?
1. Schedule status is set to `FAILED`
2. Error details are logged
3. Webhook is called with error information
4. User can retry or modify constraints

### How do I check schedule status?
```bash
GET /api/v1/schedules/{id}
```

**Status values**:
- `PENDING`: Created, not yet sent to engine
- `PROCESSING`: Sent to engine, waiting for result
- `COMPLETED`: Engine returned result
- `FAILED`: Engine error or timeout
- `APPROVED`: Manager approved
- `REJECTED`: Manager rejected

---

## Deployment Questions

### What are the minimum system requirements?
- **Java**: 21 (Eclipse Temurin or OpenJDK)
- **PostgreSQL**: 15+
- **Memory**: 2GB RAM minimum, 4GB recommended
- **CPU**: 2 cores minimum, 4 cores recommended
- **Disk**: 10GB minimum

### How do I run locally?
```bash
# 1. Set environment variables
export POSTGRES_DB_HOST=localhost
export POSTGRES_DB_NAME=genesis
export POSTGRES_DB_USER=genesis_user
export POSTGRES_DB_PASS=your_password
export AWS_COGNITO_USER_POOL_ID=your_pool_id
export AWS_COGNITO_CLIENT_ID=your_client_id
export AWS_COGNITO_CLIENT_SECRET=your_client_secret

# 2. Run migrations
./mvnw flyway:migrate

# 3. Start application
./mvnw spring-boot:run
```

See [OPERATIONS.md - Local Development](OPERATIONS.md#local-development) for complete guide.

### How do I deploy with Docker?
```bash
# Build image
docker build -t genesis-api:latest .

# Run with Docker Compose
docker-compose up -d
```

See [OPERATIONS.md - Docker Deployment](OPERATIONS.md#docker-deployment) for `docker-compose.yml` example.

### How do I check if the application is healthy?
```bash
curl http://localhost:8080/actuator/health
```

**Expected response**:
```json
{
  "status": "UP",
  "components": {
    "db": {"status": "UP"},
    "diskSpace": {"status": "UP"}
  }
}
```

---

## Troubleshooting Questions

### I'm getting "Connection refused" errors
**Possible causes**:
1. PostgreSQL is not running
2. Wrong database host/port
3. Firewall blocking connection

**Solutions**:
```bash
# Check PostgreSQL is running
pg_isready -h localhost -p 5432

# Verify connection
psql -h localhost -U genesis_user -d genesis

# Check environment variables
echo $POSTGRES_DB_HOST
```

### I'm getting "401 Unauthorized" errors
**Possible causes**:
1. Missing `Authorization` header
2. Expired JWT token
3. Invalid token signature

**Solutions**:
```bash
# Check token is included
curl -H "Authorization: Bearer YOUR_TOKEN" http://localhost:8080/api/v1/employees

# Decode token to check expiration (use jwt.io)

# Get a fresh token
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{"emailAddress": "user@example.com", "password": "password"}'
```

### Flyway migration failed, what do I do?
**If migration checksum mismatch**:
```bash
./mvnw flyway:repair
```

**If migration SQL error**:
1. Fix the SQL in the migration file
2. Manually rollback the partial migration in database
3. Run `./mvnw flyway:migrate` again

**Nuclear option (CAUTION: deletes all data)**:
```bash
./mvnw flyway:clean
./mvnw flyway:migrate
```

### Schedule generation is timing out
**Increase timeouts in `application.yaml`**:
```yaml
server:
  async:
    request-timeout: -1  # Infinite
  tomcat:
    connection-timeout: 2700000  # 45 minutes

feign:
  client:
    config:
      engine:
        connectTimeout: 0
        readTimeout: 0
```

---

## Best Practices Questions

### Should I expose entities directly in API responses?
**No!** Always use DTOs.

**Why**:
- Entities contain internal implementation details
- Entities may have circular references (infinite JSON)
- DTOs allow API contract to evolve independently

**Example**:
```java
// ❌ Bad
@GetMapping("/{id}")
public Employee getEmployee(@PathVariable Long id) {
    return employeeRepository.findById(id);
}

// ✅ Good
@GetMapping("/{id}")
public ResponseEntity<ApiResponseDTO<EmployeeResponseDTO>> getEmployee(@PathVariable Long id) {
    Employee employee = employeeRepository.findById(id).orElseThrow();
    EmployeeResponseDTO dto = employeeMapper.toResponseDTO(employee);
    return ResponseEntity.ok(new ApiResponseDTO<>(true, dto, "Success"));
}
```

### How should I handle validation?
Use Bean Validation annotations on DTOs:

```java
public record EmployeeRequestDTO(
    @NotBlank(message = "First name is required")
    String firstName,
    
    @Email(message = "Invalid email format")
    String emailAddress,
    
    @Pattern(regexp = "\\+\\d{10,15}", message = "Invalid phone number")
    String phoneNumber
) {}
```

Add `@Valid` in controller:
```java
@PostMapping
public ResponseEntity<ApiResponseDTO<EmployeeResponseDTO>> createEmployee(
    @Valid @RequestBody EmployeeRequestDTO request
) {
    // Validation happens automatically
}
```

### Should I use `@Transactional`?
**Yes**, for operations that modify multiple entities:

```java
@Service
public class EmployeeService {
    
    @Transactional
    public void transferEmployee(Long employeeId, Long newDepartmentId) {
        Employee employee = employeeRepository.findById(employeeId).orElseThrow();
        Department newDept = departmentRepository.findById(newDepartmentId).orElseThrow();
        
        employee.setDepartment(newDept);
        employeeRepository.save(employee);
        
        // If any operation fails, entire transaction rolls back
    }
}
```

---

## Need More Help?

- **Code Examples**: See [EXAMPLES.md](EXAMPLES.md)
- **Technical Terms**: See [GLOSSARY.md](GLOSSARY.md)
- **Architecture**: See [ARCHITECTURE.md](ARCHITECTURE.md)
- **Operations**: See [OPERATIONS.md](OPERATIONS.md)
- **Modules**: See [MODULES.md](MODULES.md)
