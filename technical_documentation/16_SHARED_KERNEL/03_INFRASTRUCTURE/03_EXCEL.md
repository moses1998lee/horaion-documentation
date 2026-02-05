# 03 - Excel Infrastructure

> **Data Import/Export Adapter**

Businesses love Excel. The `excel` package provides a high-level wrapper around the **Apache POI** library to make reading and writing `.xlsx` files painless.

---

## 1. Workbook Reader
**File**: [`ExcelWorkbookReader.java`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/infrastructure/excel/helpers/ExcelWorkbookReader.java)

Reads an uploaded file and converts it into a list of Java Objects.

### Features
*   **Header Normalization**: Converts "First Name", "first_name", and "FIRST NAME" all to a canonical key `firstname`. This makes the import robust against user formatting errors.
*   **Type Safety**: Handles the conversion of Excel "General" cells to Java `String`, `Integer`, or `Double`.

---

## 2. Cell Utils
**File**: [`ExcelCellUtils.java`](file:///home/moses/genesis/api_app/horaion/src/main/java/com/horaion/app/shared/infrastructure/excel/helpers/ExcelCellUtils.java)

Helper methods to extract value from a Cell safely, avoiding `NullPointerException` if a cell is blank.

{% hint style="success" %}
**Tip:** Use this utility instead of calling `cell.getStringCellValue()` directly, as the latter throws an exception if the cell type is numeric (e.g., a phone number stored as a number).
{% endhint %}
