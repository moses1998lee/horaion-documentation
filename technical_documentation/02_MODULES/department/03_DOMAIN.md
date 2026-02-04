# Department Module Domain Logic

## Service: `DepartmentService`

Manages the logical organization of the workforce.

### Hierarchical Structure

Departments are designed to be recursive.
*   **Flat Structure**: `Branch -> [Dept A, Dept B, Dept C]`
*   **Nested Structure**: `Branch -> [Dept A -> [Sub-Dept A1, Sub-Dept A2]]`

This allows large clients (like Hypermarkets) to model "Perishables" as a parent department, with "Bakery" and "Produce" as children, sharing a common `Cost Center` or `Budget`.

### Core Flows

#### 1. Creation & Validation
**Goal**: Ensure clean data structure.

**Unique Constraint**: `(branch_id, department_code)`
*   You can have `Sales (SAL)` in Branch A and `Sales (SAL)` in Branch B.
*   You **cannot** have two `Sales (SAL)` departments in Branch A.

#### 2. Head of Department (HoD) Assignment
**Goal**: Link leadership and automate access rights.

**Logic**:
1.  **Validate**: Employee exists and belongs to the company.
2.  **Update DB**: Set `head_of_dept_id` column.
3.  **Side Effect (Security)**:
    *   Calls `CognitoGroupService`.
    *   Adds user to `PRIVILEGED_SYSTEM_USER` group.
    *   **Result**: Next time the user logs in, their JWT token will have the `MANAGER` claim.

### Entities

#### `Department` (Database Table: `departments`)

*   **Keys**:
    *   `id` (UUID, PK)
    *   `branch_id` (FK): The physical location.
    *   `parent_id` (FK): Optional parent department.
*   **Fields**:
    *   `name`: Display name.
    *   `code`: Short identifier (e.g., `IT`, `HR`).
    *   `cost_center`: Financial reporting code.
    *   `budget`: `BigDecimal` value for labor cost planning.

{% hint style="info" %}
**Note:**
**Indexes**: We index `branch_id` (for listing), `department_code` (for search), and `parent_id` (for tree traversal) to ensure performance remains high even with deep hierarchies.
{% endhint %}

### Frontend Integration Guide

Since departments can be nested indefinitely, rendering them in a flat dropdown or a tree requires recursion.

#### TypeScript Helpers

```typescript
interface Department {
  id: string;
  name: string;
  parentId?: string;
  children?: Department[]; // Populated by the helper below
}

/**
 * Transforms a flat list of departments into a Tree Structure.
 * Useful for "Tree Select" components.
 */
function buildDepartmentTree(departments: Department[]): Department[] {
  const map: Record<string, Department> = {};
  const tree: Department[] = [];

  // Initialize map
  departments.forEach(d => {
    map[d.id] = { ...d, children: [] };
  });

  // Build tree
  departments.forEach(d => {
    if (d.parentId && map[d.parentId]) {
      map[d.parentId].children!.push(map[d.id]);
    } else {
      tree.push(map[d.id]);
    }
  });

  return tree;
}

/**
 * Generates breadcrumbs for a specific department.
 * Example: "Engineering > Mobile > iOS"
 */
function getBreadcrumbs(targetId: string, allDepartments: Department[]): string[] {
  const map = new Map(allDepartments.map(d => [d.id, d]));
  const breadcrumbs: string[] = [];
  
  let current = map.get(targetId);
  while (current) {
    breadcrumbs.unshift(current.name);
    current = current.parentId ? map.get(current.parentId) : undefined;
  }
  
  return breadcrumbs;
}
```
