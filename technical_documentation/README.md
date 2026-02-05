# Horaion API - Technical Documentation

> **Complete technical specification for the Horaion Workforce Management Platform**

---

## 📋 Table of Contents

### Part 1: Overview & Strategy
- [01 - Executive Summary](01_EXECUTIVE_SUMMARY.md)
  - [1. Introduction](01_EXECUTIVE_SUMMARY.md#1-introduction)
  - [2. Strategic Importance](01_EXECUTIVE_SUMMARY.md#2-strategic-importance)
  - [3. System Overview](01_EXECUTIVE_SUMMARY.md#3-system-overview)
  - [4. Key Capabilities](01_EXECUTIVE_SUMMARY.md#4-key-capabilities)
  - [5. Success Metrics](01_EXECUTIVE_SUMMARY.md#5-success-metrics)
- [02 - System Architecture](02_SYSTEM_ARCHITECTURE.md)
  - [1. High-Level Architecture](02_SYSTEM_ARCHITECTURE.md#1-high-level-architecture)
  - [2. Core Components](02_SYSTEM_ARCHITECTURE.md#2-core-components)
  - [3. Data Flow Patterns](02_SYSTEM_ARCHITECTURE.md#data-flow-patterns)
  - [4. Async Processing](02_SYSTEM_ARCHITECTURE.md#async-processing)
- [03 - Technology Stack](03_TECHNOLOGY_STACK.md)
  - [1. Backend Stack](03_TECHNOLOGY_STACK.md#1-backend-stack)
  - [2. Frontend Stack](03_TECHNOLOGY_STACK.md#2-frontend-stack)
  - [3. Database & Storage](03_TECHNOLOGY_STACK.md#3-database--storage)
  - [4. DevOps & Infrastructure](03_TECHNOLOGY_STACK.md#4-devops--infrastructure)

### Part 2: Technical Specifications
- [04 - Backend Specifications](04_BACKEND_SPECIFICATIONS.md)
  - [Overview](04_BACKEND_SPECIFICATIONS.md#overview)
  - [Module Catalog](04_BACKEND_SPECIFICATIONS.md#module-catalog)
  - [Inter-Module Communication](04_BACKEND_SPECIFICATIONS.md#inter-module-communication)
  - [Best Practices](04_BACKEND_SPECIFICATIONS.md#best-practices)
- [05 - Frontend Specifications](05_FRONTEND_SPECIFICATIONS.md)
  - [1. Runtime Environment](05_FRONTEND_SPECIFICATIONS.md#1-runtime-environment)
  - [2. Core Dependencies](05_FRONTEND_SPECIFICATIONS.md#2-core-dependencies)
  - [3. UI/UX Requirements](05_FRONTEND_SPECIFICATIONS.md#4-uiux-requirements)
- [06 - Database Specifications](06_DATABASE_SPECIFICATIONS.md)
  - [Data Model](06_DATABASE_SPECIFICATIONS.md#data-model)
  - [API Standards](06_DATABASE_SPECIFICATIONS.md#api-standards)
  - [Shared Utilities](06_DATABASE_SPECIFICATIONS.md#shared-utilities)
- [07 - API Design](07_API_DESIGN.md)
  - [REST API Standards](07_API_DESIGN.md#rest-api-standards)
  - [Authentication](07_API_DESIGN.md#authentication)
  - [Common Workflows](07_API_DESIGN.md#common-workflows)

### Part 3: Deep Dives (Shared Kernel)
> **The core reusable components powering all modules.**

- [Overview](16_SHARED_KERNEL/01_OVERVIEW.md)
- [Core](16_SHARED_KERNEL/02_CORE/01_CONFIGURATIONS.md)
- [Infrastructure](16_SHARED_KERNEL/03_INFRASTRUCTURE/01_OVERVIEW.md)
- [Database](16_SHARED_KERNEL/04_DATABASE/01_CONFIGURATIONS.md)
- [Logging](16_SHARED_KERNEL/05_LOGGING/01_ASPECTS.md)
- [Metrics](16_SHARED_KERNEL/06_METRICS/01_CONFIGURATION.md)
- [Providers](16_SHARED_KERNEL/07_PROVIDERS/01_OVERVIEW.md)
- [Security](16_SHARED_KERNEL/08_SECURITY/01_OVERVIEW.md)
- [Global Resources](16_SHARED_KERNEL/09_GLOBAL_RESOURCES/01_OVERVIEW.md)

### Part 4: Modules
> **The business logic vertical slices.**

- [Auth](02_MODULES/auth/01_OVERVIEW.md)
- [Branch](02_MODULES/branch/01_OVERVIEW.md)
- [Company](02_MODULES/company/01_OVERVIEW.md)
- [Demand Forecast](02_MODULES/demandforecast/01_OVERVIEW.md)
- [Department](02_MODULES/department/01_OVERVIEW.md)
- [Employee](02_MODULES/employee/01_OVERVIEW.md)
- [Job Title](02_MODULES/jobtitle/01_OVERVIEW.md)
- [Leave & Availability](02_MODULES/employeeleaveavailability/01_OVERVIEW.md)
- [Me (Context)](02_MODULES/me/01_OVERVIEW.md)
- [Rule](02_MODULES/rule/01_OVERVIEW.md)
- [Schedule](02_MODULES/schedule/01_OVERVIEW.md)
- [Shift](02_MODULES/shift/01_OVERVIEW.md)

### Part 5: Infrastructure & Operations
- [08 - Infrastructure & Deployment](08_INFRASTRUCTURE_DEPLOYMENT.md)
- [09 - Security Requirements](09_SECURITY_REQUIREMENTS.md)
- [10 - Performance Requirements](10_PERFORMANCE_REQUIREMENTS.md)
- [11 - Development Workflow](11_DEVELOPMENT_WORKFLOW.md)
- [12 - Application Flowcharts](12_APPLICATION_FLOWCHARTS.md)
- [13 - Resource Requirements](13_RESOURCE_REQUIREMENTS.md)

---

## 📖 How to Use This Documentation

### For New Developers
1. Start with [01 - Executive Summary](01_EXECUTIVE_SUMMARY.md)
2. Read [02 - System Architecture](02_SYSTEM_ARCHITECTURE.md)
3. Review specific module docs in **Part 4**.

### For Architects
1. [02 - System Architecture](02_SYSTEM_ARCHITECTURE.md)
2. [03 - Technology Stack](03_TECHNOLOGY_STACK.md)
3. [16_SHARED_KERNEL](16_SHARED_KERNEL/01_OVERVIEW.md) (Deep Dive)
