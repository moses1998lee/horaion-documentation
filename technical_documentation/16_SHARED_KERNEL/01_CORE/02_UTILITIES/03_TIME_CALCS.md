# 03 - TimeBlockCalculator

> **Discrete Math for Scheduling**

---

## Overview

The Horaion scheduling engine views time not as continuous, but as discrete **Time Blocks**. This utility converts between human-readable time (`09:00`) and machine-readable indices (`36`).

**Class**: `com.horaion.app.shared.core.utils.TimeBlockCalculator`

---

## Concepts

### The 15-Minute Grid
By default, the day is divided into 96 blocks (15 minutes each).
*   **00:00** -> Index `0`
*   **00:15** -> Index `1`
*   ...
*   **23:45** -> Index `95`

### Multi-Day Indexing
For multi-day schedules, the index continues incrementing.
*   **Day 1, 00:00** -> Index `0`
*   **Day 2, 00:00** -> Index `96`
*   **Day 3, 00:00** -> Index `192`

---

## Key Functions

### `calculateBlockPosition`
Converts a specific `LocalDateTime` into a single integer index.

```java
int index = TimeBlockCalculator.calculateBlockPosition(
    LocalDateTime.of(2023, 10, 27, 9, 30), // Target
    LocalDate.of(2023, 10, 27),            // Schedule Start
    LocalTime.MIDNIGHT,                    // Day Start Time
    15                                     // Block Size
);
// Result: 38
```

### `calculateShiftTimeBlocks`
Returns a **List** of indices for a shift. Handles overnight shifts automatically.

```java
// Shift: 22:00 to 02:00 (Next Day)
List<Integer> blocks = TimeBlockCalculator.calculateShiftTimeBlocks(
    LocalTime.of(22, 0), 
    LocalTime.of(2, 0),
    LocalTime.MIDNIGHT, 
    15
);
// Result: [88, 89, ..., 95, 96, ..., 103]
```

{% hint style="danger" %}
**Critical:** The block size must be consistent across the entire schedule (15, 30, or 60 minutes). Mixing block sizes will cause calculation errors.
{% endhint %}
