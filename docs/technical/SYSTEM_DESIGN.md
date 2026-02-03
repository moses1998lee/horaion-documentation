# System Design Specification (SDS)

This document outlines the system design for the Horaion application.

## 1. Architectural Overview

The Horaion application is a **modular monolith** built using the **Spring Boot** framework. It is designed to be deployed as a single service, but it is internally organized into distinct, domain-oriented modules.

This architecture was chosen to balance development velocity and maintainability. It allows for clear separation of concerns and logical organization of code without the operational complexity of a full microservices architecture.

The application leverages several **AWS services** for core functionalities:
- **AWS Cognito:** For user authentication and identity management.
- **AWS SQS/SNS:** For asynchronous messaging and event-driven communication between components.
- **AWS SES:** For sending email notifications.

Authorization is managed by an external service, **Permit.io**, which allows for fine-grained access control policies to be managed independently of the core application code.

## 2. Component Diagram

The following diagram illustrates the high-level components of the system and their interactions.

```mermaid
graph TD
    subgraph "External Users"
        A[Web/Mobile Client]
    end

    subgraph "Horaion Application (Modular Monolith)"
        B[API Gateway / Controllers]
        subgraph "Core Modules"
            Auth[Auth Module]
            Company[Company Module]
            Branch[Branch Module]
            Employee[Employee Module]
            Schedule[Schedule Module]
            Rule[Rule Module]
            Notification[Notification Module]
        end
    end

    subgraph "External Services"
        C[AWS Cognito]
        D[AWS SQS/SNS]
        E[AWS SES]
        F[Permit.io PDP]
        G[PostgreSQL DB]
    end

    A -- HTTPS --> B
    B -- Authenticates with --> C
    B -- Authorizes with --> F
    B -- Interacts with --> Core Modules

    Auth -- Validates JWT --> C
    CoreModules -- Reads/Writes --> G
    CoreModules -- Publish/Subscribe Events --> D
    Notification -- Sends Emails via --> E

    classDef core fill:#D9EAD3,stroke:#333,stroke-width:2px;
    class Auth,Company,Branch,Employee,Schedule,Rule,Notification core;
```

## 3. Module Descriptions

This section provides a brief overview of each module's responsibility within the Horaion application.

- **Auth Module:** Handles user authentication by validating JWTs from AWS Cognito. It is the primary gateway for securing endpoints.
- **Company Module:** Manages the creation and configuration of tenant companies.
- **Branch Module:** Manages company branches, which are physical or logical locations within a company.
- **Department Module:** Manages departments within a branch.
- **Employee Module:** Manages employee data, roles, and their assignments to branches and departments.
- **Schedule Module:** Core module responsible for generating and managing work schedules. It interacts with an external scheduling engine.
- **Rule Module:** Manages the business rules and constraints that govern schedule generation.
- **Demand Forecast Module:** Manages demand forecasts which can be used as input for scheduling.
- **Shift Module:** Manages the individual shifts that make up a schedule.
- **Notification Module:** Handles all outbound notifications (e.g., email, SMS) using AWS services.
- **Shared Module:** Contains cross-cutting concerns such as security configurations, database connections, logging, and common infrastructure code.