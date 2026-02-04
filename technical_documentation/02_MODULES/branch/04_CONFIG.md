# Branch Module Configuration

## Database Configuration

The Branch module shares the correct database connection with the rest of the application. No specific separate database configuration is required.

## Application Properties

There are no unique external API keys (like AWS Cognito keys) for the Branch module. It relies purely on:

1.  **PostgreSQL**: For storing branch data.
2.  **Security Context**: For validating user permissions (`@PermitCheck`).

{% hint style="success" %}
**Tip / Success:**
Since this module is purely internal (CRUD logic), it is one of the easiest to deploy. It has zero external dependencies on 3rd party APIs.
{% endhint %}
