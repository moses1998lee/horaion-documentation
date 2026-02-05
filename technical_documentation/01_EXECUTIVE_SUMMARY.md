# 01 - Executive Summary

> **Horaion Workforce Management Platform - Executive Summary**

---

## 1.1 System Overview

**Horaion** is a comprehensive workforce management platform designed to optimize employee scheduling, manage organizational hierarchies, and streamline workforce operations for multi-location businesses.

{% hint style="info" %}
**Note:** Horaion is built specifically for businesses with complex scheduling needs across multiple locations, where manual scheduling becomes impractical and error-prone.
{% endhint %}

### Core Purpose
Automate complex scheduling operations while maintaining compliance with labor regulations, employee preferences, and business constraints through integration with an external optimization engine.

### System Type
- **Category**: Enterprise Workforce Management System
- **Deployment**: Cloud-based REST API
- **Architecture**: Vertical Slice Architecture with microservice-ready design
- **Integration**: External optimization engine via Feign HTTP client

```mermaid
graph TB
    subgraph "Horaion Platform"
        API[REST API Layer]
        Auth[AWS Cognito Auth]
        DB[(PostgreSQL Database)]
        Engine[Optimization Engine]
    end
    
    subgraph "Client Applications"
        Web[Web Dashboard]
        Mobile[Mobile App]
    end
    
    Web -->|HTTPS/JWT| API
    Mobile -->|HTTPS/JWT| API
    API -->|Validate Token| Auth
    API -->|Data Operations| DB
    API -->|Schedule Generation| Engine
    
    style API fill:#4A90E2,color:#fff
    style Auth fill:#E27D60,color:#fff
    style DB fill:#85CDCA,color:#fff
    style Engine fill:#E8A87C,color:#fff
```

**Diagram Explanation**: This architecture diagram illustrates the core components of the Horaion platform and their interactions:

1. **Client Applications** (Web/Mobile): Users access the system through browser-based dashboards or mobile applications
2. **REST API Layer**: The central hub that processes all requests, enforces business logic, and coordinates between services
3. **AWS Cognito Auth**: Handles user authentication and JWT token validation for secure access
4. **PostgreSQL Database**: Stores all application data with JSONB support for flexible schemas
5. **Optimization Engine**: External service that solves complex scheduling problems using advanced algorithms

---

## 1.2 Key Features

### Organizational Management
- **Multi-tenant Architecture**: Support multiple companies with complete data isolation
- **Hierarchical Structure**: Company → Branch → Department → Employee
- **Role Management**: Flexible employee roles with proficiency tracking

```mermaid
graph TD
    Company[Company<br/>Multi-tenant Root]
    Branch1[Branch: HQ]
    Branch2[Branch: Store A]
    Dept1[Dept: Sales]
    Dept2[Dept: Warehouse]
    Emp1[Employee: John]
    Emp2[Employee: Jane]
    
    Company --> Branch1
    Company --> Branch2
    Branch1 --> Dept1
    Branch1 --> Dept2
    Dept1 --> Emp1
    Dept2 --> Emp2
    
    style Company fill:#2C5F2D,color:#fff
    style Branch1 fill:#97BC62,color:#fff
    style Branch2 fill:#97BC62,color:#fff
    style Dept1 fill:#C9E4CA,color:#000
    style Dept2 fill:#C9E4CA,color:#000
```

**Diagram Explanation**: The organizational hierarchy follows a strict top-down structure:
- **Company** (Tenant): The root entity representing an entire organization
- **Branch**: Physical or logical locations (stores, offices, warehouses)
- **Department**: Functional units within a branch (Sales, Operations, etc.)
- **Employee**: Individual workers assigned to a specific department

{% hint style="warning" %}
**Important:** All data queries automatically filter by the authenticated user's `companyId` to ensure strict tenant isolation. Cross-tenant data access is prevented at the database level.
{% endhint %}

### Employee Management
- **CRUD Operations**: Complete employee lifecycle management
- **AWS Cognito Integration**: Automated user account creation and authentication
- **Excel Import**: Bulk employee upload with validation
- **Custom Fields**: JSONB storage for flexible employee attributes

{% hint style="success" %}
**Tip:** The Excel import feature supports bulk onboarding of hundreds of employees simultaneously, with detailed error reporting for any validation failures.
{% endhint %}

###Schedule Generation
- **Optimization Engine Integration**: External service for complex schedule generation
- **Async Processing**: Long-running operations (15-30 minutes) handled asynchronously
- **Constraint Management**: Rules, shifts, demand forecasts, leave availability
- **Approval Workflow**: PENDING → IN_PROGRESS → APPROVED/REJECTED states

