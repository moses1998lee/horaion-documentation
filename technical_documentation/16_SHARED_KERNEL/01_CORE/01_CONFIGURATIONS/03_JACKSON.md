# 03 - Jackson Configuration

> **JSON Serialization Standards**

---

## Overview

Horaion uses **Jackson** `ObjectMapper` for all JSON processing. This configuration ensures consistent date and time formatting across all API endpoints.

**Class**: `com.horaion.app.shared.core.configurations.JacksonConfiguration`

---

## Configuration Details

### 1. Java 8 Time Support
We register the `JavaTimeModule` to correctly handle modern Java date types:
*   `java.time.Instant`
*   `java.time.LocalDate`
*   `java.time.LocalDateTime`

### 2. ISO-8601 Enforcement
We explicitly **disable** `SerializationFeature.WRITE_DATES_AS_TIMESTAMPS`.

*   **Bad (Timestamp)**: `1698401200` (Ambiguous, hard to debug)
*   **Good (ISO-8601)**: `"2023-10-27T10:06:40Z"` (Standard, readable)

{% hint style="danger" %}
**Critical:** **Never** serialize dates as raw numbers (Activities, Timestamps). Frontend apps expect ISO-8601 strings. If you see a number in a date field, this configuration has been bypassed.
{% endhint %}

---

## Customizing Serialization

If you need a specific format for just *one* field, use the `@JsonFormat` annotation to override the global default.

```java
public class EventDTO {
    
    // Uses Global Default (ISO-8601)
    private Instant createdAt;
    
    // Override: Custom Pattern
    @JsonFormat(pattern = "yyyy-MM-dd")
    private LocalDate eventDate;
}
```
