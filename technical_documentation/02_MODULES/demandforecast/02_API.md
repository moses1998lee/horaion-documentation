# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1/companies/{companyId}/departments/{departmentId}/demand-forecasts`
**Context**: All endpoints are scoped to a specific `Department`. You cannot fetch a forecast "globally" without knowing which department it belongs to.
{% endhint %}

## Controller: `DemandForecastController`

### 1. Get All Forecasts (Paginated)

**Endpoint**: `GET .../demand-forecasts`

Retrieves a history of forecasts for a department.

*   **Query Parameters**:

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `page` | `int` | `0` | Page number. |
| `size` | `int` | `10` | Items per page. |

### 2. Get Forecast by Date

**Endpoint**: `GET .../demand-forecasts/date/{date}`

Finds the specific plan for a single calendar day.

*   **Path Parameters**:
    *   `date`: YYYY-MM-DD (e.g., `2024-12-25`).
*   **Success Response** (`200 OK`): Returns the complex timeblock structure for that day.
*   **Error Responses**:
    *   `404 Not Found`: If no plan exists for this specific date.

### 3. Get Forecast by Date Range

**Endpoint**: `GET .../demand-forecasts/range`

**Use Case**: Fetching a full week's worth of demand to display on the Weekly Calendar View.

*   **Query Parameters**:
    *   `startDate`: YYYY-MM-DD.
    *   `endDate`: YYYY-MM-DD.

### 4. Create / Upsert Forecast

**Endpoint**: `POST .../demand-forecasts`

Creates a new forecast.

{% hint style="success" %}
**Tip / Success:**
**Idempotency**: If a forecast already exists for this date, the service logic will intelligently **Update** it instead of crashing with a "Duplicate" error. This makes the frontend logic much simpler (just "Save", don't worry about "Create vs Edit").
{% endhint %}

*   **Request Body**: `CreateDemandForecastRequest`
    ```json
    {
      "date": "2024-12-25",
      "forecastSource": "manual",
      "timeblocks": [
        {
          "index": 17,
          "demands": [
            { "roleId": "uuid-for-cashier", "demandValue": 5 },
            { "roleId": "uuid-for-manager", "demandValue": 1 }
          ]
        },
        {
          "index": 18,
          "demands": [
             { "roleId": "uuid-for-cashier", "demandValue": 5 }
          ]
        }
      ]
    }
    ```
    *   **Logic**:
        *   `index: 17` = **08:30**. 5 Cashiers, 1 Manager.
        *   `index: 18` = **09:00**. 5 Cashiers. (Manager went home?).

### 5. Update Forecast

**Endpoint**: `PUT .../demand-forecasts/{id}`

Updates metadata or demand values for an existing ID.

### 6. Soft Delete

**Endpoint**: `DELETE .../demand-forecasts/{id}/soft`

Marks the forecast as deleted but keeps it for historical data training.

{% hint style="danger" %}
**Critical:**
**Hard Delete vs Soft Delete**:
*   `DELETE /{id}/soft`: **Preferred**. Sets `is_deleted=true`. Preserves history for AI.
*   `DELETE /{id}`: **Destructive**. Wipes the record from the database. Use only if a user made a mistake and wants to "Undo" a creation immediately.
{% endhint %}

## Exception Handling

Mapped in `DemandForecastExceptionHandler`.

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `DemandForecastNotFoundException` | `404 Not Found` | ID or Date not found. |
| `DepartmentNotFoundException` | `404 Not Found` | Invalid Department ID provided. |
