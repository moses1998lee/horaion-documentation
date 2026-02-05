# 01 - Shared Kernel Overview

> **Common Components & Infrastructure for the Horaion Platform**

---

## Introduction

The **Shared Kernel** (`com.horaion.app.shared`) is the backbone of the Horaion application. It provides a robust set of reusable components, infrastructure adapters, and standardized patterns that all feature modules must use.

**Why a Shared Kernel?**
Instead of duplicating logic in every module (like "how to send an email" or "how to format a JSON error"), we centralize these concerns here. This ensures:
1.  **Consistency**: Every API endpoint returns the same JSON structure.
2.  **Maintainability**: Fix a bug in the Email Service, and it's fixed everywhere.
3.  **Speed**: Developers focus on business logic (`ShiftService`), not plumbing (`S3Client`).

---

## Architecture

The Shared Kernel is organized into distinct layers, mirroring the application's clean architecture:

```mermaid
graph TD
    subgraph Shared_Kernel [com.horaion.app.shared]
        Core[Core Layer]
        Infra[Infrastructure Layer]
        Sec[Security Layer]
        DB[Database Layer]
    end

    subgraph Core
        Resp[Standard Responses]
        Ex[Global Exceptions]
        Util[Utilities]
    end

    subgraph Infra
        AWS[AWS Clients]
        Excel[Excel Engine]
        Notify[Notification Engine]
    end

    subgraph Sec
        Context[Security Context]
        Filters[Auth Filters]
    end

    subgraph DB
        Audit[Auditing]
        BaseRepo[Base Repositories]
    end

    Core --> Infra
    Core --> Sec
    Core --> DB

    style Shared_Kernel fill:#f3e5f5,stroke:#7b1fa2
    style Core fill:#e3f2fd,stroke:#1565c0
    style Infra fill:#e8f5e9,stroke:#2e7d32
    style Sec fill:#fff3e0,stroke:#ef6c00
    style DB fill:#ffebee,stroke:#c62828
```

### Diagram Walkthrough

1.  **Core Layer (The Foundation)**: This is the bottom of the pyramid. `Resp`, `Ex` (Exceptions), and `Util` classes live here. They don't depend on *anything* else. Everyone else uses them.
2.  **Infrastructure Layer (The Adapters)**: This layer does the "dirty work" of talking to the outside world (`AWS`, `Excel`, `Notify`). It depends on **Core** because it might need to throw a `BaseException` or return a standard response.
3.  **Security Layer (The Gatekeeper)**: Handles `Filters` and contexts. It protects the other layers.
4.  **Database Layer (The Storage)**: Contains `BaseRepository` definitions.

**Understanding the Arrows**:
The arrows (`-->`) represent **dependency flow**.
*   **Core** points to **Infra**, **Sec**, and **DB**: This means `Core` is used by *everyone*.
*   **Feature Modules** (not shown): Will sit on top and consume these shared layers.

---

## Package Structure

```
com.horaion.app.shared
├── core
│   ├── configurations
│   ├── constants
│   ├── enums
│   ├── exceptions
│   ├── helpers
│   ├── interfaces
│   ├── repositories
│   ├── responses
│   └── utils
├── database
│   ├── configurations
│   ├── constants
│   ├── enums
│   └── properties
├── infrastructure
│   ├── aws
│   ├── engine
│   ├── excel
│   ├── genesis
│   ├── messaging
│   ├── notification
│   └── properties
├── logging
│   ├── aspects
│   ├── constants
│   ├── exceptions
│   └── properties
├── metrics
│   └── configurations
├── providers
│   ├── impl
│   ├── FieldOptionConstants.java
│   └── FieldOptionProvider.java
├── resolvers
│   └── RuleFieldOptionResolver.java
└── security
    ├── annotations
    ├── aspects
    ├── configurations
    ├── constants
    ├── exceptions
    ├── handlers
    ├── jwt
    ├── properties
    ├── services
    └── utils
```

---

## Guiding Principles

### 1. No Business Logic
The Shared Kernel **never** contains business rules (e.g., "A shift cannot be longer than 8 hours"). It only contains *capabilities* (e.g., "Here is how you calculate the difference between two timestamps").

### 2. Standardization
If a pattern is used in more than one module, it belongs here.
*   **Bad**: `ShiftController` defines its own `ErrorResponse` class.
*   **Good**: `ShiftController` uses `shared.core.responses.ErrorResponse`.

### 3. Dependency Rule
The Shared Kernel should have minimal dependencies on the rest of the application. It is the *foundation*, not the *roof*. Feature modules depend on Shared; Shared does not depend on Feature modules.
