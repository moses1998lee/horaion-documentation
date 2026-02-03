# System Architecture

## Table of Contents
- [Overview](#overview)
- [System Context](#system-context)
- [Application Architecture](#application-architecture)
- [Security Architecture](#security-architecture)
- [Data Flow Patterns](#data-flow-patterns)
- [Async Processing](#async-processing)
- [Architectural Decisions](#architectural-decisions)

---

## Overview

Genesis is built on **Spring Boot 3.4.0** with **Java 21**, following a **Vertical Slice Architecture** pattern. The system is designed for:
- **Multi-tenancy**: Supporting multiple companies with data isolation
- **Scalability**: Async processing for long-running operations
- **Security**: JWT-based authentication with AWS Cognito
- **Extensibility**: Plugin-like module system for new features

### Core Design Principles
1. **Domain-Driven Design**: Business logic organized by domain modules
2. **API-First**: RESTful APIs with OpenAPI documentation
3. **Security by Default**: All endpoints authenticated unless explicitly public
4. **Observability**: Comprehensive logging with request tracing

---

## System Context

Genesis integrates with several external systems to provide complete workforce management capabilities.

```mermaid
C4Context
    title System Context Diagram - Genesis Platform

    Person(admin, "System Administrator", "Manages companies, branches, departments")
    Person(manager, "Department Manager", "Creates schedules, manages employees")
    Person(employee, "Employee", "Views schedule, submits leave requests")
    
    System(genesis, "Genesis API", "Workforce Management Platform")
    
    System_Ext(cognito, "AWS Cognito", "User authentication and identity management")
    System_Ext(engine, "Schedule Engine", "Optimization engine for schedule generation")
    System_Ext(ses, "AWS SES", "Email notifications")
    System_Ext(sqs, "AWS SQS/SNS", "Async message queue")
    
    Rel(admin, genesis, "Manages organization", "HTTPS/REST")
    Rel(manager, genesis, "Creates schedules", "HTTPS/REST")
    Rel(employee, genesis, "Views schedule", "HTTPS/REST")
    
    Rel(genesis, cognito, "Authenticates users", "AWS SDK")
    Rel(genesis, engine, "Generates schedules", "REST/Feign")
    Rel(genesis, ses, "Sends emails", "AWS SDK")
    Rel(genesis, sqs, "Publishes events", "AWS SDK")
    
    Rel(engine, genesis, "Webhook callback", "HTTPS/REST")
```

### External Dependencies

| System | Purpose | Protocol | Notes |
|--------|---------|----------|-------|
| **AWS Cognito** | User authentication, user pool management | AWS SDK | JWT token provider |
| **Schedule Engine** | Optimization algorithm for shift scheduling | REST API (Feign) | Timeout: 45 minutes |
| **AWS SES** | Transactional emails (welcome, notifications) | AWS SDK | Async via SQS |
| **AWS SQS/SNS** | Event-driven messaging | AWS SDK | Decouples heavy operations |
| **PostgreSQL** | Primary data store | JDBC | HikariCP connection pool |

---

## Application Architecture

### Package Structure

The codebase follows a clean modular structure:

```
com.resetrix.genesis/
├── GenesisApplication.java          # Spring Boot entry point
├── configurations/                   # Spring configuration classes
│   ├── SecurityConfiguration.java   # JWT, CORS, auth rules
│   ├── AsyncConfiguration.java      # Async executor setup
│   ├── FeignConfiguration.java      # HTTP client config
│   └── DatabaseConfiguration.java   # DataSource, JPA config
├── modules/                          # Business domain modules
│   ├── auth/                        # Authentication & registration
│   ├── company/                     # Company management
│   ├── branch/                      # Branch management
│   ├── department/                  # Department management
│   ├── employee/                    # Employee CRUD + Cognito sync
│   ├── shift/                       # Shift templates
│   ├── schedule/                    # Schedule generation & approval
│   ├── rule/                        # Business rules engine
│   ├── demandforecast/              # Demand forecasting
│   ├── employeeleaveavailability/   # Leave requests
│   └── employeerole/                # Role definitions
├── shared/                           # Cross-cutting concerns
│   ├── aspects/                     # AOP logging (LogAspect)
│   ├── dtos/                        # Shared DTOs (ApiResponseDTO)
│   ├── securities/                  # JWT validators, converters
│   ├── helpers/                     # Utility classes
│   ├── validations/                 # Custom validators
│   └── enums/                       # Shared enumerations
├── exceptions/                       # Exception hierarchy
│   └── handlers/                    # Global exception handlers
└── diagnostics/                      # Health checks, connectivity tests
```

### Vertical Slice Architecture

Each module is a **vertical slice** containing all layers needed for that feature:

```mermaid
graph TB
    subgraph "Employee Module (Vertical Slice)"
        EC[EmployeeController<br/>REST API Layer]
        ES[EmployeeService<br/>Business Logic]
        ER[EmployeeRepository<br/>Data Access]
        EE[Employee Entity<br/>Domain Model]
        EDTO[DTOs<br/>Request/Response]
        EM[EmployeeMapper<br/>Entity ↔ DTO]
    end
    
    subgraph "Schedule Module (Vertical Slice)"
        SC[ScheduleController]
        SS[ScheduleService]
        SR[ScheduleRepository]
        SE[Schedule Entity]
        SDTO[DTOs]
        SM[ScheduleMapper]
    end
    
    EC --> ES --> ER --> EE
    EC --> EDTO
    ES --> EM
    
    SC --> SS --> SR --> SE
    SC --> SDTO
    SS --> SM
    
    ES -.->|Uses| SS
    
    style EC fill:#e1f5fe
    style SC fill:#e1f5fe
    style ES fill:#fff3e0
    style SS fill:#fff3e0
    style ER fill:#e8f5e8
    style SR fill:#e8f5e8
```

**Benefits of Vertical Slices:**
- **Cohesion**: All code for a feature is in one place
- **Independence**: Modules can evolve separately
- **Testability**: Each module can be tested in isolation
- **Ownership**: Clear boundaries for team ownership

---

## Security Architecture

### Authentication Flow (AWS Cognito + JWT)

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API as Genesis API
    participant Security as SecurityConfiguration
    participant Cognito as AWS Cognito
    participant DB as PostgreSQL
    
    Note over User,DB: Registration Flow
    User->>Frontend: Enter registration details
    Frontend->>API: POST /auth/register
    API->>Cognito: Create user in user pool
    Cognito-->>API: User created (unconfirmed)
    API->>DB: Create Employee record
    Cognito->>User: Send confirmation email
    
    Note over User,DB: Confirmation Flow
    User->>Frontend: Enter confirmation code
    Frontend->>API: POST /auth/confirm
    API->>Cognito: Confirm user
    Cognito-->>API: User confirmed
    
    Note over User,DB: Login Flow
    User->>Frontend: Enter credentials
    Frontend->>API: POST /auth/login
    API->>Cognito: Authenticate
    Cognito-->>API: JWT tokens (access, ID, refresh)
    API-->>Frontend: Return tokens
    
    Note over User,DB: Authenticated Request
    Frontend->>API: GET /employees (Bearer token)
    API->>Security: Validate JWT signature
    Security->>Security: Decode token claims
    Security->>Security: Extract user ID (sub)
    Security-->>API: Authentication object
    API->>DB: Query with user context
    DB-->>API: Data
    API-->>Frontend: Response
```

### Security Configuration

The `SecurityConfiguration` class defines:

1. **Public Endpoints** (no authentication required):
   - `/actuator/health` - Health checks
   - `/auth/**` - Registration, login, confirmation
   - `/swagger-ui/**` - API documentation

2. **Protected Endpoints** (JWT required):
   - All other endpoints require valid JWT in `Authorization: Bearer <token>` header

3. **IP Whitelisting**:
   - Engine callback endpoints (`/schedule/update-status`) validate source IP
   - Configured via `ENGINE_ALLOWED_IPS` environment variable

4. **JWT Validation**:
   - Signature verification using Cognito's public keys (JWKS)
   - Expiration check
   - Audience (`aud`) and issuer (`iss`) validation

### Authorization Model

```mermaid
graph TD
    Request[Incoming Request] --> JWT[JWT Token]
    JWT --> Validate[Validate Signature]
    Validate --> Extract[Extract Claims]
    Extract --> Sub[User ID - sub claim]
    Extract --> Email[Email - email claim]
    Extract --> Groups[User Groups - cognito:groups]
    
    Sub --> Context[Security Context]
    Email --> Context
    Groups --> Context
    
    Context --> Controller[Controller Method]
    Controller --> Service[Service Layer]
    Service --> Check{Authorization Check}
    Check -->|Authorized| Execute[Execute Business Logic]
    Check -->|Denied| Error[403 Forbidden]
```

---

## Data Flow Patterns

### Standard CRUD Flow

```mermaid
sequenceDiagram
    participant Client
    participant Controller
    participant Service
    participant Mapper
    participant Repository
    participant DB
    participant LogAspect
    
    Client->>Controller: POST /api/v1/employees
    Controller->>LogAspect: @Around advice triggered
    LogAspect->>LogAspect: Log request, generate requestId
    LogAspect->>Controller: Proceed
    Controller->>Controller: Validate request DTO
    Controller->>Service: createEmployee(requestDTO)
    Service->>LogAspect: @Around advice triggered
    LogAspect->>Service: Proceed
    Service->>Service: Business validation
    Service->>Mapper: toEntity(requestDTO)
    Mapper-->>Service: Employee entity
    Service->>Repository: save(entity)
    Repository->>DB: INSERT INTO employees
    DB-->>Repository: Saved entity
    Repository-->>Service: Employee entity
    Service->>Mapper: toResponseDTO(entity)
    Mapper-->>Service: EmployeeResponseDTO
    Service-->>LogAspect: Return result
    LogAspect->>LogAspect: Log success, duration
    LogAspect-->>Controller: Return result
    Controller->>Controller: Wrap in ApiResponseDTO
    Controller-->>Client: 201 Created + response
```

### Error Handling Flow

```mermaid
graph TD
    Request[HTTP Request] --> Controller
    Controller --> Service
    Service --> Exception{Exception Thrown?}
    
    Exception -->|ResourceNotFoundException| Handler1[GlobalExceptionHandler]
    Exception -->|ValidationException| Handler1
    Exception -->|RuntimeException| Handler1
    Exception -->|Checked Exception| Handler1
    
    Handler1 --> Log[Log Error with Stack Trace]
    Log --> Build[Build ProblemDetailsDTO]
    Build --> Response[HTTP Response]
    
    Response --> Status{HTTP Status}
    Status -->|404| NotFound[Not Found Response]
    Status -->|400| BadRequest[Bad Request Response]
    Status -->|500| ServerError[Internal Server Error]
    
    NotFound --> Client[Client]
    BadRequest --> Client
    ServerError --> Client
```

---

## Async Processing

Genesis uses Spring's `@Async` for long-running operations, particularly schedule generation.

### Async Configuration

```java
// AsyncConfiguration.java
@Configuration
@EnableAsync
public class AsyncConfiguration {
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

### Schedule Generation Flow (Async)

```mermaid
sequenceDiagram
    participant Client
    participant API as ScheduleController
    participant Service as ScheduleService
    participant Engine as External Engine
    participant DB as PostgreSQL
    participant Webhook as Webhook Callback
    
    Note over Client,Webhook: Async Schedule Generation
    
    Client->>API: POST /schedules (create schedule request)
    API->>Service: createSchedule()
    Service->>DB: Save schedule (status: PENDING)
    Service->>Service: @Async generateScheduleAsync()
    Service-->>API: Return immediately
    API-->>Client: 202 Accepted (schedule created)
    
    Note over Service,Engine: Async execution in background thread
    
    Service->>Service: Gather constraints (rules, demand, employees)
    Service->>Engine: POST /optimize (Feign client, timeout: 45min)
    
    alt Success
        Engine-->>Service: Optimization result
        Service->>DB: Update schedule (status: COMPLETED)
        Service->>Webhook: POST callback (if configured)
        Webhook-->>Client: Notification
    else Failure
        Engine-->>Service: Error
        Service->>DB: Update schedule (status: FAILED)
        Service->>Webhook: POST callback with error
    end
```

### Timeout Configuration

For long-running operations, Genesis configures extended timeouts:

```yaml
# application.yaml
server:
  async:
    request-timeout: -1  # No timeout (45 minutes)
  tomcat:
    connection-timeout: 2700000  # 45 minutes
    keep-alive-timeout: 2700000

feign:
  client:
    config:
      engine:
        connectTimeout: 0  # Infinite (45 minutes)
        readTimeout: 0     # Infinite (45 minutes)
        retryer: never     # No retries for idempotency
```

---

## Architectural Decisions

### ADR-001: Vertical Slice Architecture

**Status**: Accepted

**Context**: Traditional layered architecture (Controller → Service → Repository) creates tight coupling across layers and makes it difficult to understand complete features.

**Decision**: Organize code by business features (modules), with each module containing all necessary layers.

**Consequences**:
- ✅ Easier to locate all code for a feature
- ✅ Modules can be developed independently
- ✅ Clearer ownership boundaries
- ❌ Some code duplication across modules
- ❌ Requires discipline to avoid cross-module dependencies

---

### ADR-002: AWS Cognito for Authentication

**Status**: Accepted

**Context**: Need managed authentication with email verification, password reset, and MFA support.

**Decision**: Use AWS Cognito as the identity provider, with JWT tokens for API authentication.

**Consequences**:
- ✅ Managed service (no custom auth logic)
- ✅ Built-in security features (MFA, password policies)
- ✅ Scalable and reliable
- ❌ AWS vendor lock-in
- ❌ Additional cost for Cognito usage

---

### ADR-003: AOP for Cross-Cutting Logging

**Status**: Accepted

**Context**: Need consistent logging across all service and controller methods without manual instrumentation.

**Decision**: Implement `LogAspect` using Spring AOP to automatically log method entry, exit, duration, and errors.

**Consequences**:
- ✅ Consistent logging format
- ✅ No boilerplate in business code
- ✅ Request tracing with MDC
- ❌ Small performance overhead
- ❌ Debugging can be harder (proxy wrapping)

**Implementation**: See `com.resetrix.genesis.shared.aspects.LogAspect`

---

### ADR-004: External Optimization Engine

**Status**: Accepted

**Context**: Schedule optimization is computationally expensive and requires specialized algorithms.

**Decision**: Delegate schedule generation to an external optimization engine via REST API.

**Consequences**:
- ✅ Separation of concerns (API vs. optimization)
- ✅ Can scale optimization independently
- ✅ Easier to swap optimization algorithms
- ❌ Network latency and reliability concerns
- ❌ Need to handle long-running operations (async)

---

### ADR-005: Flyway for Database Migrations

**Status**: Accepted

**Context**: Need version-controlled, repeatable database schema changes.

**Decision**: Use Flyway for all schema migrations with `validate-on-migrate` enabled.

**Consequences**:
- ✅ Version control for database schema
- ✅ Automatic migration on startup
- ✅ Rollback support
- ❌ Requires careful migration script authoring
- ❌ Failed migrations can block startup

**Migration Files**: `src/main/resources/db/migrations/V*.sql`

---

## Next Steps

For implementation details, see:
- [TECHNICAL.md](TECHNICAL.md) - Data models, API patterns, error handling
- [MODULES.md](MODULES.md) - Individual module documentation
- [OPERATIONS.md](OPERATIONS.md) - Deployment and monitoring



---

## Document Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-30 | Genesis Team | Initial architecture documentation |
| 2.0 | 2026-02-02 | Genesis Team | Verified and cleaned - removed template content |

