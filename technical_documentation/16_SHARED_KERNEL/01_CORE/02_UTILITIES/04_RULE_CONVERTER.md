# 04 - RuleValueConverter

> **Data Marshalling for Optimization Engine**

---

## Overview

The Genesis optimization engine (built in Python) is strict about data types. The `RuleValueConverter` ensures that Java objects are formatted exactly as the engine expects.

**Class**: `com.horaion.app.shared.core.utils.RuleValueConverter`

---

## Formatting Rules

### Booleans
Java uses lowercase `true`/`false`. The Engine expects **Title Case**.

*   Java: `true` -> Engine: `"True"`
*   Java: `false` -> Engine: `"False"`

### Numbers
*   **Doubles/BigDecimals**: Trailing zeros are stripped to save JSON payload size.
    *   `10.5000` -> `"10.5"`
    *   `12.0` -> `"12"`

### Collections
Multi-value rules (e.g., "Must have skills A, B, C") are converted to **comma-separated strings**.
*   List: `["Java", "Python"]` -> String: `"Java,Python"`

---

## API Usage

Use the static methods for safe conversion. They handle `null` gracefully (returning `null`).

```java
// Converting a Map of arbitrary inputs
Map<String, String> payload = RuleValueConverter.convertRuleMap(rawInputs);
```

{% hint style="warning" %}
**Important:** Do not use `.toString()` on Integers or Booleans when sending data to the Engine. Direct string conversion might produce `"true"` (lowercase), which will cause the Engine to crash or misinterpret the rule. Always use `RuleValueConverter`.
{% endhint %}