```mermaid
stateDiagram-v2
    [*] --> PENDING: Manager Creates Schedule
    PENDING --> IN_PROGRESS: Sent to Engine
    IN_PROGRESS --> COMPLETED: Engine Returns Result
    IN_PROGRESS --> FAILED: Engine Error
    COMPLETED --> APPROVED: Manager Approves
    COMPLETED --> REJECTED: Manager Rejects
    APPROVED --> [*]: Schedule Published
    REJECTED --> [*]: Discarded
    FAILED --> [*]: Retry Required
```

**Diagram Explanation**: The schedule lifecycle manages the asynchronous generation process:

1. **PENDING**: Initial state when a manager creates a schedule request
2. **IN_PROGRESS**: Schedule request sent to the optimization engine (15-30 min processing)
3. **COMPLETED**: Engine successfully generated a schedule (ready for review)
4. **FAILED**: Engine encountered an error (infeasible constraints, timeout, etc.)
5. **APPROVED**: Manager reviewed and approved the AI-generated schedule
6. **REJECTED**: Manager rejected the schedule (may require manual adjustments)

{% hint style="danger" %}
**Critical:** Schedule generation can take 15-30 minutes. The system uses asynchronous processing to avoid blocking the API. Do not expect immediate results.
{% endhint %}

### Leave Management
- **Leave Tracking**: Employee leave availability and requests
- **Integration**: Leave data feeds into schedule generation

---

## 1.3 Technology Stack Summary

### Backend
- **Runtime**: Java 21 LTS
- **Framework**: Spring Boot 4.0.0
- **Database**: PostgreSQL 12+ with JSONB support
- **Authentication**: AWS Cognito with JWT validation
- **API**: RESTful API (Level 2 maturity)
- **Build Tool**: Maven 3.9+
- **Connection Pool**: HikariCP

{% hint style="info" %}
**Note:** The platform uses Spring Boot 4.0.0, which requires Java 21 LTS. Ensure your development environment is configured accordingly.
{% endhint %}

### Infrastructure
- **Containerization**: Docker
- **Staging**: Digital Ocean (Droplets)
- **Production**: AWS Lightsail
- **Database**: PostgreSQL

### External Integrations
- **Optimization Engine**: Feign HTTP client with 45-minute timeouts
- **Email**: AWS SES (Planned Integration)
- **Messaging**: AWS SQS/SNS (Planned Integration)

{% hint style="warning" %}
**Important:** The Feign client timeout is set to 45 minutes specifically to accommodate the optimization engine's processing time. Do not reduce this value without understanding the implications.
{% endhint %}

### Frontend
- **Framework**: React 19.1.0
- **Build Tool**: Vite 6.x
- **Styling**: Tailwind CSS 4.x
- **Routing**: TanStack Router

---

## 1.4 Target Audience

### User Role Hierarchy

```mermaid
mindmap
  root((Horaion Users))
    Primary Users
      System Admin
        ::icon(fa fa-cogs)
        Org Management
        User Management
      Dept Manager
        ::icon(fa fa-user-tie)
        Schedule Creation
        Approvals
      Employee
        ::icon(fa fa-user)
        View Schedule
        Request Leave
    Secondary Users
      HR Personnel
        ::icon(fa fa-users)
        Onboarding
        Bulk Imports
      Ops Team
        ::icon(fa fa-server)
        Monitoring
        Deployment
```

**Diagram Explanation**: This mindmap illustrates the complete hierarchy of users who interact with the Horaion platform, categorized by their primary function.

**Detailed Breakdown**:
1. **Primary Users (Daily Operations)**:
   - **System Admin**: The super-user responsible for configuring the entire organization (companies, branches, departments) and managing user access
   - **Dept Manager**: The core operational user who creates schedules, manages employees within their department, and approves/rejects AI-generated shifts
   - **Employee**: The end-user who logs in to view their assigned roster, check upcoming shifts, and submit time-off requests
   
2. **Secondary Users (Support & Maintenance)**:
   - **HR Personnel**: Handles administrative tasks like bulk importing employee data via Excel, managing employee onboarding workflows, and maintaining employee records
   - **Ops Team**: Technical staff (DevOps/SRE) who ensure the infrastructure is running smoothly, handle deployments, monitor system health, and manage database backups

#### 1. System Administrators
- **Responsibilities**: Company, branch, department management
- **Access Level**: Full system access across all tenants

