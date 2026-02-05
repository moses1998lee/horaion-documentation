# 07 - Core Repositories

> **Data Access Layer Extensions**

Spring Data JPA is powerful, but we extend it to enforce our specific security requirements regarding Resource Identification.

---

## 1. UUID Repository `<T>`
**File**: [`UuidRepository.java`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/core/repositories/UuidRepository.java)

### The "ID vs UUID" Strategy

In Horaion, every database table usually has **two** unique identifiers:

1.  **Primary Key (`Long id`)**:
    *   **Type**: Sequential Integer (1, 2, 3...).
    *   **Use**: Internal Foreign Keys, JOINs, fast indexing.
    *   **Exposure**: **NEVER** expose this to the Frontend.
2.  **Business Key (`UUID uuid`)**:
    *   **Type**: Random 128-bit string (`550e8400-e29b...`).
    *   **Use**: Public URLs (`/api/employees/550e...`).
    *   **Exposure**: Safe to expose.

{% hint style="danger" %}
**Critical Security Risk:** Exposing sequential Integer IDs allows **Insecure Direct Object Reference (IDOR)** attacks. A hacker can verify "Employee 100" exists, then guess "Employee 101" to scrape your entire database. UUIDs make guessing impossible.
{% endhint %}

### How it works
This repository adds a pre-built method to find entities by this safe key:

```java
public interface UuidRepository<T> extends JpaRepository<T, Long> {
    
    // Finds record by secure UUID
    Optional<T> findByUuid(UUID uuid);

    // Checks existence safely
    boolean existsByUuid(UUID uuid);
}
```

By extending `UuidRepository` instead of plain `JpaRepository`, your module automatically gains these security features.
