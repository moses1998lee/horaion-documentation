# Employee Module Configuration

## Database Configuration

The **Employee Module** manages the `employees` and `employee_department_assignments` tables.

### Indexes & Performance

| Index Name | Column | Purpose |
| :--- | :--- | :--- |
| `idx_employees_name` | `last_name`, `first_name` | **Composite Index**. Speeds up "Search by Name" which happens on every shift assignment screen. |
| `idx_employees_company_id` | `company_id` | Tenant isolation. |
| `idx_employees_employment_type` | `employment_type` | Filtering (e.g., "Show me all Contractors"). |

## Application Properties

### External Dependencies
*   **AWS Cognito**: This module writes to Cognito groups. Ensure the `horaion.cognito.user-pool-id` is correctly set in `application.yml`.
*   **S3 (Optional)**: If profile pictures are enabled, `horaion.s3.bucket-name` is used to store the images referenced in `profile_picture_url`.

{% hint style="info" %}
**Note:**
**Code Logic Constant**: `EMPLOYEE_CODE_LENGTH` is defined as `8` in `EmployeeService.java`. This is currently hardcoded and not configurable via YAML.
{% endhint %}
