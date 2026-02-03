# System Architecture

> **Genesis Workforce Management Platform - System Architecture**

## Table of Contents

* [Overview](02_SYSTEM_ARCHITECTURE.md#overview)
* [System Context](02_SYSTEM_ARCHITECTURE.md#system-context)
* [Application Architecture](02_SYSTEM_ARCHITECTURE.md#application-architecture)
* [Security Architecture](02_SYSTEM_ARCHITECTURE.md#security-architecture)
* [Data Flow Patterns](02_SYSTEM_ARCHITECTURE.md#data-flow-patterns)
* [Async Processing](02_SYSTEM_ARCHITECTURE.md#async-processing)
* [Architectural Decisions](02_SYSTEM_ARCHITECTURE.md#architectural-decisions)

***

## Overview

Genesis is built on **Spring Boot 3.4.0** with **Java 21**, following a **Vertical Slice Architecture** pattern. The system is designed for:

* **Multi-tenancy**: Supporting multiple companies with data isolation
* **Scalability**: Async processing for long-running operations
* **Security**: JWT-based authentication with AWS Cognito
* **Extensibility**: Plugin-like module system for new features

### Core Design Principles

1. **Domain-Driven Design**: Business logic organized by domain modules
2. **API-First**: RESTful APIs with OpenAPI documentation
3. **Security by Default**: All endpoints authenticated unless explicitly public
4. **Observability**: Comprehensive logging with request tracing

***

## System Context

Genesis integrates with several external systems to provide complete workforce management capabilities.

```mermaid
graph TB
    subgraph Client_Layer [Client Layer]
        WebApp[Web Application<br/>React/Angular]
        Mobile[Mobile App<br/>iOS/Android]
        Tools[API Testing Tools<br/>Postman/Insomnia]
    end

    subgraph Gateway_Layer [Gateway Layer]
        LB[Load Balancer<br/>Nginx/AWS ALB]
    end

    subgraph Application_Server [Genesis Application Server - Spring Boot]
        subgraph Web_Layer [Web Layer]
            Filters[Security Filters]
            Exceptions[Exception Handlers]
            Controllers[REST Controllers]
            Filters --> Controllers
            Exceptions --> Controllers
        end

        subgraph Business_Layer [Business Layer]
            AuthS[Auth Service]
            EmpS[Employee Service]
            SchedS[Schedule Service]
            RuleS[Rule Service]
            DemandS[Demand Forecast Service]
        end

        subgraph Integration_Layer [Integration Layer]
            CognitoS[Cognito Service]
            WebhookS[Webhook Service]
            Feign[Feign Client]
        end

        subgraph Data_Layer [Data Layer]
            JPA[Spring Data JPA]
            Flyway[Flyway Migrations]
        end
    end

    subgraph External_Services [External Services]
        CognitoProvider[AWS Cognito<br/>Identity Provider]
        ExtWebhook[External Webhook<br/>Endpoints]
        AI[Schedule Engine<br/>AI Optimization]
    end

    subgraph Data_Store [Data Store]
        DB[(PostgreSQL 15<br/>Primary Database)]
    end

    %% Client -> Gateway
    WebApp --> LB
    Mobile --> LB
    Tools --> LB

    %% Gateway -> App
    LB --> Controllers

    %% Controller -> Business
    Controllers --> AuthS
    Controllers --> EmpS
    Controllers --> SchedS
    Controllers --> RuleS
    Controllers --> DemandS

    %% Business -> Integration / Data
    AuthS --> CognitoS
    EmpS --> CognitoS
    EmpS --> WebhookS
    EmpS --> JPA
    SchedS --> Feign
    SchedS --> JPA
    RuleS --> JPA
    DemandS --> JPA

    %% Integration -> External
    CognitoS --> CognitoProvider
    WebhookS --> ExtWebhook
    Feign --> AI

    %% Data -> Store
    JPA --> DB
    Flyway --> DB

    %% Styling
    style Application_Server fill:#f5f5f5,stroke:#333,stroke-width:2px
    style Web_Layer fill:#e3f2fd,stroke:#2196f3
    style Business_Layer fill:#fff3e0,stroke:#ff9800
    style Integration_Layer fill:#f3e5f5,stroke:#9c27b0
    style Data_Layer fill:#e8f5e8,stroke:#4caf50
    style Gateway_Layer fill:#fffde7,stroke:#fbc02d
    style Client_Layer fill:#fce4ec,stroke:#e91e63
    style External_Services fill:#eceff1,stroke:#607d8b
    style Data_Store fill:#e0e0e0,stroke:#333
    style CognitoProvider fill:#ff9900,color:white,stroke:#cf7600
    style DB fill:#336791,color:white,stroke:#254a6b
    style AI fill:#0288d1,color:white,stroke:#01579b
```

> **Diagram Explanation**: This high-level overview shows the three main tiers of the Genesis ecosystem: **User Roles** (who interacts with the system), the core **Genesis API**, and **External Services** (AWS infrastructure and Schedule Engine) that provide essential capabilities.

### External Dependencies

| System              | Purpose                                       | Protocol         | Notes                      |
| ------------------- | --------------------------------------------- | ---------------- | -------------------------- |
| **AWS Cognito**     | User authentication, user pool management     | AWS SDK          | JWT token provider         |
| **Schedule Engine** | Optimization algorithm for shift scheduling   | REST API (Feign) | Timeout: 45 minutes        |
| **AWS SES**         | Transactional emails (welcome, notifications) | AWS SDK          | Async via SQS              |
| **AWS SQS/SNS**     | Event-driven messaging                        | AWS SDK          | Decouples heavy operations |
| **PostgreSQL**      | Primary data store                            | JDBC             | HikariCP connection pool   |

***

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
│   ├── DatabaseConfiguration.java   # DataSource, JPA config
│   ├── WebServerConfiguration.java  # TC/Tomcat timeouts
│   └── CognitoConfiguration.java    # AWS Cognito setup
├── modules/                          # Business domain modules
│   ├── auth/                        # Authentication & registration
│   ├── company/                     # Company management
│   ├── branch/                      # Branch management
│   ├── department/                  # Department management
│   ├── employee/                    # Employee CRUD + Cognito sync
│   ├── shift/                       # Shift templates
│   ├── schedule/                    # Schedule generation & approval
│   ├── rule/                        # Business rules engine
│   ├── ruleanswer/                  # Rule answers & configurations
│   ├── demandforecast/              # Demand forecasting
│   ├── employeeleaveavailability/   # Leave requests
│   └── employeerole/                # Role definitions
├── shared/                           # Cross-cutting concerns
│   ├── aspects/                     # AOP logging (LogAspect)
│   ├── constants/                   # Global constants
│   ├── converters/                  # JPA attribute converters
│   ├── dtos/                        # Shared DTOs (ApiResponseDTO)
│   ├── entities/                    # Base entities (BaseEntity)
│   ├── enums/                       # Shared enumerations
│   ├── feign/                       # Feign client interfaces
│   ├── helpers/                     # Utility classes
│   ├── processors/                  # Data processing logic
│   ├── resolvers/                   # Custom argument resolvers
│   ├── securities/                  # JWT validators, converters
│   └── validations/                 # Custom validators
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

> **Diagram Explanation**: Unlike traditional layered architectures (Controller -> Service -> Repository), Vertical Slices group code by **feature**. This diagram illustrates how the 'Employee' and 'Schedule' modules are self-contained with their own full stack of components, promoting modularity and reducing cross-domain coupling.

**Benefits of Vertical Slices:**

* **Cohesion**: All code for a feature is in one place
* **Independence**: Modules can evolve separately
* **Testability**: Each module can be tested in isolation
* **Ownership**: Clear boundaries for team ownership

***

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

> **Diagram Explanation**: This sequence details the security lifecycle: from **Registration** (creating a Cognito user), to **Confirmation** (verifying email), to **Login** (exchanging credentials for JWTs), and finally making **Authenticated Requests** using the Bearer token.

**Step-by-Step Breakdown:**

1. **Register**: User submits details; API creates a Cognito user (Unconfirmed) and a local Employee record.
2. **Confirm**: User enters the verification code sent to their email to activate their Cognito account.
3. **Login**: User authenticates with email/password and receives JWTs (Access, ID, Refresh).
4. **Access**: User includes the `Access Token` in the Authorization header to call protected API endpoints.

### Security Configuration

The `SecurityConfiguration` class defines:

1. **Public Endpoints** (no authentication required):
   * `/actuator/health` - Health checks
   * `/auth/**` - Registration, login, confirmation
   * `/swagger-ui/**` - API documentation
2. **Protected Endpoints** (JWT required):
   * All other endpoints require valid JWT in `Authorization: Bearer <token>` header
3. **IP Whitelisting**:
   * Engine callback endpoints (`/schedule/update-status`) validate source IP
   * Configured via `ENGINE_ALLOWED_IPS` environment variable
4. **JWT Validation**:
   * Signature verification using Cognito's public keys (JWKS)
   * Expiration check
   * Audience (`aud`) and issuer (`iss`) validation

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

> **Diagram Explanation**: This flow demonstrates how an incoming JWT is processed: validating the signature, extracting claims (User ID, Email, Groups), and populating the Spring Security Context to authorize execution in the Controller and Service layers.

***

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

> **Diagram Explanation**: A standard request lifecycle: The **Controller** validates DTOs, the **Service** applies business logic and uses **Mappers** to convert between DTOs and Entities, while **LogAspect** automatically handles auditing at the boundaries.

**Step-by-Step Breakdown:**

1. **Request**: Client sends data (Payload) to the API Endpoint.
2. **Validation**: Controller validates the input (e.g., checking required fields).
3. **Processing**: Service layer executes business logic and converts DTOs to Entities.
4. **Persistence**: Repository saves the data to the database using Hibernate/JPA.
5. **Response**: The system converts the saved data back to a Response DTO and returns it.

### Error Handling Flow

```mermaid
graph TD
    Request[HTTP Request] --> Controller
    Controller --> Service
    Service --> Exception{Exception Thrown?}
    
    %% Vary arrow lengths to prevent label overlap
    Exception -->|ResourceNotFoundException| Handler1[GlobalExceptionHandler]
    Exception --->|ValidationException| Handler1
    Exception ---->|RuntimeException| Handler1
    Exception ----->|Checked Exception| Handler1
    
    Handler1 --> Log[Log Error with Stack Trace]
    Log --> Build[Build ProblemDetailsDTO]
    Build --> Response[HTTP Response]
    
    Response --> Status{HTTP Status}
    %% Increase arrow lengths for better spacing
    Status -->|404| NotFound[Not Found Response]
    Status -->|400| BadRequest[Bad Request Response]
    Status -->|500| ServerError[Internal Server Error]
    
    NotFound --> Client[Client]
    BadRequest --> Client
    ServerError --> Client
```

> **Diagram Explanation**: The centralized exception handling mechanism: All exceptions thrown in the application are caught by the **GlobalExceptionHandler**, which logs the error and converts it into a standardized JSON error response with the appropriate HTTP status code.

***

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

> **Diagram Explanation**: The asynchronous optimization process: A client initiates a request, which immediately returns 'Accepted'. In the background, the service aggregates data, calls the external **Schedule Engine** (with a long timeout), and updates the database upon completion or failure.

**Step-by-Step Breakdown:**

1. **Initiate**: Client requests a schedule. API creates a "PENDING" record and immediately returns `202 Accepted`.
2. **Process**: A background thread gathers necessary data (Employees, Rules, Shifts).
3. **Optimize**: The system calls the external Schedule Engine (up to 45 min wait).
4. **Complete**: Engine returns the result. The system saves the schedule and notifies the user via Webhook.

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

***

## Architectural Decisions

### ADR-001: Vertical Slice Architecture

**Status**: Accepted

**Context**: Traditional layered architecture (Controller → Service → Repository) creates tight coupling across layers and makes it difficult to understand complete features.

**Decision**: Organize code by business features (modules), with each module containing all necessary layers.

**Consequences**:

* ✅ Easier to locate all code for a feature
* ✅ Modules can be developed independently
* ✅ Clearer ownership boundaries
* ❌ Some code duplication across modules
* ❌ Requires discipline to avoid cross-module dependencies

***

### ADR-002: AWS Cognito for Authentication

**Status**: Accepted

**Context**: Need managed authentication with email verification, password reset, and MFA support.

**Decision**: Use AWS Cognito as the identity provider, with JWT tokens for API authentication.

**Consequences**:

* ✅ Managed service (no custom auth logic)
* ✅ Built-in security features (MFA, password policies)
* ✅ Scalable and reliable
* ❌ AWS vendor lock-in
* ❌ Additional cost for Cognito usage

***

### ADR-003: AOP for Cross-Cutting Logging

**Status**: Accepted

**Context**: Need consistent logging across all service and controller methods without manual instrumentation.

**Decision**: Implement `LogAspect` using Spring AOP to automatically log method entry, exit, duration, and errors.

**Consequences**:

* ✅ Consistent logging format
* ✅ No boilerplate in business code
* ✅ Request tracing with MDC
* ❌ Small performance overhead
* ❌ Debugging can be harder (proxy wrapping)

**Implementation**: See `com.resetrix.genesis.shared.aspects.LogAspect`

***

### ADR-004: External Optimization Engine

**Status**: Accepted

**Context**: Schedule optimization is computationally expensive and requires specialized algorithms.

**Decision**: Delegate schedule generation to an external optimization engine via REST API.

**Consequences**:

* ✅ Separation of concerns (API vs. optimization)
* ✅ Can scale optimization independently
* ✅ Easier to swap optimization algorithms
* ❌ Network latency and reliability concerns
* ❌ Need to handle long-running operations (async)

***

### ADR-005: Flyway for Database Migrations

**Status**: Accepted

**Context**: Need version-controlled, repeatable database schema changes.

**Decision**: Use Flyway for all schema migrations with `validate-on-migrate` enabled.

**Consequences**:

* ✅ Version control for database schema
* ✅ Automatic migration on startup
* ✅ Rollback support
* ❌ Requires careful migration script authoring
* ❌ Failed migrations can block startup

**Migration Files**: `src/main/resources/db/migrations/V*.sql`

***

## Next Steps

For implementation details, see:

* [TECHNICAL.md](../compiled/technical_documentation/TECHNICAL.md) - Data models, API patterns, error handling
* [MODULES.md](../compiled/technical_documentation/MODULES.md) - Individual module documentation
* [OPERATIONS.md](../compiled/technical_documentation/OPERATIONS.md) - Deployment and monitoring

***

## 2.5 Module Dependencies

The following diagram illustrates the verified dependency graph between the core modules, derived from the codebase structure:

```mermaid
graph TD
    subgraph Employee_Modules [Employee Domain]
        LAM[Leave/Availability Module] --> EM[Employee Module]
        EM --> ERM[Employee Role Module]
    end

    subgraph Scheduling_Modules [Scheduling Domain]
        SM[Schedule Module]
        ShiftM[Shift Module]
        DFM[Demand Forecast Module]
        RM[Rule Module]

        SM --> ShiftM
        SM --> DFM
        SM --> RM
        SM --> DepM
        
        ShiftM --> DepM
        RM --> DepM
    end

    subgraph Core_Modules [Core Domain]
        DepM[Department Module] --> BM[Branch Module]
        BM --> CM[Company Module]
        AuthM[Auth Module]
    end

    subgraph Shared_Layer [Shared Infrastructure]
        Shared[Shared Utilities]
        Config[Configurations]
        Ex[Exceptions]
    end

    %% Cross-layer dependencies
    CM --> Shared
    AuthM --> Shared
    AuthM --> Config
    
    %% Styling
    style SM fill:#d4edda,stroke:#155724
    style AuthM fill:#f8d7da,stroke:#721c24
    style Shared fill:#e2e3e5,stroke:#383d41
    style DepM fill:#fff3cd,stroke:#856404
```

> **Diagram Notes**:
>
> * **Schedule Module**: Orchestrates the scheduling process, pulling data from Shifts, Forecasts, and Rules.
> * **Core Hierarchy**: Enforces the strict `Company -> Branch -> Department` organizational structure.
> * **Shared Layer**: Provides cross-cutting utilities (Exceptions, Constants, AOP Aspects) used by valid feature modules.
