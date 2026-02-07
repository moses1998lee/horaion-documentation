# Horaion API - Technical Documentation

> **Complete technical specification for the Horaion Workforce Management Platform**

---

## 📋 Complete Table of Contents

### Part 1: Overview & Strategy
- **[01 - Executive Summary](01_EXECUTIVE_SUMMARY.md)**
  - [1.1 System Overview](01_EXECUTIVE_SUMMARY.md#1-1-system-overview)
  - [1.2 Key Features](01_EXECUTIVE_SUMMARY.md#1-2-key-features)
  - [1.3 Technology Stack Summary](01_EXECUTIVE_SUMMARY.md#1-3-technology-stack-summary)
  - [1.4 Target Audience](01_EXECUTIVE_SUMMARY.md#1-4-target-audience)
  - [1.5 Business Value Proposition](01_EXECUTIVE_SUMMARY.md#1-5-business-value-proposition)
  - [1.6 System Metrics](01_EXECUTIVE_SUMMARY.md#1-6-system-metrics)
  - [1.7 Document Purpose](01_EXECUTIVE_SUMMARY.md#1-7-document-purpose)
  - [1.8 Next Steps](01_EXECUTIVE_SUMMARY.md#1-8-next-steps)

- **[02 - System Architecture](02_SYSTEM_ARCHITECTURE.md)**
  - [Overview](02_SYSTEM_ARCHITECTURE.md#overview)
  - [System Context](02_SYSTEM_ARCHITECTURE.md#system-context)
  - [Application Architecture](02_SYSTEM_ARCHITECTURE.md#application-architecture)
  - [Security Architecture](02_SYSTEM_ARCHITECTURE.md#security-architecture)
  - [Data Flow Patterns](02_SYSTEM_ARCHITECTURE.md#data-flow-patterns)
  - [Async Processing](02_SYSTEM_ARCHITECTURE.md#async-processing)
  - [Architectural Decisions](02_SYSTEM_ARCHITECTURE.md#architectural-decisions)
  - [2.5 Module Dependencies](02_SYSTEM_ARCHITECTURE.md#2-5-module-dependencies)

- **[03 - Technology Stack](03_TECHNOLOGY_STACK.md)**
  - [System Stack Overview](03_TECHNOLOGY_STACK.md#system-stack-overview)
  - [Backend Technology Stack](03_TECHNOLOGY_STACK.md#backend-technology-stack)
  - [Database Technology](03_TECHNOLOGY_STACK.md#database-technology)
  - [Infrastructure & Deployment](03_TECHNOLOGY_STACK.md#infrastructure--deployment)
  - [Frontend Technology Stack](03_TECHNOLOGY_STACK.md#frontend-technology-stack)
  - [Development Tools](03_TECHNOLOGY_STACK.md#development-tools)
  - [Monitoring & Observability](03_TECHNOLOGY_STACK.md#monitoring--observability)
  - [External Integrations](03_TECHNOLOGY_STACK.md#external-integrations)
  - [Technology Decisions](03_TECHNOLOGY_STACK.md#technology-decisions)
  - [Version Matrix](03_TECHNOLOGY_STACK.md#version-matrix)

---

### Part 2: Technical Specifications

- **[04 - Backend Strategy](04_BACKEND_SPECIFICATIONS.md)**
  - [Overview](04_BACKEND_SPECIFICATIONS.md#overview)
  - [Module Catalog](04_BACKEND_SPECIFICATIONS.md#module-catalog)
  - [8. Layered Architecture](04_BACKEND_SPECIFICATIONS.md#8-layered-architecture)
  - [Inter-Module Communication](04_BACKEND_SPECIFICATIONS.md#inter-module-communication)
  - [Best Practices](04_BACKEND_SPECIFICATIONS.md#best-practices)
  - [Advanced Module Patterns](04_BACKEND_SPECIFICATIONS.md#advanced-module-patterns)
  - [Module-Specific Patterns](04_BACKEND_SPECIFICATIONS.md#module-specific-patterns)

- **[05 - Frontend Strategy](05_FRONTEND_SPECIFICATIONS.md)**
  - [1. Runtime Environment](05_FRONTEND_SPECIFICATIONS.md#1-runtime-environment)
  - [2. Core Dependencies](05_FRONTEND_SPECIFICATIONS.md#2-core-dependencies)
  - [3. UI/UX Stack](05_FRONTEND_SPECIFICATIONS.md#3-uiux-stack)
  - [4. Forms & Validation](05_FRONTEND_SPECIFICATIONS.md#4-forms--validation)
  - [5. Development Dependencies](05_FRONTEND_SPECIFICATIONS.md#5-development-dependencies)
  - [6. Utilities](05_FRONTEND_SPECIFICATIONS.md#6-utilities)
  - [7. Deployment](05_FRONTEND_SPECIFICATIONS.md#7-deployment)
  - [8. Directory Structure](05_FRONTEND_SPECIFICATIONS.md#8-directory-structure)
  - [9. Environment Configuration](05_FRONTEND_SPECIFICATIONS.md#9-environment-configuration)
  - [10. API Integration](05_FRONTEND_SPECIFICATIONS.md#10-api-integration)

- **[06 - Database Strategy](06_DATABASE_SPECIFICATIONS.md)**
  - [6.1 Strategy Overview](06_DATABASE_SPECIFICATIONS.md#6-1-strategy-overview)
  - [6.2 Key Data Patterns](06_DATABASE_SPECIFICATIONS.md#6-2-key-data-patterns)
  - [6.3 Module Schemas](06_DATABASE_SPECIFICATIONS.md#6-3-module-schemas)
  - [6.4 Shared Tables](06_DATABASE_SPECIFICATIONS.md#6-4-shared-tables)

- **[07 - API Design](07_API_DESIGN.md)**
  - [Authentication Examples](07_API_DESIGN.md#authentication-examples)
  - [Employee Management](07_API_DESIGN.md#employee-management)
  - [Schedule Generation](07_API_DESIGN.md#schedule-generation)
  - [Error Handling](07_API_DESIGN.md#error-handling)
  - [Common Workflows](07_API_DESIGN.md#common-workflows)
  - [Tips for Developers](07_API_DESIGN.md#tips-for-developers)

---

### Part 3: Infrastructure & Operations

- **[08 - Infrastructure Strategy](08_INFRASTRUCTURE_DEPLOYMENT.md)**
  - [Deployment Strategy](08_INFRASTRUCTURE_DEPLOYMENT.md#deployment-strategy)
  - [Deployment](08_INFRASTRUCTURE_DEPLOYMENT.md#deployment)
  - [Configuration Strategy](08_INFRASTRUCTURE_DEPLOYMENT.md#configuration-strategy)
  - [Database Management](08_INFRASTRUCTURE_DEPLOYMENT.md#database-management)
  - [Monitoring & Health Checks](08_INFRASTRUCTURE_DEPLOYMENT.md#monitoring--health-checks)
  - [Troubleshooting](08_INFRASTRUCTURE_DEPLOYMENT.md#troubleshooting)
  - [Performance Tuning](08_INFRASTRUCTURE_DEPLOYMENT.md#performance-tuning)
  - [Security Best Practices](08_INFRASTRUCTURE_DEPLOYMENT.md#security-best-practices)

- **[09 - Security Policy](09_SECURITY_REQUIREMENTS.md)**
  - [9.1 Authentication Policy](09_SECURITY_REQUIREMENTS.md#9-1-authentication-policy)
  - [9.2 Authorization Policy](09_SECURITY_REQUIREMENTS.md#9-2-authorization-policy)
  - [9.3 Data Protection](09_SECURITY_REQUIREMENTS.md#9-3-data-protection)
  - [9.4 Network Security](09_SECURITY_REQUIREMENTS.md#9-4-network-security)

- **[10 - Performance Strategy](10_PERFORMANCE_REQUIREMENTS.md)**
  - [13.1 Response Time](10_PERFORMANCE_REQUIREMENTS.md#13-1-response-time)
  - [13.2 Scalability](10_PERFORMANCE_REQUIREMENTS.md#13-2-scalability)
  - [13.3 Monitoring & Logging](10_PERFORMANCE_REQUIREMENTS.md#13-3-monitoring-logging)
  - [Performance Optimization Checklist](10_PERFORMANCE_REQUIREMENTS.md#performance-optimization-checklist)
  - [Performance Testing Plan](10_PERFORMANCE_REQUIREMENTS.md#performance-testing-plan)
  
- **[11 - Development Workflow](11_DEVELOPMENT_WORKFLOW.md)**
  - [14.1 Development Environment](11_DEVELOPMENT_WORKFLOW.md#14-1-development-environment)
  - [14.2 CI/CD Pipeline](11_DEVELOPMENT_WORKFLOW.md#14-2-cicd-pipeline)
  - [Testing Strategy](11_DEVELOPMENT_WORKFLOW.md#testing-strategy)
  - [Code Quality](11_DEVELOPMENT_WORKFLOW.md#code-quality)

- **[12 - Application Flowcharts](12_APPLICATION_FLOWCHARTS.md)**
  - [Required Flowcharts](12_APPLICATION_FLOWCHARTS.md#required-flowcharts)
  - [Additional Workflows (Optional)](12_APPLICATION_FLOWCHARTS.md#additional-workflows-optional)

- **[13 - Resource Requirements](13_RESOURCE_REQUIREMENTS.md)**

---

### Part 4: Modules
> **Business logic vertical slices - each module follows the pattern: Overview → API → Domain → Config**

#### **[Auth Module](02_MODULES/auth/01_OVERVIEW.md)**
- [Overview](02_MODULES/auth/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/auth/02_API.md)
- [Domain Logic](02_MODULES/auth/03_DOMAIN.md)
- [Configuration](02_MODULES/auth/04_CONFIGURATION.md)

#### **[Branch Module](02_MODULES/branch/01_OVERVIEW.md)**
- [Overview](02_MODULES/branch/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/branch/02_API.md)
- [Domain Logic](02_MODULES/branch/03_DOMAIN.md)
- [Configuration](02_MODULES/branch/04_CONFIGURATION.md)
    
#### **[Company Module](02_MODULES/company/01_OVERVIEW.md)**
- [Overview](02_MODULES/company/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/company/02_API.md)
- [Domain Logic](02_MODULES/company/03_DOMAIN.md)
- [Configuration](02_MODULES/company/04_CONFIGURATION.md)

#### **[Demand Forecast Module](02_MODULES/demandforecast/01_OVERVIEW.md)**
- [Overview](02_MODULES/demandforecast/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/demandforecast/02_API.md)
- [Domain Logic](02_MODULES/demandforecast/03_DOMAIN.md)
- [Configuration](02_MODULES/demandforecast/04_CONFIGURATION.md)

#### **[Department Module](02_MODULES/department/01_OVERVIEW.md)**
- [Overview](02_MODULES/department/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/department/02_API.md)
- [Domain Logic](02_MODULES/department/03_DOMAIN.md)
- [Configuration](02_MODULES/department/04_CONFIGURATION.md)

#### **[Employee Module](02_MODULES/employee/01_OVERVIEW.md)**
- [Overview](02_MODULES/employee/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/employee/02_API.md)
- [Domain Logic](02_MODULES/employee/03_DOMAIN.md) *(includes Cognito Workflow)*
- [Configuration](02_MODULES/employee/04_CONFIGURATION.md)

#### **[Employee Leave & Availability Module](02_MODULES/employeeleaveavailability/01_OVERVIEW.md)**
- [Overview](02_MODULES/employeeleaveavailability/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/employeeleaveavailability/02_API.md)
- [Domain Logic](02_MODULES/employeeleaveavailability/03_DOMAIN.md)
- [Configuration](02_MODULES/employeeleaveavailability/04_CONFIGURATION.md)

#### **[Job Title Module](02_MODULES/jobtitle/01_OVERVIEW.md)**
- [Overview](02_MODULES/jobtitle/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/jobtitle/02_API.md)
- [Domain Logic](02_MODULES/jobtitle/03_DOMAIN.md)
- [Configuration](02_MODULES/jobtitle/04_CONFIGURATION.md)

#### **[Me (User Context) Module](02_MODULES/me/01_OVERVIEW.md)**
- [Overview](02_MODULES/me/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/me/02_API.md)
- [Domain Logic](02_MODULES/me/03_DOMAIN.md)
- [Configuration](02_MODULES/me/04_CONFIGURATION.md)

#### **[Rule Module](02_MODULES/rule/01_OVERVIEW.md)**
- [Overview](02_MODULES/rule/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/rule/02_API.md)
- [Domain Logic](02_MODULES/rule/03_DOMAIN.md)
- [Configuration](02_MODULES/rule/04_CONFIGURATION.md)

#### **[Schedule Module](02_MODULES/schedule/01_OVERVIEW.md)**
- [Overview](02_MODULES/schedule/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/schedule/02_API.md)
- [Domain Logic](02_MODULES/schedule/03_DOMAIN.md)
- [Configuration](02_MODULES/schedule/04_CONFIGURATION.md)

#### **[Shift Module](02_MODULES/shift/01_OVERVIEW.md)**
- [Overview](02_MODULES/shift/01_OVERVIEW.md)
- [API Endpoints](02_MODULES/shift/02_API.md)
- [Domain Logic](02_MODULES/shift/03_DOMAIN.md)
- [Configuration](02_MODULES/shift/04_CONFIGURATION.md)

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
5. **Don't Reinvent the Wheel**: Check **[Shared Kernel](16_SHARED_KERNEL/01_OVERVIEW.md)** for reusable core components (Security, DTOs, Utils) used across the entire application.

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
