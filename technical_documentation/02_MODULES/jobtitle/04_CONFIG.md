# Job Title Module Configuration

## Database Configuration

The **Job Title Module** manages the `job_titles` table.

### Indexes & Performance

| Index Name | Column | Purpose |
| :--- | :--- | :--- |
| `idx_job_titles_company_id` | `company_id` | **Multi-Tenancy**: Ensures branch managers only see their company's roles. |
| `idx_job_titles_name` | `name` | **Uniqueness Check**: Fast validation during creation to prevent duplicates. |

### Constraints

*   `uq_job_titles_company_name`: Composite Unique Constraint on `(company_id, name)`. 
    *   *Effect*: Company A can have a "Manager" and Company B can have a "Manager", but Company A cannot have two "Manager" roles.

## Application Properties

### AWS Cognito Integration
This module is the primary writer to the `horaion.cognito.user-pool-id`.

{% hint style="success" %}
**Tip / Success:**
**Minimal IAM Policy**: To follow the principle of least privilege, attach this policy to the ECS Task Role or EC2 Instance Profile running the backend.
{% endhint %}

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cognito-idp:CreateGroup",
                "cognito-idp:GetGroup",
                "cognito-idp:AdminAddUserToGroup"
            ],
            "Resource": "arn:aws:cognito-idp:us-east-1:123456789012:userpool/us-east-1_AbCdEfGh"
        }
    ]
}
```
