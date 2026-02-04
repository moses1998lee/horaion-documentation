# Company Module Configuration

## Database Configuration

The Company module uses the primary application database. It defines the core `companies` table.

## Application Properties

This module does not require specific external API configurations. It is self-contained within the application's core logic.

{% hint style="info" %}
**Note:**
While it has no specific properties, this module is **critical** for the configuration of other modules. For example, the `auth` module often needs to look up a company to validate a user's login context.
{% endhint %}
