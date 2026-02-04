# Company Module

| Attribute     | Details                                       |
| :------------ | :-------------------------------------------- |
| **Namespace** | `com.horaion.app.modules.company`             |
| **Status**    | 🟢 Stable                                     |
| **Criticality** | Critical (Root of Multi-tenancy)             |
| **Dependencies** | None (Top-level entity)                      |

## Executive Summary

The **Company Module** is the foundation of Horaion's multi-tenant architecture. Every user, branch, shift, and schedule belongs to a **Company**. It manages the unique business entities that subscribe to our platform.

{% hint style="success" %}
**Tip / Success:**
Think of the **Company** as the "Tenant" or the "Account Owner". It is the strict boundary for data isolation. User A from Company X should **never** see data from Company Y.
{% endhint %}

{% hint style="warning" %}
**Important / Warning:**
**Unique Identity**: Every company is uniquely identified by its **Registration Number** (e.g., CIPC Number, Tax ID). This prevents duplicate accounts for the same real-world business.
{% endhint %}

## Hierarchy & Relationships

The Company is the root node of the entire Horaion data structure.

```mermaid
graph TD
    Root[Global Horaion Platform] --> CompanyA[Company: Acme Corp]
    Root --> CompanyB[Company: Beta Ltd]
    
    CompanyA --> Branch1[Branch: HQ]
    CompanyA --> Branch2[Branch: Warehouse]
    
    Branch1 --> Emp1[Employee: Alice]
    Branch2 --> Emp2[Employee: Bob]
    
    CompanyB --> Branch3[Branch: Main Office]
```

## Core Capabilities

1.  **Tenant Management**:
    *   **Onboarding**: Managing the transition from a "New Sign-up" to a "Fully Configured" active tenant.
    *   **Lifecycle**: Creation, Suspension, and Deletion of company accounts.
2.  **Identity Verification**:
    *   Enforcing legal uniqueness via `registrationNumber`.
3.  **Discovery**:
    *   Providing admin-level search tools to find companies by name or ID.

## Responsibilities

*   **Single Source of Truth**: For the legal entity's profile.
*   **Data Isolation Root**: All other modules (Auth, Branch, Employee) reference `company_id` to enforce security barriers.
*   **Compliance**: Storing the legal registration number for audit trails.

## Module Architecture

```mermaid
graph TD
    Client[Client App] --> CompanyController
    
    subgraph "Company Module"
        CompanyController --> CompanyService
        CompanyService --> CompanyRepository
    end
    
    CompanyService --> SecurityContext[Security Context]
    CompanyRepository --> DB[(Database)]
```
