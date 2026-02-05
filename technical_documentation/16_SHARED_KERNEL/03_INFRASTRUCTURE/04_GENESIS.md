# 04 - Genesis Clients

> **Internal Microservice Communication**

The `genesis` package contains the **Feign Clients** used to communicate with other services within the Horaion (formerly Genesis) ecosystem.

**Package**: `com.horaion.app.shared.infrastructure.genesis.clients`

---

## Overview

Although Horaion is currently a Modular Monolith, we design our interactions as if they were remote services. This prepares us for a future split into Microservices.

### Client Generatrion
These clients (`GenesisEmployeeClient`, `GenesisBranchClient`) are typically used to fetch data from other modules.

**Example**:
The **Schedule Module** needs to know if an Employee exists. It calls:
```java
// Logic inside Schedule Service
var employee = genesisEmployeeClient.getEmployeeById(empId);
```

{% hint style="warning" %}
**Important:** Do not bypass the Client to access another module's Database Repository directly. This violates the module boundary and creates tight coupling ("Database Integration" anti-pattern). Always go through the public API or these Clients.
{% endhint %}
