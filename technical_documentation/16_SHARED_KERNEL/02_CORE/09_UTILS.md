# 09 - Core Utilities

> **Complex Logic, Simplified**

---

## 1. Service Executor ("Safe Runner")
**File**: ``ServiceOperationExecutor.java``

**Context**: Business logic often needs three things:
1.  **Logging**: "Started X", "Finished X (15ms)".
2.  **Error Handling**: "If DB fails, wrap in custom exception".
3.  **Tracing**: "Keep the Trace ID alive".

Instead of writing this `try-catch-log` block in every method, we use the **Service Executor**.

### Usage Pattern

**Before (Messy)**
```java
public void doWork() {
    log.info("Starting work...");
    long start = System.currentTimeMillis();
    try {
        repository.save(data);
        log.info("Finished work in {}ms", System.currentTimeMillis() - start);
    } catch (Exception e) {
        log.error("Work failed", e);
        throw new RuntimeException(e);
    }
}
```

**After (Clean)**
```java
public void doWork() {
    serviceExecutor.execute("doWork", () -> {
        repository.save(data);
    });
}
```

{% hint style="success" %}
**Success:** This ensures standard log formats across the entire platform, making debugging 10x easier.
{% endhint %}

---

## 2. Time Calculations
**File**: ``TimeBlockCalculator.java``

A robust utility for handling time ranges, specifically for **Shift Management**.

### Key Methods
*   `isOverlapping(StartA, EndA, StartB, EndB)`: Determines if two shifts clash. Crucial for the "Double Booking" constraint.
*   `calculateDuration(Start, End)`: Returns duration in minutes.

{% hint style="info" %}
**Note:** This utility handles edge cases like "End time is before Start time" (crossing midnight) depending on implementation details. Always check the JavaDoc.
{% endhint %}
