# 04 - Static Templates

> **Non-Code Assets: Excel and HTML**

---

## 4.1 Overview

The `src/main/resources/templates` directory contains static files that are read by the application at runtime. These are typically used for generating outputs (reports) or inputs (import parsing).

---

## 4.2 Excel Import Templates

**File**: `src/main/resources/templates/Employee_Import_Template.xlsx`

**Purpose**:
This file serves as the "Master Copy" for the Bulk Employee Import feature.

### Usage Flow
1.  **Download**: The Frontend calls `GET /api/v1/employees/template`.
2.  **Read**: The Backend reads this file from the `classpath`.
3.  **Serve**: The file bytes are streamed back to the user's browser.

### Why store it here?
Storing the template in the backend ensures that the **validation logic** (in `ExcelService`) and the **template structure** (columns, headers) are always in sync. If we stored the template in the Frontend (React code), we might change the backend parser but forget to update the frontend file, causing errors.

{% hint style="warning" %}
**Warning:** If you modify this Excel file, ensure you also update the `EmployeeImportDto` and the parsing logic in `ExcelService` to match the new column order/names.
{% endhint %}
