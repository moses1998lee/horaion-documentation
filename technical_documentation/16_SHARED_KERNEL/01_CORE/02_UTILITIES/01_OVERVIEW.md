# 01 - Utilities Overview

> **Core Logic and Helper Functions**

---

## Introduction

The `shared/core/utils` package contains high-value, domain-agnostic logic used throughout the system. These are **pure functions** or **safe execution wrappers** that simplify complex tasks.

## Utility Catalog

| Class | Purpose | Key Feature |
| :--- | :--- | :--- |
| **[ServiceOperationExecutor](02_SERVICE_EXECUTOR.md)** | "Safe Execute" wrapper for services. | Standardizes logging, tracing, and exception handling. |
| **[TimeBlockCalculator](03_TIME_CALCS.md)** | Scheduler math engine. | Converts `LocalDateTime` to integer `blockIndex` (0-95). |
| **[RuleValueConverter](04_RULE_CONVERTER.md)** | Engine data marshalling. | Formats data for the Python optimization engine ("True" vs "true"). |

---

{% hint style="success" %}
**Tip:** Always prefer using these utilities over writing custom logic. For example, calculating time blocks manually is error-prone (especially around midnight); `TimeBlockCalculator` handles these edge cases perfectly.
{% endhint %}
