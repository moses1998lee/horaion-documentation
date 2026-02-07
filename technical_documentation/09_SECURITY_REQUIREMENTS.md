# - Security Requirements

> **Policies, Compliance, and Threat Model**

---

## Authentication Policy

**Requirement**: All human and machine access must be authenticated via **AWS Cognito**.

*   **Protocol**: OAuth 2.0 / OIDC.
*   **Token Format**: JWT (JSON Web Token).
*   **MFA**: Multifactor Authentication is **mandatory** for all Administrator roles.

> [!TIP]
> For implementation details (JWT validation filters, Claims extraction), see [Shared Security](../technical_documentation/16_SHARED_KERNEL/08_SECURITY/01_OVERVIEW.md).

---

## Authorization Policy

**Requirement**: Zero Trust. Authenticated users generally have **no access** until explicitly granted.

### Role-Based Access Control (RBAC)
We employ a tiered role system:
1.  **SysAdmin**: Full platform control (Cross-tenant).
2.  **CompanyAdmin**: Full control within their `company_id`.
3.  **Manager**: Write access to their `department_id`.
4.  **Employee**: Read-only access to their own data.

### Scope Enforcement
Every API request must be validated against the `custom:companyId` claim in the JWT. A user sending a valid token for Company A must be rejected (403 Forbidden) if they try to access `/api/v1/companies/{Company_B_ID}`.

---

## Data Protection

### At Rest
*   **Database**: Encrypted bucket/volume (AWS/DO managed encryption).
*   **Backups**: Encrypted and retained for 30 days.

### In Transit
*   **TLS**: All traffic must be encrypted via TLS 1.2+.
*   **No Mixed Content**: HTTP requests must be redirected to HTTPS.

### PII (Personally Identifiable Information)
The following fields are classified as 'Sensitive':
*   Email Address
*   Phone Number
*   Home Address

Access to these fields must be logged via the Audit Trail.

---

## Network Security

*   **Public Access**: Only port 443 (HTTPS) is exposed to the internet via the Load Balancer.
*   **Private Subnet**: The Database and private microservices (Engine) must reside in a private VPC subnet, inaccessible from the public internet.
*   **IP Whitelisting**: The `Engine Callback` endpoint is restricted to a specific IP range defined in `app.security.engine-ip-whitelist`.
