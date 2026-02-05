# 03 - Database Enums

> **Supported Database Types**

**File**: [`DatabaseType.java`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/database/enums/DatabaseType.java)

This enum acts as a registry for the databases our platform supports. It links the **JDBC Prefix** to the correct **Driver Class**.

### Registry

| Enum | Prefix | Driver Class |
| :--- | :--- | :--- |
| `POSTGRESQL` | `jdbc:postgresql:` | `org.postgresql.Driver` |
| `H2` | `jdbc:h2:` | `org.h2.Driver` |
| `MYSQL` | `jdbc:mysql:` | `com.mysql.cj.jdbc.Driver` |

### Logic
The `fromJdbcUrl(String url)` method iterates through these values to determine which driver to load.

```java
// Example
String url = "jdbc:postgresql://localhost:5432/horaion";
DatabaseType type = DatabaseType.fromJdbcUrl(url); // Returns POSTGRESQL
String driver = type.getDriverClassName(); // Returns "org.postgresql.Driver"
```

{% hint style="success" %}
**Success:** This means you never have to manually update `driver-class-name` in your YAML files when switching databases (e.g., from H2 in tests to Postgres in Prod). The system knows what to do.
{% endhint %}
