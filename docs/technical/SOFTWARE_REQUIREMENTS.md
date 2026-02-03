# Software Requirements Specification (SRS)

This document outlines the software requirements for the Horaion application.

## 1. User Roles and Characteristics

Based on the application's structure and authorization model, the following user roles are inferred:

- **System Administrator:** Responsible for managing tenant companies and overall system configuration. This role likely has the highest level of permissions.
- **Company Administrator / Manager:** Responsible for managing a single company's data, including branches, departments, employees, and scheduling rules.
- **Employee / Standard User:** A regular user who can view their schedule, manage their availability, and perform other self-service actions.

## 2. Functional Requirements

The functional requirements are derived from the system's API, as exposed by its various modules. The live API documentation (Swagger UI) provides a comprehensive, detailed list of all capabilities. This section provides a high-level summary.

### 2.1. Authentication and User Management
- **FR-AUTH-1:** The system shall authenticate users using credentials managed by AWS Cognito.
- **FR-AUTH-2:** The system shall secure API endpoints, requiring a valid JSON Web Token (JWT) for access.

### 2.2. Company and Organization Structure
- **FR-COMP-1:** The system shall allow for the creation and management of Companies.
- **FR-COMP-2:** The system shall allow for the creation and management of Branches within a Company.
- **FR-COMP-3:** The system shall allow for the creation and management of Departments within a Branch.

### 2.3. Employee Management
- **FR-EMP-1:** The system shall allow for the creation, update, and retrieval of Employee profiles.
- **FR-EMP-2:** The system shall support importing a list of employees from an Excel file.
- **FR-EMP-3:** The system shall allow for assigning Employees to roles (e.g., "Manager", "Associate").
- **FR-EMP-4:** The system shall allow for assigning Employees to one or more Departments.

### 2.4. Scheduling
- **FR-SCHED-1:** The system shall be able to generate a work schedule based on a set of configurable rules and demand forecasts.
- **FR-SCHED-2:** The system shall allow for the creation and management of individual Shifts.
- **FR-SCHED-3:** The system shall allow for the management of scheduling rules and constraints.
- **FR-SCHED-4:** The system shall allow for the management of demand forecasts.

### 2.5. Notifications
- **FR-NOTIF-1:** The system shall be capable of sending notifications (e.g., email) to users.

## 3. Non-Functional Requirements

### 3.1. Security
- **NFR-SEC-1:** All access to the API must be authenticated via OAuth 2.0 and JWTs, validated against the configured AWS Cognito User Pool.
- **NFR-SEC-2:** Fine-grained authorization and access control shall be managed by an external Permit.io Policy Decision Point (PDP). All relevant API calls must be checked against Permit.io policies.
- **NFR-SEC-3:** The application shall not store sensitive user credentials (passwords).

### 3.2. Reliability
- **NFR-REL-1:** The application shall use a connection pool to manage database connections efficiently.
- **NFR-REL-2:** Asynchronous operations, such as sending notifications or processing schedule generation, shall be managed via message queues (AWS SQS) to ensure decoupling and resilience.

### 3.3. Maintainability
- **NFR-MNT-1:** The codebase shall be organized into logical, domain-focused modules.
- **NFR-MNT-2:** Database schema changes shall be managed and versioned through Flyway migrations.
- **NFR-MNT-3:** Code quality and style shall be enforced using Checkstyle, with a minimum line coverage of 80% enforced by JaCoCo.

### 3.4. Usability
- **NFR-USE-1:** The system shall provide a comprehensive, interactive API documentation endpoint (Swagger UI) for developers.