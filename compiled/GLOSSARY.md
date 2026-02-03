# Glossary

A comprehensive reference of technical terms, acronyms, and concepts used in the Genesis platform.

---

## A

### ADR (Architectural Decision Record)
A document that captures an important architectural decision made along with its context and consequences.

**Example**: ADR-003 documents the decision to use AOP for logging.

### AOP (Aspect-Oriented Programming)
A programming paradigm that allows separation of cross-cutting concerns (like logging, security) from business logic.

**In Genesis**: The `LogAspect` uses AOP to automatically log all service and controller methods without modifying their code.

### API (Application Programming Interface)
A set of rules and protocols for building and interacting with software applications.

**In Genesis**: RESTful HTTP APIs exposed at `/api/v1/*` endpoints.

### Async (Asynchronous)
Operations that run in the background without blocking the main thread.

**In Genesis**: Schedule generation runs asynchronously using `@Async` annotation, allowing the API to return immediately while processing continues.

### Audit Fields
Standard database columns that track when records are created and modified.

**In Genesis**: `created_at`, `updated_at`, `created_by`, `updated_by`

---

## B

### Bean
A Java object managed by the Spring Framework container.

**Example**: Services, repositories, and controllers are all Spring beans.

### Branch
A physical or logical location within a company (e.g., store, office, warehouse).

**Hierarchy**: Company → Branch → Department

---

## C

### Cognito (AWS Cognito)
Amazon's managed authentication service that handles user sign-up, sign-in, and access control.

**In Genesis**: Used for user authentication. Each employee has a `cognito_sub` that links to their Cognito user.

### Controller
The layer that handles HTTP requests and returns responses.

**In Genesis**: Classes ending with `Controller` (e.g., `EmployeeController`) define REST API endpoints.

### CRUD
Create, Read, Update, Delete - the four basic operations for persistent storage.

**Example**: 
- Create: `POST /api/v1/employees`
- Read: `GET /api/v1/employees/{id}`
- Update: `PUT /api/v1/employees/{id}`
- Delete: `DELETE /api/v1/employees/{id}`

---

## D

### Department
An organizational unit within a branch. The primary scope for scheduling.

**Example**: Sales Department, Operations Department

### DTO (Data Transfer Object)
An object that carries data between processes, typically used for API requests and responses.

**In Genesis**: All API requests use `*RequestDTO` and responses use `*ResponseDTO` classes.

**Why**: DTOs decouple the API contract from the internal entity structure, allowing independent evolution.

---

## E

### Entity
A Java class that represents a database table.

**In Genesis**: Classes in `entity/` packages (e.g., `Employee`, `Schedule`) are JPA entities.

### ERD (Entity Relationship Diagram)
A visual representation of database tables and their relationships.

