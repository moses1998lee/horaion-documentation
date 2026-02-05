# 02 - Logging Constants

> **Standardized MDC Keys**

**File**: ``AspectConstant.java``

To ensure our log aggregation system (e.g., Splunk, ELK Stack) can parse logs correctly, we use fixed keys for the Mapped Diagnostic Context.

---

## Keys

| Constant | Key String | Description |
| :--- | :--- | :--- |
| `MDC_REQUEST_ID` | `requestId` | 8-char random ID for the lifecycle of the request. |
| `MDC_USER_ID` | `userId` | The Security Principal (e.g., 'john.doe'), if authenticated. |
| `MDC_METHOD` | `method` | Short name of the method being executed (e.g., `EmployeeService.create`). |
| `MDC_HTTP_METHOD` | `httpMethod` | GET, POST, PUT, etc. |
| `MDC_ENDPOINT` | `endpoint` | The URI path (e.g., `/api/v1/employees`). |

{% hint style="warning" %}
**Important:** Do not use these keys for your own custom MDC variables to avoid collisions. If you need custom tracing, prefix it (e.g., `job_resourceId`).
{% endhint %}