#### 2. Department Managers
- **Responsibilities**: Schedule creation, employee management within their department
- **Access Level**: Department-scoped access (can only see their department's data)

#### 3. Employees
- **Responsibilities**: View schedules, submit leave requests
- **Access Level**: Self-service only (can only see their own data)

#### 4. HR Personnel
- **Responsibilities**: Employee onboarding, data management, bulk imports
- ** Access Level**: Company-wide employee access

#### 5. Operations Teams
- **Responsibilities**: System monitoring, deployment, infrastructure maintenance
- **Access Level**: Infrastructure and backend access

{% hint style="success" %}
**Tip:** Role-based access control (RBAC) is enforced at the API level using JWT claims. Each request is validated against the user's assigned role before processing.
{% endhint %}

---

## 1.5 Business Value Proposition

### For Businesses

#### Cost Reduction
- **Automated Scheduling**: Reduces manual scheduling time by 80%
- **Optimization**: Minimizes labor costs while meeting demand
- **Compliance**: Reduces risk of labor law violations

#### Operational Efficiency
- **Multi-location Support**: Manage workforce across multiple branches from a single dashboard
- **Real-time Updates**: Instant schedule changes and notifications
- **Data-driven Decisions**: Analytics on workforce utilization and labor costs

#### Scalability
- **Multi-tenant**: Single deployment serves multiple companies
- **Async Processing**: Handles complex operations without blocking other users
- **Cloud-native**: Scales horizontally with business growth

### For Employees

#### Work-Life Balance
- **Preference Management**: Employee availability and preferences considered in AI scheduling
- **Transparency**: Clear visibility into upcoming schedules weeks in advance
- **Leave Management**: Easy leave request submission with approval workflow

#### Accessibility
- **Self-service**: View schedules without requiring manager intervention
- **Mobile-ready**: RESTful API supports dedicated mobile applications

### For Managers

#### Time Savings
- **Automated Generation**: Schedules created in minutes instead of hours
- **Bulk Operations**: Import hundreds of employees via Excel templates
- **Approval Workflow**: Streamlined schedule review and approval process

#### Flexibility
- **Custom Rules**: Define business-specific scheduling rules (max hours, rest periods, etc.)
- **Constraint Management**: Balance employee preferences with business operational needs
- **Real-time Adjustments**: Quick manual schedule modifications when needed

{% hint style="success" %}
**Success Story:** Pilot deployments show managers save an average of 10 hours per week on scheduling tasks, allowing them to focus on strategic business activities.
{% endhint %}

---

## 1.6 System Metrics

### Current Capabilities
- **Entities**: 12 core business entities
- **Modules**: 12 vertical slice modules
- **API Endpoints**: 50+ RESTful endpoints
- **Database Tables**: 12 main tables + audit/job tracking tables
- **Async Thread Pools**: 2 dedicated executors (schedule & general tasks)
- **External Integrations**: 3 (Cognito, Optimization Engine, SES)

### Performance Characteristics
- **Schedule Generation**: 15-30 minutes (optimization engine-dependent)
- **API Response**: Target <200ms for standard CRUD operations
- **Concurrent Users**: Designed for 1-5 users (Bespoke Phase) scaling to 100+ (SaaS Phase)
- **Database Connections**: 10 max (HikariCP pool)

```mermaid
graph LR
    A[Client Request] -->|<200ms| B[API Response]
    C[Schedule Request] -->|15-30 min| D[Async Completion]
    
    style A fill:#4A90E2,color:#fff
    style B fill:#85CDCA,color:#fff
    style C fill:#E8A87C,color:#fff
    style D fill:#C9E4CA,color:#000
```

**Diagram Explanation**: Performance characteristics vary significantly by operation type:
- **Synchronous Operations** (left): CRUD operations return within 200ms for responsive user experience
- **Asynchronous Operations** (right): Complex schedule generation runs in background for 15-30 minutes

{% hint style="info" %}
**Note:** The 15-30 minute processing time for schedules is inherent to the optimization problem (NP-hard). The system uses async processing to keep the API responsive.
{% endhint %}

---

## 1.7 Document Purpose

This technical documentation provides:
1. **System Architecture**: Design patterns, component relationships, and architectural decisions
2. **Technical Specifications**: Backend implementation details, database schemas, and API contracts
3. **Infrastructure Guide**: Deployment strategies, configuration management, and operational procedures
4. **Development Workflows**: How to build, test, and deploy the application
5. **Module Documentation**: Deep dives into each business domain module

### Intended Audience
- **Software Engineers**: New team members onboarding to the codebase
- **System Architects**: Understanding design decisions and patterns
- **DevOps Engineers**: Deployment, configuration, and operational procedures
- **Technical Managers**: High-level system overview and capabilities

---

## 1.8 Next Steps

{% hint style="success" %}
**Recommended Reading Path:**
1. **System Architecture** → [02_SYSTEM_ARCHITECTURE.md](02_SYSTEM_ARCHITECTURE.md) for architectural patterns and design decisions
2. **Backend Strategy** → [04_BACKEND_SPECIFICATIONS.md](04_BACKEND_SPECIFICATIONS.md) for module structure and implementation details
3. **Security Policy** → [09_SECURITY_REQUIREMENTS.md](09_SECURITY_REQUIREMENTS.md) for authentication and authorization requirements
{% endhint %}

---

*Last Updated: 2026-02-05 | Version: Spring Boot 4.0.0 | Java 21 LTS*
