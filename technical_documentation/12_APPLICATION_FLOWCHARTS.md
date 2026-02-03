# 12 - Application Flowcharts

> **Genesis Workforce Management Platform - Application Flowcharts**

---

## Required Flowcharts

### 1. Excel Uploading Flow

**Description**:
The system allows bulk importing of employees via an Excel file. The process is designed to be robust, handling variable header names and resolving dependencies (Branch, Department, Role) dynamically.

**Header Normalization Explanation**:
> The system "normalizes" headers (stripping spaces, underscores, and converting to lowercase) to improve **User Experience**. This ensures that variations like "First Name", "firstname", or "First_Name" are all accepted as valid, preventing frustration if the user modifies the template slightly.

**Mermaid Flowchart**:
```mermaid
flowchart TD
    Start([User Uploads Excel]) --> Parse["<b>Parse Excel</b><br/>(ExcelHelper.java)"]
    Parse --> Norm{"Normalize Headers?"}
    Norm -->|Yes| MapHeaders["Map 'First Name' -> 'firstname'"]
    MapHeaders --> Validate{"Required Fields Present?"}
    
    Validate -->|No| Error[Throw ExcelParseException]
    Validate -->|Yes| ParseRows[Parse Data Rows]
    
    ParseRows --> ProcessField["<b>EmployeeFieldProcessor</b><br/>Process each cell"]
    ProcessField --> BuildObj[Build Employee Objects]
    
    BuildObj --> Service[<b>EmployeeService</b>]
    Service --> Resolve["Resolve Dependencies<br/>(Find Branch/Dept/Role IDs)"]
    Resolve --> SaveDB[(Save to Database)]
    
    SaveDB --> SyncCognito["<b>Sync with Cognito</b><br/>(Create User Accounts)"]
    
    SyncCognito --> Count[Count Success/Failures]
    Count --> Response(["<b>Return Response</b><br/>(Total, Cognito Created/Failed)"])
    
    Error --> ResponseError(["Return 400 Bad Request"])
```

**Key Steps**:
1.  **Header Normalization**: Headers are sanitized (lowercase, no symbols) to ensure matching works regardless of formatting.
2.  **Row Parsing**: `EmployeeFieldProcessor` extracts values and handles type conversion.
3.  **Dependency Resolution**: The system looks up `Branch`, `Department`, and `Role` by name. If found, their IDs are linked; otherwise, the row may fail or be skipped.
4.  **Database Save**: Valid employees are persisted to PostgreSQL.
5.  **Cognito Sync**: An immediate sync attempts to create AWS Cognito users for the new employees.
6.  **Response**: Returns a summary (e.g., "Imported 50 employees. 48 Cognito accounts created, 2 failed").

---

### 2. Employee View List & Individual

**Description**:
Standard CRUD view flows. The list view supports pagination and filtering, while the detail view fetches by ID.

**Mermaid Flowchart**:
```mermaid
flowchart TD
    User([User]) --> View{"View Type?"}
    View -->|List| ListReq["GET /api/v1/employees"]
    ListReq --> DB[("Fetch Pageable w/ Filters")]
    DB --> ListResp(["Return Page JSON"])
    
    View -->|Detail| DetailReq["GET /api/v1/employees/{id}"]
    DetailReq --> Check{"Exists?"}
    Check -->|No| 404(["Return 404 Not Found"])
    Check -->|Yes| DBDetail[(Fetch One)]
    DBDetail --> DetailResp(["Return Employee JSON"])
```

> **Diagram Explanation**: A decision tree for fetching data.

**Logic Flow**:
1.  **User Request**: Do they want to see *everyone* (List) or *one person* (Detail)?
2.  **List View**: The system gets a "Page" of 20 employees. Filters (like "Department=Sales") are applied here.
3.  **Detail View**: The system checks "Does ID 5 exist?". If yes, return it. If no, show a 404 error.

---

### 3. Employee Update Flow

**Description**:
Updates an existing employee's details.
> **Note**: This process updates the **Database Only**. It does *not* automatically sync changes (like email address) to AWS Cognito in the current version.

**Mermaid Flowchart**:
```mermaid
flowchart TD
    Start([User Edits Form]) --> Submit["PUT /api/v1/employees/{id}"]
    Submit --> Validate{"Valid DTO?"}
    Validate -->|No| 400(["Return 400 Bad Request"])
    
    Validate -->|Yes| Find{"Exist in DB?"}
    Find -->|No| 404(["Return 404 Not Found"])
    
    Find -->|Yes| CheckCode{"Code Changed?"}
    CheckCode -->|Yes| Unique{"Is Code Unique?"}
    Unique -->|No| 409(["Return 409 Conflict"])
    
    Unique -->|Yes| UpdateOps[Update Entity Fields]
    CheckCode -->|No| UpdateOps
    
    UpdateOps --> Save[(Save to DB)]
    Save --> Return(["Return Updated JSON"])
```

> **Diagram Explanation**: The safety checks performed when you edit a profile.

**Validation Steps**:
1.  **Form Check**: Did you send valid JSON? (No? -> 400 Error).
2.  **Existence Check**: Does this person actually exist? (No? -> 404 Error).
3.  **Uniqueness Check**: Did you change their Employee Code to one that someone else already has? (Yes? -> 409 Conflict).
4.  **Success**: If all clear, save to DB and return the new data.

---

### 4. Employee Create Flow (Single)

**Description**:
Creates a single employee record manually.
> **Note**: Unlike the Excel Upload, this endpoint does **not** create a Cognito user account automatically. The employee is created in the database only.

**Mermaid Flowchart**:
```mermaid
flowchart TD
    Start([User Submits Form]) --> Post["POST /api/v1/employees"]
    Post --> Validate{"Valid DTO?"}
    Validate -->|No| 400(["Return 400 Bad Request"])
    
    Validate -->|Yes| Unique{"Is Code Unique?"}
    Unique -->|No| 409(["Return 409 Conflict"])
    
    Unique -->|Yes| Resolve[Resolve Dept/Role IDs]
    Resolve --> Exists{"Reference Exists?"}
    Exists -->|No| 404(["Return 404 Not Found"])
    
    Exists -->|Yes| Save[(Save to DB)]
    Save --> Return(["Return 201 Created"])
```

> **Diagram Explanation**: Creating a new user manually (one-by-one).

**Key Difference from Excel Upload**:
*   This flow resolves relationships (Dept, Role) *immediately* and fails if they are wrong.
*   **Step 1**: Check if Employee Code is unique.
*   **Step 2**: Find the IDs for the named Department and Role.
*   **Step 3**: Save. (Note: Cognito account creation is a separate step).

---

## Additional Workflows (Optional)

### 5. Schedule Generation Flow
(Future Documentation)

### 6. Authentication Flow
(Future Documentation)



---




