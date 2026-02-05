# Horaion API - Technical Documentation

> **Complete technical specification for the Horaion Workforce Management Platform**

---

## 📋 Complete Table of Contents

### Part 1: Overview & Strategy
- **[01 - Executive Summary](01_EXECUTIVE_SUMMARY.md)**
  - [1. Introduction](01_EXECUTIVE_SUMMARY.md#1-introduction)
  - [2. Strategic Importance](01_EXECUTIVE_SUMMARY.md#2-strategic-importance)
  - [3. System Overview](01_EXECUTIVE_SUMMARY.md#3-system-overview)
  - [4. Key Capabilities](01_EXECUTIVE_SUMMARY.md#4-key-capabilities)
  - [5. Success Metrics](01_EXECUTIVE_SUMMARY.md#5-success-metrics)
  
- **[02 - System Architecture](02_SYSTEM_ARCHITECTURE.md)**
  - [1. High-Level Architecture](02_SYSTEM_ARCHITECTURE.md#1-high-level-architecture)
  - [2. Core Components](02_SYSTEM_ARCHITECTURE.md#2-core-components)
  - [3. Data Flow Patterns](02_SYSTEM_ARCHITECTURE.md#data-flow-patterns)
  - [4. Async Processing](02_SYSTEM_ARCHITECTURE.md#async-processing)
  
- **[03 - Technology Stack](03_TECHNOLOGY_STACK.md)**
  - [1. Backend Stack](03_TECHNOLOGY_STACK.md#1-backend-stack)
  - [2. Frontend Stack](03_TECHNOLOGY_STACK.md#2-frontend-stack)
  - [3. Database & Storage](03_TECHNOLOGY_STACK.md#3-database--storage)
  - [4. DevOps & Infrastructure](03_TECHNOLOGY_STACK.md#4-devops--infrastructure)

---

### Part 2: Technical Specifications

- **[04 - Backend Strategy](04_BACKEND_SPECIFICATIONS.md)**
  - [Overview](04_BACKEND_SPECIFICATIONS.md#overview)
  - [Module Catalog](04_BACKEND_SPECIFICATIONS.md#module-catalog)
  - [Inter-Module Communication](04_BACKEND_SPECIFICATIONS.md#inter-module-communication)
  - [Best Practices](04_BACKEND_SPECIFICATIONS.md#best-practices)

- **[05 - Frontend Strategy](05_FRONTEND_SPECIFICATIONS.md)**
  - [1. Runtime Environment](05_FRONTEND_SPECIFICATIONS.md#1-runtime-environment)
  - [2. Core Dependencies](05_FRONTEND_SPECIFICATIONS.md#2-core-dependencies)
  - [3. UI/UX Requirements](05_FRONTEND_SPECIFICATIONS.md#4-uiux-requirements)

- **[06 - Database Strategy](06_DATABASE_SPECIFICATIONS.md)**
  - [6.1 Strategy Overview](06_DATABASE_SPECIFICATIONS.md#61-strategy-overview)
  - [6.2 Key Data Patterns](06_DATABASE_SPECIFICATIONS.md#62-key-data-patterns)
  - [6.3 Module Schemas](06_DATABASE_SPECIFICATIONS.md#63-module-schemas)

- **[07 - API Design](07_API_DESIGN.md)**
  - [REST API Standards](07_API_DESIGN.md#rest-api-standards)
  - [Authentication](07_API_DESIGN.md#authentication)
  - [Common Workflows](07_API_DESIGN.md#common-workflows)

---

### Part 3: Infrastructure & Operations

- **[08 - Infrastructure Strategy](08_INFRASTRUCTURE_DEPLOYMENT.md)**
  - [Deployment Strategy](08_INFRASTRUCTURE_DEPLOYMENT.md#deployment-strategy)
  - [Configuration](08_INFRASTRUCTURE_DEPLOYMENT.md#configuration-strategy)
  - [Database Management](08_INFRASTRUCTURE_DEPLOYMENT.md#database-management)
  - [Monitoring & Health Checks](08_INFRASTRUCTURE_DEPLOYMENT.md#monitoring--health-checks)

- **[09 - Security Policy](09_SECURITY_REQUIREMENTS.md)**
  - [9.1 Authentication Policy](09_SECURITY_REQUIREMENTS.md#91-authentication-policy)
  - [9.2 Authorization Policy](09_SECURITY_REQUIREMENTS.md#92-authorization-policy)
  - [9.3 Data Protection](09_SECURITY_REQUIREMENTS.md#93-data-protection)
  - [9.4 Network Security](09_SECURITY_REQUIREMENTS.md#94-network-security)

- **[10 - Performance Strategy](10_PERFORMANCE_REQUIREMENTS.md)**
- **[11 - Development Workflow](11_DEVELOPMENT_WORKFLOW.md)**
- **[12 - Application Flowcharts](12_APPLICATION_FLOWCHARTS.md)**
- **[13 - Resource Requirements](13_RESOURCE_REQUIREMENTS.md)**

---

### Part 4: Modules
> **Business logic vertical slices - each module follows the pattern: Overview → API → Domain → Config**

#### **[Auth Module](02_MODULES/auth/01_OVERVIEW.md)**
- [Overview](02_MODULES/auth/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/auth/02_API.md)
- [Domain Logic](02_MODULES/auth/03_DOMAIN.md)
- [Configuration](02_MODULES/auth/04_CONFIG.md)

#### **[Branch Module](02_MODULES/branch/01_OVERVIEW.md)**
- [Overview](02_MODULES/branch/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/branch/02_API.md)
- [Domain Logic](02_MODULES/branch/03_DOMAIN.md)
- [Configuration](02_MODULES/branch/04_CONFIG.md)

#### **[Company Module](02_MODULES/company/01_OVERVIEW.md)**
- [Overview](02_MODULES/company/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/company/02_API.md)
- [Domain Logic](02_MODULES/company/03_DOMAIN.md)
- [Configuration](02_MODULES/company/04_CONFIG.md)

#### **[Demand Forecast Module](02_MODULES/demandforecast/01_OVERVIEW.md)**
- [Overview](02_MODULES/demandforecast/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/demandforecast/02_API.md)
- [Domain Logic](02_MODULES/demandforecast/03_DOMAIN.md)
- [Configuration](02_MODULES/demandforecast/04_CONFIG.md)

#### **[Department Module](02_MODULES/department/01_OVERVIEW.md)**
- [Overview](02_MODULES/department/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/department/02_API.md)
- [Domain Logic](02_MODULES/department/03_DOMAIN.md)
- [Configuration](02_MODULES/department/04_CONFIG.md)

#### **[Employee Module](02_MODULES/employee/01_OVERVIEW.md)**
- [Overview](02_MODULES/employee/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/employee/02_API.md)
- [Domain Logic](02_MODULES/employee/03_DOMAIN.md) *(includes Cognito Workflow)*
- [Configuration](02_MODULES/employee/04_CONFIG.md)

#### **[Employee Leave & Availability Module](02_MODULES/employeeleaveavailability/01_OVERVIEW.md)**
- [Overview](02_MODULES/employeeleaveavailability/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/employeeleaveavailability/02_API.md)
- [Domain Logic](02_MODULES/employeeleaveavailability/03_DOMAIN.md)
- [Configuration](02_MODULES/employeeleaveavailability/04_CONFIG.md)

#### **[Job Title Module](02_MODULES/jobtitle/01_OVERVIEW.md)**
- [Overview](02_MODULES/jobtitle/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/jobtitle/02_API.md)
- [Domain Logic](02_MODULES/jobtitle/03_DOMAIN.md)
- [Configuration](02_MODULES/jobtitle/04_CONFIG.md)

#### **[Me (User Context) Module](02_MODULES/me/01_OVERVIEW.md)**
- [Overview](02_MODULES/me/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/me/02_API.md)
- [Domain Logic](02_MODULES/me/03_DOMAIN.md)
- [Configuration](02_MODULES/me/04_CONFIG.md)

#### **[Rule Module](02_MODULES/rule/01_OVERVIEW.md)**
- [Overview](02_MODULES/rule/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/rule/02_API.md)
- [Domain Logic](02_MODULES/rule/03_DOMAIN.md)
- [Configuration](02_MODULES/rule/04_CONFIG.md)

#### **[Schedule Module](02_MODULES/schedule/01_OVERVIEW.md)**
- [Overview](02_MODULES/schedule/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/schedule/02_API.md)
- [Domain Logic](02_MODULES/schedule/03_DOMAIN.md)
- [Configuration](02_MODULES/schedule/04_CONFIG.md)

#### **[Shift Module](02_MODULES/shift/01_OVERVIEW.md)**
- [Overview](02_MODULES/shift/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/shift/02_API.md)
- [Domain Logic](02_MODULES/shift/03_DOMAIN.md)
- [Configuration](02_MODULES/shift/04_CONFIG.md)

---

### Part 5: Shared Kernel
> **Core reusable components powering all modules**

#### **[Shared Kernel Overview](16_SHARED_KERNEL/01_OVERVIEW.md)**

#### **[02 - Core](16_SHARED_KERNEL/02_CORE/01_CONFIGURATIONS.md)**
- [Configurations](16_SHARED_KERNEL/02_CORE/01_CONFIGURATIONS.md) - Java @Configuration classes
- [Constants](16_SHARED_KERNEL/02_CORE/02_CONSTANTS.md) - Application-wide constants
- [Enums](16_SHARED_KERNEL/02_CORE/03_ENUMS.md) - Shared enumerations
- [Exceptions](16_SHARED_KERNEL/02_CORE/04_EXCEPTIONS.md) - Custom exception hierarchy
- [Helpers](16_SHARED_KERNEL/02_CORE/05_HELPERS.md) - Utility helper classes
- [Interfaces](16_SHARED_KERNEL/02_CORE/06_INTERFACES.md) - Common interfaces
- [Repositories](16_SHARED_KERNEL/02_CORE/07_REPOSITORIES.md) - Base repository patterns
- [Responses](16_SHARED_KERNEL/02_CORE/08_RESPONSES.md) - Standard API response DTOs
- [Utils](16_SHARED_KERNEL/02_CORE/09_UTILS.md) - Utility functions

#### **[03 - Infrastructure](16_SHARED_KERNEL/03_INFRASTRUCTURE/01_AWS.md)**
- [AWS Integration](16_SHARED_KERNEL/03_INFRASTRUCTURE/01_AWS.md) - Cognito, SNS, SQS
- [Engine Client](16_SHARED_KERNEL/03_INFRASTRUCTURE/02_ENGINE.md) - Optimization engine communication
- [Excel Processing](16_SHARED_KERNEL/03_INFRASTRUCTURE/03_EXCEL.md) - Apache POI utilities
- [Genesis](16_SHARED_KERNEL/03_INFRASTRUCTURE/04_GENESIS.md) - Project Genesis integration
- [Messaging](16_SHARED_KERNEL/03_INFRASTRUCTURE/05_MESSAGING.md) - Message queue integration
- [Notification](16_SHARED_KERNEL/03_INFRASTRUCTURE/06_NOTIFICATION.md) - Email/SMS notifications
- [Properties](16_SHARED_KERNEL/03_INFRASTRUCTURE/07_PROPERTIES.md) - Configuration properties

#### **[04 - Database](16_SHARED_KERNEL/04_DATABASE/01_CONFIGURATIONS.md)**
- [Configurations](16_SHARED_KERNEL/04_DATABASE/01_CONFIGURATIONS.md) - HikariCP setup
- [Constants](16_SHARED_KERNEL/04_DATABASE/02_CONSTANTS.md) - Database constants
- [Enums](16_SHARED_KERNEL/04_DATABASE/03_ENUMS.md) - Database-related enums
- [Properties](16_SHARED_KERNEL/04_DATABASE/04_PROPERTIES.md) - Database properties

#### **[05 - Logging](16_SHARED_KERNEL/05_LOGGING/01_ASPECTS.md)**
- [Aspects](16_SHARED_KERNEL/05_LOGGING/01_ASPECTS.md) - AOP logging interceptors
- [Constants](16_SHARED_KERNEL/05_LOGGING/02_CONSTANTS.md) - Log message constants
- [Exceptions](16_SHARED_KERNEL/05_LOGGING/03_EXCEPTIONS.md) - Logging exceptions
- [Properties](16_SHARED_KERNEL/05_LOGGING/04_PROPERTIES.md) - Logback configuration

#### **[06 - Metrics](16_SHARED_KERNEL/06_METRICS/01_CONFIGURATION.md)**
- [Configuration](16_SHARED_KERNEL/06_METRICS/01_CONFIGURATION.md) - Micrometer setup

#### **[07 - Providers](16_SHARED_KERNEL/07_PROVIDERS/01_OVERVIEW.md)**
- [Overview](16_SHARED_KERNEL/07_PROVIDERS/01_OVERVIEW.md) - Provider pattern explanation
- [Interface](16_SHARED_KERNEL/07_PROVIDERS/02_INTERFACE.md) - Provider interfaces
- [Implementations](16_SHARED_KERNEL/07_PROVIDERS/03_IMPLEMENTATIONS.md) - Concrete providers

#### **[08 - Security](16_SHARED_KERNEL/08_SECURITY/01_OVERVIEW.md)**
- [Overview](16_SHARED_KERNEL/08_SECURITY/01_OVERVIEW.md) - Security architecture
- [Authentication](16_SHARED_KERNEL/08_SECURITY/02_AUTHENTICATION.md) - JWT validation
- [Authorization](16_SHARED_KERNEL/08_SECURITY/03_AUTHORIZATION.md) - Role-based access
- [Context](16_SHARED_KERNEL/08_SECURITY/04_CONTEXT.md) - Security context management

#### **[09 - Global Resources](16_SHARED_KERNEL/09_GLOBAL_RESOURCES/01_OVERVIEW.md)**
- [Overview](16_SHARED_KERNEL/09_GLOBAL_RESOURCES/01_OVERVIEW.md) - Resource mapping
- [Properties](16_SHARED_KERNEL/09_GLOBAL_RESOURCES/02_PROPERTIES.md) - application.yaml reference
- [Migrations](16_SHARED_KERNEL/09_GLOBAL_RESOURCES/03_MIGRATIONS.md) - Flyway database migrations
- [Templates](16_SHARED_KERNEL/09_GLOBAL_RESOURCES/04_TEMPLATES.md) - Static assets (Excel templates)

---

## 📖 How to Use This Documentation

### For New Developers
1. **Start Here**: [01 - Executive Summary](01_EXECUTIVE_SUMMARY.md) for the big picture
2. **Understand Structure**: [02 - System Architecture](02_SYSTEM_ARCHITECTURE.md) for technical overview
3. **Pick a Module**: Choose a business domain from **Part 4: Modules** and read its Overview
4. **Dive Deep**: Explore the module's API, Domain Logic, and Configuration files

### For Backend Engineers
1. **Architecture First**: [02 - System Architecture](02_SYSTEM_ARCHITECTURE.md)
2. **Module Catalog**: [04 - Backend Strategy](04_BACKEND_SPECIFICATIONS.md)
3. **Shared Patterns**: Browse [16_SHARED_KERNEL](16_SHARED_KERNEL/01_OVERVIEW.md) for reusable components
4. **Specific Module**: Jump to the module you're working on in **Part 4**

### For DevOps/SRE
1. **Infrastructure**: [08 - Infrastructure Strategy](08_INFRASTRUCTURE_DEPLOYMENT.md)
2. **Database Migrations**: [Global Resources - Migrations](16_SHARED_KERNEL/09_GLOBAL_RESOURCES/03_MIGRATIONS.md)
3. **Configuration**: [Global Resources - Properties](16_SHARED_KERNEL/09_GLOBAL_RESOURCES/02_PROPERTIES.md)
4. **Monitoring**: [Shared Kernel - Metrics](16_SHARED_KERNEL/06_METRICS/01_CONFIGURATION.md)

### For Security Auditors
1. **Security Policy**: [09 - Security Policy](09_SECURITY_REQUIREMENTS.md)
2. **Implementation**: [Shared Kernel - Security](16_SHARED_KERNEL/08_SECURITY/01_OVERVIEW.md)
3. **Authentication Flow**: [Security - Authentication](16_SHARED_KERNEL/08_SECURITY/02_AUTHENTICATION.md)
4. **Data Protection**: [Database Strategy](06_DATABASE_SPECIFICATIONS.md)

### For Architects
1. **System Design**: [02 - System Architecture](02_SYSTEM_ARCHITECTURE.md)
2. **Technology Choices**: [03 - Technology Stack](03_TECHNOLOGY_STACK.md)
3. **Module Structure**: [04 - Backend Strategy](04_BACKEND_SPECIFICATIONS.md)
4. **Shared Kernel**: [16_SHARED_KERNEL](16_SHARED_KERNEL/01_OVERVIEW.md) for cross-cutting concerns

---

## 📂 Documentation Structure

```
technical_documentation/
├── 01-13: Strategy & Specifications (High-Level)
├── 02_MODULES/: Business Logic (12 Modules)
│   └── {module}/
│       ├── 01_OVERVIEW.md
│       ├── 02_API.md
│       ├── 03_DOMAIN.md
│       └── 04_CONFIG.md
└── 16_SHARED_KERNEL/: Reusable Components (9 Sections)
    ├── 01_OVERVIEW.md
    ├── 02_CORE/: Configurations, Exceptions, DTOs
    ├── 03_INFRASTRUCTURE/: AWS, Engine, Excel
    ├── 04_DATABASE/: HikariCP, Enums
    ├── 05_LOGGING/: AOP Aspects, Logback
    ├── 06_METRICS/: Micrometer
    ├── 07_PROVIDERS/: Context Providers
    ├── 08_SECURITY/: JWT, RBAC
    └── 09_GLOBAL_RESOURCES/: YAML, Flyway, Templates
```

---

## 🔍 Quick Links

**Most Frequently Accessed**:
- [Employee Module](02_MODULES/employee/01_OVERVIEW.md) - Core HR functionality
- [Schedule Module](02_MODULES/schedule/01_OVERVIEW.md) - AI-powered scheduling
- [Security Overview](16_SHARED_KERNEL/08_SECURITY/01_OVERVIEW.md) - Authentication & Authorization
- [Global Properties](16_SHARED_KERNEL/09_GLOBAL_RESOURCES/02_PROPERTIES.md) - Configuration reference

**For Troubleshooting**:
- [Database Migrations](16_SHARED_KERNEL/09_GLOBAL_RESOURCES/03_MIGRATIONS.md) - Schema version history
- [Logging Configuration](16_SHARED_KERNEL/05_LOGGING/01_ASPECTS.md) - Debug logging setup
- [Infrastructure Deployment](08_INFRASTRUCTURE_DEPLOYMENT.md) - Deployment guide

---

*This documentation follows the "Gold Standard": detailed, newbie-friendly, and professionally structured. Each section is designed to be self-contained while linking to related topics for deeper exploration.*
