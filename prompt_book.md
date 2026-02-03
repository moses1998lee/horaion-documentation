# Documentation Generation Prompts

This guide provides a set of "Task Prompts" you can feed to the agent to generate the specific sections of your System Design Document.

## Prerequisites
Ensure the **Master System Prompt** (Layer 2) has been provided to the agent first to set the persona and context.

---

## Task 1: Context & Architecture (Sections 0-3)
**Goal:** Establish the high-level boundaries, architecture, and scope.

**Prompt to Copy:**
```text
Please generate **Sections 0, 1, 2, and 3** of the System Design Document.

**Key Context from Codebase:**
-   **Backend:** `genesis-api` (Spring Boot, Java 21, PostgreSQL, Hibernate, Flyway).
-   **Frontend:** `genesis-app/genesis-app` (React, Vite, TypeScript, Tailwind/Shadcn likely).
-   **ETL Service:** `genesis-app/genesis-etl-service` (Python, Flask, MongoDB, Pandas).

**Requirements:**
1.  **Architecture Diagram (Mermaid):** clearly show the interaction between the React Client, the Spring Boot API, and the Flask ETL Service.
2.  **System Context:** Define the actors (likely specific user roles defined in `genesis-api` security config) and external systems (AWS Cognito for Auth, S3 likely for uploads).
3.  **Output File:** `01-context-and-architecture.md`
```

---

## Task 2: Backend Engineering (Section 4)
**Goal:** Deep dive into the Spring Boot API.

**Prompt to Copy:**
```text
Please generate **Section 4: Backend Engineering Specifications**.

**Source Material:**
-   Explore `genesis-api/pom.xml` for dependencies.
-   Explore `genesis-api/src/main/java` for structure (Controllers, Services, Repositories).
-   Check `configurations` folder for Security and API setup.

**Requirements:**
1.  **Tech Stack:** Verify Java version, Spring Boot version.
2.  **API Design:** API styling (REST), error handling conventions (`@ControllerAdvice`).
3.  **Security:** Detail the OAuth2 Resource Server setup with Cognito.
4.  **Diagrams:** Sequence diagram of a typical request (e.g., Authenticated Request flow).
5.  **Output File:** `02-backend-specs.md`
```

---

## Task 3: Frontend & ETL Engineering (Section 5 + Extra)
**Goal:** Deep dive into the Client and the ETL Worker.

**Prompt to Copy:**
```text
Please generate **Section 5: Frontend Engineering Specifications** and a new **Section 5.9: ETL Service Specifications**.

**Source Material:**
-   **Frontend:** `genesis-app/genesis-app` (check `package.json`, `vite.config.ts`, `src/components`, `src/pages` or `routes`).
-   **ETL:** `genesis-app/genesis-etl-service` (check `requirements.txt`, `app` folder structure).

**Requirements:**
1.  **Frontend:** Document the Build System (Vite), State Management, and Routing.
2.  **ETL:** Document the Python/Flask architecture, MongoDB usage, and how it processes data (Pandas).
3.  **Output File:** `03-frontend-and-etl-specs.md`
```

---

## Task 4: Data Architecture & Database (Sections 6-7)
**Goal:** Document the data models (SQL & NoSQL).

**Prompt to Copy:**
```text
Please generate **Sections 6 and 7: Data Architecture & Detailed Database Specs**.

**Source Material:**
-   **Postgres:** `genesis-api/src/main/resources/db/migration` (Flyway SQL files) and JPA Entities in `genesis-api/src/main/java/.../domain` or `entity`.
-   **MongoDB:** Check the ETL service for any schematic definitions or implicit schemas in the code.

**Requirements:**
1.  **ER Diagram (Mermaid):** Reverse engineer the ERD from the SQL/JPA entities.
2.  **Schema definitions:** List key tables (e.g., `Employee`, `Users`) and their relationships.
3.  **Data Flow:** Explain how data moves from CSV/Excel upload -> ETL (Mongo) -> Backend (Postgres) if applicable.
4.  **Output File:** `04-data-architecture.md`
```

---

## Task 5: Core Workflows (Section 8)
**Goal:** Document the "Business Logic".

**Prompt to Copy:**
```text
Please generate **Section 8: Core Business Workflows**.

**Source Material:**
-   Look for "Service" classes in `genesis-api` and `genesis-etl-service` that handle complex logic (e.g., `UploadService`, `ProcessingService`).
-   Look for "Excel Upload" related code found in `genesis-api/COGNITO_EXCEL_UPLOAD.md` (if relevant) or the codebase.

**Requirements:**
1.  **Excel Upload Workflow:** Create a comprehensive Sequence Diagram (Mermaid) tracking a file from frontend upload -> ETL processing -> Database storage.
2.  **Validation:** Describe how data validation happens at each stage.
3.  **Output File:** `05-core-workflows.md`
```

---

## Task 6: Ops, Security & Deployment (Sections 9-15)
**Goal:** Document the "Production" aspect.

**Prompt to Copy:**
```text
Please generate **Sections 9 through 15: Security, Infrastructure, Ops, and Appendices**.

**Source Material:**
-   **Infra:** `genesis-api/docker`, `genesis-app/genesis-etl-service/dockerfile`, `docker-compose.yml`.
-   **Security:** Spring Security config, typical CORS settings.
-   **CI/CD:** Check for `.github/workflows` or any build scripts.

**Requirements:**
1.  **Deployment Diagram:** Visualise how the containers (API, App, ETL, DB) are deployed (e.g., Docker Compose or Cloud specifics).
2.  **Env:** Document key Environment Variables found in `.env` or config files.
3.  **Output File:** `06-ops-and-security.md`
```
