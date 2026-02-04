# API Reference

{% hint style="info" %}
**Note:**
**Base Path**: `/api/v1/companies/{companyId}/departments/{departmentId}/demand-forecasts`
All endpoints require both `companyId` and `departmentId`.
{% endhint %}

## Controller: `DemandForecastController`

### 1. Get All Forecasts (Paginated)

**Endpoint**: `GET .../demand-forecasts`

Retrieves a history of forecasts for a department.

*   **Parameters**: `page`, `size`.
*   **Success Response** (`200 OK`): `Page<DemandForecastResponse>`.

### 2. Get Forecast by Date

**Endpoint**: `GET .../demand-forecasts/date/{date}`

Finds the specific plan for a single calendar day.

*   **Parameters**:
    *   `date`: YYYY-MM-DD (e.g., `2024-12-25`).
*   **Success Response** (`200 OK`): Returns the complex timeblock structure for that day.

### 3. Get Forecast by Date Range

**Endpoint**: `GET .../demand-forecasts/range`

Useful for fetching a full week's worth of demand to display on a calendar view.

*   **Parameters**:
    *   `startDate`: YYYY-MM-DD.
    *   `endDate`: YYYY-MM-DD.

### 4. Create / Upsert Forecast

**Endpoint**: `POST .../demand-forecasts`

Creates a new forecast. If a forecast already exists for this date, the logic inside `DemandForecastService` will intelligently **Update** it instead of crashing.

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
        }
      ]
    }
    ```
    {% hint style="success" %}
    **Tip / Success:**
    **Index 17?** That means **08:30 AM**.
    Formula: `(8 * 2) + 1 = 17`.
    {% endhint %}

### 5. Update Forecast

**Endpoint**: `PUT .../demand-forecasts/{id}`

Updates metadata or demand values.

### 6. Soft Delete

**Endpoint**: `DELETE .../demand-forecasts/{id}/soft`

Marks the forecast as deleted but keeps it for historical data training.

{% hint style="danger" %}
**Critical:**
**Hard Delete**: There is also a `DELETE /{id}` endpoint, but it completely wipes the record. Use Soft Delete whenever possible to preserve data for AI training usage.
{% endhint %}

## Exception Handling

Mapped in `DemandForecastExceptionHandler` (generic handler).

| Exception | HTTP Status | Description |
| :--- | :--- | :--- |
| `DemandForecastNotFoundException` | `404 Not Found` | ID or Date not found. |
| `DepartmentNotFoundException` | `404 Not Found` | Invalid Department ID. |
