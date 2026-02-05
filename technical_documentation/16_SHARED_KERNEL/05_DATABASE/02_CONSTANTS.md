# 02 - Database Constants

> **Connection Pool Defaults**

We define sensible default values for our Connection Pool configuration. These values are used if you do *not* override them in `application.yaml`.

**File**: [`DatabaseConstants.java`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/database/constants/DatabaseConstants.java)

---

## Default Values

| Constant | Value | Description |
| :--- | :--- | :--- |
| `DEFAULT_MAXIMUM_POOL_SIZE` | **10** | Maximum number of active connections to the database. |
| `DEFAULT_MINIMUM_IDLE` | **10** | We keep the pool full ("Fixed Size Pool") to prevent latency spikes during traffic bursts. |
| `DEFAULT_CONNECTION_TIMEOUT` | **30s** | How long a thread waits for a connection before throwing an error. |
| `DEFAULT_MAX_LIFETIME` | **30m** | How long a connection lives before being retired (prevents firewall issues). |

{% hint style="info" %}
**Why Fixed Pool Size?**
By setting `minimumIdle` equal to `maximumPoolSize`, HikariCP creates a fixed-size pool. This is recommended for production because it avoids the overhead of creating/destroying connections during runtime.
{% endhint %}

---

## JDBC Prefixes
We use these constants to auto-detect the database type:
*   `JDBC_POSTGRESQL`: `"jdbc:postgresql:"`
*   `JDBC_MYSQL`: `"jdbc:mysql:"`
*   `JDBC_H2`: `"jdbc:h2:"`
