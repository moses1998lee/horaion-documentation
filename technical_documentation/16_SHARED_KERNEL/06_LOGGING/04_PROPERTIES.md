# 04 - Logging Properties

> **Configuration as Code**

**File**: ``LogAspectProperties.java``

**Prefix**: `app.logging.aspect`

Allows you to fine-tune the behavior of the auto-logger without recompiling code.

---

## Configuration Options

| Property | Default | Description |
| :--- | :--- | :--- |
| `enabled` | `true` | Master switch. Set to `false` to disable all AOP logging (useful for performance testing). |
| `slow-execution-threshold-ms` | `1000` | Methods slower than this will log as **WARN**. |

### YAML Example

```yaml
app:
  logging:
    aspect:
      enabled: true
      slow-execution-threshold-ms: 2000 # Increase threshold for slower environments
```

{% hint style="success" %}
**Success:** Using a Record for configuration properties (Java 16+) reduces boilerplate and ensures immutability.
{% endhint %}