**See**: [TECHNICAL.md](TECHNICAL.md#entity-relationship-diagram)

---

## F

### Feign
A declarative HTTP client library that makes writing web service clients easier.

**In Genesis**: Used to communicate with the external Schedule Engine.

**Example**:
```java
@FeignClient(name = "engine", url = "${engine.url}")
public interface EngineClient {
    @PostMapping("/optimize")
    ScheduleResult optimize(@RequestBody OptimizationRequest request);
}
```

### Flyway
A database migration tool that versions and manages schema changes.

**In Genesis**: Migration files are in `src/main/resources/db/migrations/V*.sql`

---

## H

### HikariCP
A high-performance JDBC connection pool.

**In Genesis**: Manages database connections efficiently with configurable pool size.

---

## J

### JPA (Java Persistence API)
A specification for object-relational mapping in Java.

**In Genesis**: Hibernate is the JPA implementation used.

### JSONB
A PostgreSQL data type for storing JSON data in binary format, allowing efficient querying.

**In Genesis**: Used for flexible fields like `Employee.additional_fields`, `Schedule.schedule_data`

### JWT (JSON Web Token)
A compact, URL-safe token format for securely transmitting information between parties.

**In Genesis**: Cognito issues JWT tokens after login. These tokens are validated on every API request.

**Structure**: `header.payload.signature`

---

## M

### Mapper
A class that converts between entities and DTOs.

**In Genesis**: Classes ending with `Mapper` (e.g., `EmployeeMapper`) handle conversions.

**Why**: Keeps conversion logic separate and reusable.

### MDC (Mapped Diagnostic Context)
A mechanism for enriching log messages with contextual information.

**In Genesis**: Every request has a `requestId` and `userId` in MDC, making it easy to trace logs.

### Migration
A versioned SQL script that modifies the database schema.

**In Genesis**: Flyway migrations follow the pattern `V{version}__{description}.sql`

### Module
A self-contained vertical slice of functionality containing all layers (controller, service, repository, entity, DTOs).

**In Genesis**: 12 modules including Auth, Employee, Schedule, etc.

---

## O

### OAuth 2.0
An authorization framework that enables applications to obtain limited access to user accounts.

**In Genesis**: Cognito implements OAuth 2.0 for token-based authentication.

---

## P

### Pagination
Dividing large result sets into smaller pages.

**In Genesis**: Use `page` and `size` query parameters.

**Example**: `GET /api/v1/employees?page=0&size=20`

### Pointcut
In AOP, a predicate that matches join points (method executions).

**In Genesis**: `LogAspect` defines pointcuts for service and controller methods.

---

## R

### Repository
The data access layer that interacts with the database.

**In Genesis**: Interfaces extending `JpaRepository` (e.g., `EmployeeRepository`)

### REST (Representational State Transfer)
An architectural style for designing networked applications using HTTP.

**In Genesis**: All APIs follow RESTful conventions.

### RFC 7807 (Problem Details)
A standard format for HTTP API error responses.

**In Genesis**: All errors return Problem Details format with `type`, `title`, `status`, `detail`, `instance`.

---

## S

### Schedule
A generated work schedule for a department over a date range.

**Lifecycle**: PENDING → PROCESSING → COMPLETED → APPROVED

### Service
The business logic layer between controllers and repositories.

**In Genesis**: Classes ending with `Service` (e.g., `EmployeeService`) contain business rules.

### Shift
A template for work shifts (e.g., "Morning Shift", "Night Shift").

**Components**: Shift name, type, and time blocks.

### Soft Delete
Marking records as deleted without physically removing them from the database.

**In Genesis**: Most entities have `is_active` or `deleted_at` fields.

### Spring Boot
An opinionated framework for building production-ready Spring applications.

**In Genesis**: Version 3.4.0 is used.

---

## T

### Tenant
In multi-tenancy, an isolated customer or organization.

**In Genesis**: Each `Company` is a tenant with isolated data.

---

## V

### Vertical Slice Architecture
An architectural pattern where features are organized as self-contained slices rather than horizontal layers.

**In Genesis**: Each module (Employee, Schedule, etc.) contains all layers needed for that feature.

**Benefits**: High cohesion, low coupling, easier to understand and modify.

---

## W

### Webhook
An HTTP callback that notifies external systems of events.

**In Genesis**: Used to notify clients when async operations (like Cognito account creation) complete.

**Example**: `POST https://your-app.com/webhook` with status update.

---

## Acronyms Quick Reference

| Acronym | Full Term | Description |
|---------|-----------|-------------|
| ADR | Architectural Decision Record | Documents important design decisions |
| AOP | Aspect-Oriented Programming | Cross-cutting concerns pattern |
| API | Application Programming Interface | HTTP endpoints for client interaction |
| CRUD | Create, Read, Update, Delete | Basic data operations |
| DTO | Data Transfer Object | API request/response objects |
| ERD | Entity Relationship Diagram | Database schema visualization |
| JPA | Java Persistence API | ORM specification |
| JWT | JSON Web Token | Authentication token format |
| MDC | Mapped Diagnostic Context | Logging context enrichment |
| ORM | Object-Relational Mapping | Database-to-object conversion |
| REST | Representational State Transfer | API architectural style |
| SES | Simple Email Service | AWS email service |
| SNS | Simple Notification Service | AWS pub/sub messaging |
| SQS | Simple Queue Service | AWS message queue |

---

## See Also

- [README.md](README.md) - Project overview
- [ARCHITECTURE.md](ARCHITECTURE.md) - System design
- [TECHNICAL.md](TECHNICAL.md) - Technical specifications
- [MODULES.md](MODULES.md) - Module reference
- [OPERATIONS.md](OPERATIONS.md) - Deployment guide
