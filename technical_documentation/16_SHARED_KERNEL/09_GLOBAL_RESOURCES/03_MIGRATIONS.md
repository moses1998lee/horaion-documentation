# 03 - Database Migrations

> **Schema Management with Flyway**

---

## 3.1 Strategy overview

Horaion uses **Flyway** to manage database schema changes. This essentially treats the database structure as code ("Infrastructure as Code").

*   **Format**: SQL-based migrations.
*   **Location**: `src/main/resources/db/migrations`
*   **Activation**: Runs automatically on application startup.

{% hint style="success" %}
**Success:** Using Flyway ensures that new developers just need to run `docker-compose up`, and the database automatically builds itself from scratch to the latest version. No manual SQL scripts required.
{% endhint %}

---

## 3.2 Naming Convention

We follow the standard Flyway V-naming convention:

`V{Version}__{Description}.sql`

*   **V**: Prefix (mandatory).
*   **{Version}**: Sequential number (e.g., `1`, `2`, `30`). Or timestamp.
*   **__**: Double underscore separator (critical).
*   **{Description}**: Human-readable name (underscores instead of spaces).

### Example Sequence
1.  `V1__create_notification_records.sql`
2.  `V2__create_companies.sql`
..
30. `V30__rename_status_to_job_status_in_schedules.sql`

---

## 3.3 Key Migrations Breakdown

### Core Structure (V1-V9)
Sets up the fundamental tenant hierarchy:
*   `V2__create_companies.sql`: The root tenant table.
*   `V3__create_branches.sql`: Links to Companies.
*   `V4__create_departments.sql`: Links to Branches.
*   `V6__create_employees.sql`: The main user entity.

### Feature Implementation (V10-V20)
Adds specific module capabilities:
*   `V10__create_rules.sql`: Rule engine storage.
*   `V14__create_schedule_module_tables.sql`: A massive migration creating the core Scheduling tables.

### Refactoring & Fixes (V20+)
Shows the evolution of the project:
*   `V17__make_import_jobs_branch_id_nullable.sql`: A fix to allow company-wide imports.
*   `V30__rename_status_to_job_status_in_schedules.sql`: Fixing a column name collision.

{% hint style="info" %}
**Note:** You must **never** modify an existing migration file after it has been merged/deployed. If you need to fix a mistake in `V5`, you create `V31__fix_v5_mistake.sql`. Modifying `V5` will cause check-sum errors on other developers' machines.
{% endhint %}
