# Learning Path

Suggested reading order and learning paths for different roles working with the Genesis platform.

---

## For New Developers (First Time with Genesis)

### Week 1: Understanding the Platform

**Day 1-2: Overview & Setup**
1. Read [README.md](README.md) - Understand what Genesis is and its capabilities
2. Read [GLOSSARY.md](GLOSSARY.md) - Familiarize yourself with technical terms
3. Follow [OPERATIONS.md - Local Development](OPERATIONS.md#local-development) - Set up your local environment
4. Verify setup with health check: `curl http://localhost:8080/actuator/health`

**Day 3-4: Architecture & Design**
1. Read [ARCHITECTURE.md](ARCHITECTURE.md) - Understand the system design
2. Focus on:
   - Vertical Slice Architecture
   - Authentication Flow
   - Data Flow Patterns
3. Review the 5 ADRs to understand key decisions

**Day 5: Data Model**
1. Read [TECHNICAL.md - Data Model](TECHNICAL.md#data-model) - Study the ERD
2. Understand the organizational hierarchy: Company → Branch → Department → Employee
3. Review core entities: Employee, Schedule, Shift
4. Explore the database using `psql`:
   ```bash
   psql -h localhost -U genesis_user -d genesis
   \dt  # List tables
   \d employees  # Describe employees table
   ```

### Week 2: Hands-On Development

**Day 1-2: API Exploration**
1. Read [EXAMPLES.md - Authentication Examples](EXAMPLES.md#authentication-examples)
2. Practice the authentication flow:
   - Register a user
   - Confirm email
   - Login and get JWT token
3. Make authenticated requests to existing endpoints

**Day 3-4: Module Deep Dive**
1. Read [MODULES.md](MODULES.md) - Understand all 12 modules
2. Pick one module (start with Employee) and explore:
   - Controller: `EmployeeController.java`
   - Service: `EmployeeService.java`
   - Repository: `EmployeeRepository.java`
   - Entity: `Employee.java`
   - DTOs: `EmployeeRequestDTO.java`, `EmployeeResponseDTO.java`
3. Trace a request from controller to database and back

**Day 5: Make Your First Change**
1. Add a new field to an existing entity (e.g., `middleName` to Employee)
2. Create a Flyway migration
3. Update the entity, DTOs, and mapper
4. Test the change locally

### Week 3: Advanced Topics

**Day 1-2: Async Processing**
1. Read [ARCHITECTURE.md - Async Processing](ARCHITECTURE.md#async-processing)
2. Study the schedule generation flow
3. Understand `@Async` and webhook callbacks

**Day 3-4: Error Handling & Logging**
1. Read [TECHNICAL.md - Error Handling](TECHNICAL.md#error-handling)
2. Study `GlobalExceptionHandler.java`
3. Read [TECHNICAL.md - Logging](TECHNICAL.md#logging--observability)
4. Explore `LogAspect.java`
5. Practice: Trigger an error and trace it through logs

**Day 5: Review & Practice**
1. Read [FAQ.md](FAQ.md) - Common questions and answers
2. Try the workflows in [EXAMPLES.md - Common Workflows](EXAMPLES.md#common-workflows)
3. Review code with a senior developer

---

## For Backend Developers (Experienced with Spring Boot)

### Fast Track (3-5 Days)

**Day 1: Architecture & Data Model**
1. [README.md](README.md) - Quick overview
2. [ARCHITECTURE.md](ARCHITECTURE.md) - Focus on:
   - Vertical Slice Architecture
   - Security Architecture
   - ADRs
3. [TECHNICAL.md - Data Model](TECHNICAL.md#data-model) - Study ERD
4. [GLOSSARY.md](GLOSSARY.md) - Skim for Genesis-specific terms

**Day 2: Module Structure**
1. [MODULES.md](MODULES.md) - Understand all modules
2. Pick 2-3 modules and explore the code:
   - Employee (basic CRUD)
   - Schedule (async processing)
   - Auth (Cognito integration)
3. Trace a complete request flow

**Day 3: API Standards & Patterns**
1. [TECHNICAL.md - API Standards](TECHNICAL.md#api-standards) - Response envelopes, pagination
2. [EXAMPLES.md](EXAMPLES.md) - Practice API calls
3. Study shared utilities:
   - `ValidationHelper`
   - `RepositoryHelper`
   - `LogAspect`

**Day 4: Development**
1. Set up local environment: [OPERATIONS.md](OPERATIONS.md#local-development)
2. Make a small feature change
3. Write tests
4. Review [FAQ.md](FAQ.md) for best practices

**Day 5: Advanced Topics**
1. Async processing and webhooks
2. Error handling patterns
3. Database migrations with Flyway
4. Deployment with Docker

---

## For Frontend Developers (API Consumers)

### API Integration Path (2-3 Days)

**Day 1: Authentication**
1. [README.md](README.md) - Platform overview
2. [EXAMPLES.md - Authentication Examples](EXAMPLES.md#authentication-examples)
3. Practice:
   - Register user
   - Login and get JWT tokens
   - Make authenticated requests
   - Handle token expiration

**Day 2: API Endpoints**
1. [TECHNICAL.md - API Standards](TECHNICAL.md#api-standards) - Response format, pagination
2. [MODULES.md](MODULES.md) - Review endpoints for each module
3. [EXAMPLES.md - Employee Management](EXAMPLES.md#employee-management) - CRUD operations
4. Practice:
   - List employees with pagination
   - Create/update/delete employees
   - Handle validation errors

**Day 3: Advanced Features**
1. [EXAMPLES.md - Schedule Generation](EXAMPLES.md#schedule-generation) - Async operations
2. [EXAMPLES.md - Error Handling](EXAMPLES.md#error-handling) - RFC 7807 Problem Details
3. [FAQ.md](FAQ.md) - Common API questions
4. Practice:
   - Create schedule request
   - Poll for status
   - Handle webhook callbacks
   - Error handling

---

## For Architects & Tech Leads

### Architecture Review Path (1-2 Days)

**Part 1: System Design**
1. [README.md](README.md) - Platform capabilities
2. [ARCHITECTURE.md](ARCHITECTURE.md) - Complete read, focus on:
   - System Context
   - Vertical Slice Architecture
   - Security Architecture
   - All 5 ADRs
3. [TECHNICAL.md - Data Model](TECHNICAL.md#data-model) - ERD and entity relationships

**Part 2: Technical Decisions**
1. Review ADRs:
   - ADR-001: Vertical Slice Architecture
   - ADR-002: AWS Cognito for Authentication
   - ADR-003: AOP for Cross-Cutting Logging
   - ADR-004: External Optimization Engine
   - ADR-005: Flyway for Database Migrations
2. Evaluate trade-offs and consequences
3. Consider scalability and extensibility

**Part 3: Operations & Deployment**
1. [OPERATIONS.md](OPERATIONS.md) - Deployment strategies, monitoring
2. Review:
   - Docker configuration
   - Environment variables
   - Database management
   - Performance tuning

---

## For DevOps Engineers

### Operations Path (2-3 Days)

**Day 1: Deployment**
1. [README.md](README.md) - Technology stack
2. [OPERATIONS.md - Deployment](OPERATIONS.md#deployment) - Complete read
3. Practice:
   - Local deployment
   - Docker build and run
   - Docker Compose setup
4. Verify health checks

**Day 2: Configuration & Database**
1. [OPERATIONS.md - Configuration](OPERATIONS.md#configuration) - Environment variables
2. [OPERATIONS.md - Database Management](OPERATIONS.md#database-management) - Flyway migrations
3. Practice:
   - Configure different environments (dev, sit, prod)
   - Run migrations
   - Connection pool tuning

**Day 3: Monitoring & Troubleshooting**
1. [OPERATIONS.md - Monitoring](OPERATIONS.md#monitoring--health-checks) - Actuator, logs, metrics
2. [OPERATIONS.md - Troubleshooting](OPERATIONS.md#troubleshooting) - Common issues
3. [FAQ.md](FAQ.md) - Troubleshooting questions
4. Practice:
   - Set up Prometheus scraping
   - Configure log aggregation
   - Simulate and resolve common issues

---

## For QA Engineers

### Testing Path (2-3 Days)

**Day 1: Understanding the System**
1. [README.md](README.md) - Platform overview
2. [MODULES.md](MODULES.md) - All modules and their responsibilities
3. [TECHNICAL.md - API Standards](TECHNICAL.md#api-standards) - Expected response formats

**Day 2: API Testing**
1. [EXAMPLES.md](EXAMPLES.md) - All code examples
2. Practice:
   - Authentication flow
   - CRUD operations
   - Pagination and sorting
   - Error scenarios
3. [TECHNICAL.md - Error Handling](TECHNICAL.md#error-handling) - Expected error responses

**Day 3: Test Scenarios**
1. [EXAMPLES.md - Common Workflows](EXAMPLES.md#common-workflows) - End-to-end flows
2. [FAQ.md](FAQ.md) - Edge cases and error scenarios
3. Create test cases for:
   - Happy paths
   - Validation errors
   - Authentication failures
   - Async operations
   - Concurrent requests

---

## Recommended Reading Order by Topic

### Topic: Authentication & Security
1. [ARCHITECTURE.md - Security Architecture](ARCHITECTURE.md#security-architecture)
2. [EXAMPLES.md - Authentication Examples](EXAMPLES.md#authentication-examples)
3. [MODULES.md - Auth Module](MODULES.md#1-auth-module)
4. [FAQ.md - Authentication Questions](FAQ.md#how-does-authentication-work)

### Topic: Data Model & Database
1. [TECHNICAL.md - Data Model](TECHNICAL.md#data-model)
2. [OPERATIONS.md - Database Management](OPERATIONS.md#database-management)
3. [FAQ.md - Data Model Questions](FAQ.md#data-model-questions)
4. [GLOSSARY.md](GLOSSARY.md) - Database terms

### Topic: Schedule Generation
1. [MODULES.md - Schedule Module](MODULES.md#6-schedule-module)
2. [ARCHITECTURE.md - Async Processing](ARCHITECTURE.md#async-processing)
3. [EXAMPLES.md - Schedule Generation](EXAMPLES.md#schedule-generation)
4. [FAQ.md - Schedule Generation Questions](FAQ.md#schedule-generation-questions)

### Topic: API Development
1. [TECHNICAL.md - API Standards](TECHNICAL.md#api-standards)
2. [EXAMPLES.md - Employee Management](EXAMPLES.md#employee-management)
3. [FAQ.md - Development Questions](FAQ.md#development-questions)
4. [MODULES.md](MODULES.md) - All modules

---

## Learning Resources

### Code Exploration
- Start with simple modules: Employee, Company, Branch
- Move to complex modules: Schedule, Auth
- Study shared utilities: `LogAspect`, `ValidationHelper`, `RepositoryHelper`

### Hands-On Practice
1. Set up local environment
2. Make API calls using curl or Postman
3. Add a new field to an entity
4. Create a new endpoint
5. Write a Flyway migration
6. Deploy with Docker

### Code Review Checklist
- [ ] Follows Vertical Slice Architecture
- [ ] Uses DTOs (not entities) in API responses
- [ ] Proper error handling with domain exceptions
- [ ] Validation on request DTOs
- [ ] `@Transactional` for multi-entity operations
- [ ] Logging handled by `LogAspect` (no manual logging)
- [ ] Flyway migration for schema changes

---

## Next Steps After Learning

1. **Contribute**: Pick a task from the backlog
2. **Extend**: Add a new module following the existing pattern
3. **Optimize**: Identify performance bottlenecks
4. **Document**: Update documentation as you learn
5. **Mentor**: Help onboard the next new developer

---

## Quick Reference

| Role | Start Here | Time Needed |
|------|------------|-------------|
| New Developer | [README.md](README.md) | 2-3 weeks |
| Backend Developer | [ARCHITECTURE.md](ARCHITECTURE.md) | 3-5 days |
| Frontend Developer | [EXAMPLES.md](EXAMPLES.md) | 2-3 days |
| Architect | [ARCHITECTURE.md](ARCHITECTURE.md) | 1-2 days |
| DevOps | [OPERATIONS.md](OPERATIONS.md) | 2-3 days |
| QA Engineer | [MODULES.md](MODULES.md) | 2-3 days |
