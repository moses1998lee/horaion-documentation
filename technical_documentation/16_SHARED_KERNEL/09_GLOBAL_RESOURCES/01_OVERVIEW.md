# 09 - Global Resources

> **Application Configurations, Database Scripts, and Asset Templates**

---

## 9.1 Introduction

The logical **Global Resources** module physically corresponds to the `src/main/resources` directory in the project root. While these files are not Java classes inside the `com.horaion.app.shared` package, they are critical **shared artifacts** consumed by the entire application.

In the spirit of Domain-Driven Design (DDD), we categorize these based on their *domain usage* rather than just their file type.

### Directory Structure Map

| Logical Name | Physical Path | Purpose |
| :--- | :--- | :--- |
| **Properties** | `src/main/resources/application.yaml` | The "Brain". Controls behavior across all environments (Dev, Staging, Prod). |
| **Migrations** | `src/main/resources/db/migrations/` | The "Evolution". SQL scripts managed by Flyway to version-control the database schema. |
| **Templates** | `src/main/resources/templates/` | The "Static Assets". Excel templates, HTML email bodies, and other non-code files. |

{% hint style="info" %}
**Note:** Logback configuration (`logback-spring.xml`) is documented separately in the **Logging** module because it is closely tied to the `shared.logging` aspect. See [Logging Configuration](../05_LOGGING/02_CONFIGURATIONS.md).
{% endhint %}

---

## 9.2 Resource Lifecycle

Unlike Java code which is compiled:
1.  **Properties**: Are injected at runtime.
2.  **Migrations**: Are executed at startup (before the app accepts traffic).
3.  **Templates**: Are loaded on-demand or verified at startup.

This separation ensures that a change in configuration (e.g., rotating a database password) typically only requires a restart, not a re-compilation (unless using baked-in container variables).
