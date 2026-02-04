# 05 - Shared Database

> **Persistence Patterns and Repositories**

---

## 5.1 Base Repository
**Package**: `com.horaion.app.shared.core.repositories`

### UuidRepository`<T>`
This repository pattern enforces a **split-ID strategy**:
*   **Primary Key (Internal)**: `Long` (Auto-increment, used for FK joins and performance).
*   **Business Key (External)**: `UUID` (Randomly generated, exposed in APIs).

```java
@NoRepositoryBean
public interface UuidRepository<T> extends JpaRepository<T, Long> {
    Optional<T> findByUuid(UUID uuid);
}
```

**Usage**:
```java
@Repository
public interface ShiftRepository extends UuidRepository<Shift> {
    // findByUuid() is available by default
}
```

---

## 5.2 Persistence Patterns

### 1. Security vs Performance: Long vs UUID
The system uses `Long` internally because it is smaller (64-bit) and index-friendly, preventing page fragmentation. However, we **never** expose `id` (Long) to the frontend API. We always map the `uuid` field to the DTO's `id` field.

### 2. Physical vs. Logical Deletion
The `UuidRepository` supports standard `delete()` methods. However, business-critical entities (like Shifts/Employees) must implement **Soft Deletion** at the service layer (setting `is_active = false` or `deleted_at = now()`) rather than scrubbing data from the disk.

> **Note**: Currently, Soft Deletion logic is implemented per-service (e.g., in `ShiftService`), not enforced by a shared Aspect.
