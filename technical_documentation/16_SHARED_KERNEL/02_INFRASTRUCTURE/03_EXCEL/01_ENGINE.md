# 01 - Excel Engine

> **Apache POI Wrapper for Data Import**

---

## Overview

The Excel Engine simplifies reading data from `.xlsx` files. It wraps the low-level **Apache POI** library to provide a safe, row-by-row reading capability.

**Reader**: `com.horaion.app.shared.infrastructure.excel.helpers.ExcelWorkbookReader`

---

## Architecture
The engine is designed to handle "User Uploads" (e.g., Employee Lists, Shift Templates).

### Key Features
1.  **Row Iteration**: Safely iterates through rows, skipping empty ones.
2.  **Cell Safekeeping**: `ExcelCellUtils` handles the notorious complexity of Excel cell types (Numeric vs String vs Formula). It automatically converts everything to the expected string representation.
3.  **Strict Validation**: Stops immediately if the file format is invalid.

---

## Usage Example

```java
public List<EmployeeImportDTO> parse(InputStream fileStream) {
    ExcelWorkbookReader reader = new ExcelWorkbookReader(fileStream);
    List<EmployeeImportDTO> results = new ArrayList<>();

    // Skip Header (Row 0)
    for (Row row : reader.getSheet(0)) {
        if (row.getRowNum() == 0) continue;

        String name = ExcelCellUtils.getString(row, 0); // Column A
        String email = ExcelCellUtils.getString(row, 1); // Column B
        
        results.add(new EmployeeImportDTO(name, email));
    }
    
    return results;
}
```

{% hint style="danger" %}
**Critical:** Excel dates are actually stored as numbers! Used `ExcelCellUtils.getDate(row, col)` to correctly parse them into `LocalDate`, otherwise you will get a raw number like `45231.0`.
{% endhint %}
