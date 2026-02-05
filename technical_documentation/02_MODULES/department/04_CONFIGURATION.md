# Department Module Configuration

## Database Configuration

The **Department Module** uses the `departments` table.

### Indexes & Performance
We explicitly index columns to support the hierarchical nature of this data:

| Index Name | Column | Purpose |
| :--- | :--- | :--- |
| `idx_departments_branch_id` | `branch_id` | **Critical**: All queries are scoped to a branch. |
| `idx_departments_parent_id` | `parent_id` | **Tree Traversal**: Finding all children of a department. |
| `idx_departments_cost_center` | `cost_center` | Financial reporting aggregation. |

## Application Properties

This module interacts heavily with external services but requires no specific configuration of its own.

### Integrations
*   **Cognito Integration**: The logic for promoting an HOD relies on the global `horaion.cognito` configuration found in the **Auth Module**.
