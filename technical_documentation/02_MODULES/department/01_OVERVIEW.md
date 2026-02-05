# Department Module

| Attribute     | Details                                       |
| :------------ | :-------------------------------------------- |
| **Namespace** | `com.horaion.app.modules.department`          |
| **Status**    | 🟢 Stable                                     |
| **Criticality** | Medium (Organizational Structure)             |
| **Dependencies** | Branch Module, Employee Module                |

## Executive Summary

The **Department Module** allows companies to organize their workforce into functional units. Unlike a **Branch** (which is a physical location), a **Department** is a logical team (e.g., "Sales", "IT", "Bakery").

{% hint style="success" %}
**Tip / Success:**
**Hierarchy**: Departments can be nested! You can have a "Sales" department that contains "Direct Sales" and "Online Sales" sub-departments. This allows for complex organizational modeling.
{% endhint %}

{% hint style="info" %}
**Note:**
**Context**: A department *must* belong to a specific Branch. A "Sales" department in the Cape Town branch is distinct from the "Sales" department in the Durban branch.
{% endhint %}

## Hierarchy & Relationships

The Department sits below the Branch in the organizational tree.

    SubDept2 --> |Assigned| Emp2[Employee: Bob]
```

> **Diagram Explanation**: This hierarchy demonstrates the **Recursive Departmental Model**. Unlike the fixed Branch structure, Departments can be nested to match any corporate organizational chart. This allows for both "Wide" (many flat departments) and "Deep" (multiple sub-layers) structures, all anchored to a single physical Branch.

### Visual Example: Breadcrumbs
In the frontend UI, this hierarchy translates to easy navigation:
> **Cape Town** > **Engineering** > **Backend Team**

## Core Capabilities

1.  **Organizational Modeling**:
    *   Defining the functional structure of a location.
    *   Supporting nested (Parent/Child) relationships.
2.  **Leadership Management**:
    *   Assigning a **Head of Department (HOD)**.
    *   *Side Effect*: assigning an HOD automatically grants them elevated system privileges (Cognito Group update).
3.  **Financial Metadata**:
    *   Tracking `Cost Center` codes and `Budget` allocations for reporting.

## Responsibilities

*   **Validation**: Ensuring Department Codes are unique within a Branch.
*   **Access Control**: Linking Employees to Departments determines what data they can see (e.g., a Manager sees shifts for their own department).

## Module Architecture

    DepartmentRepository --> DB[(Database)]
```

> **Diagram Explanation**: The Department Module manages the internal structure of a Branch. The **DepartmentService** coordinates with the **BranchRepository** to ensure parent-child relationships remain valid and with the **Security Context** to handle privileged access assignments (HoD).
