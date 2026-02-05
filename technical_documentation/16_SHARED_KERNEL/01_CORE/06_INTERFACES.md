# 06 - Core Interfaces

> **Standardizing Service Contracts**

Interfaces in the Core package define the "Rules of Engagement" for our services. They ensure that all modules behave in a predictable way.

---

## 1. IGenericService `<T, K, ID>`
**File**: ``IGenericService.java``

**Purpose**: This is a **Marker Interface** that forces standardized naming convention for CRUD operations. If every service named their "Get All" method differently (`fetchAll`, `list`, `retrieveAll`), the codebase would be a mess.

### The Contract

| Method | Signature | Description |
| :--- | :--- | :--- |
| **Get All** | `PagedResponse<K> getAll(...)` | Must support pagination. |
| **Get One** | `K getById(ID id)` | Helper for internal use. |
| **Get public** | `K getByUuid(UUID uuid)` | The standard for public consumption. |
| **Create** | `K save(T request)` | Takes a specific Request DTO. |
| **Update** | `K update(ID id, T request)` | Updates existing record. |
| **Delete** | `void delete(ID id)` | Hard or Soft delete (implementation specific). |

### Generics Explained
*   **`T` (Request DTO)**: The object coming IN (e.g., `CreateShiftRequest`).
*   **`K` (Response DTO)**: The object going OUT (e.g., `ShiftResponse`).
*   **`ID`**: Usually `Long`, but generic for flexibility.

{% hint style="success" %}
**Tip:** By extending this interface, you ensure your Service looks and feels like every other service in the platform, lowering the learning curve for new developers.
{% endhint %}
