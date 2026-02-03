# Demand Forecast Module

## Overview

The Demand Forecast module manages workforce demand planning by role and time period. It provides forecasts used by the schedule generation engine to optimize staffing levels.

**Location**: [`modules/demandforecast/`](../../src/main/java/com/horaion/app/modules/demandforecast/)

## Key Features

- **Role-Based Forecasting**: Demand forecasts per employee role
- **Date Range Planning**: Forecasts for specific time periods
- **Schedule Integration**: Used by external engine for optimization
- **Department Scoping**: Forecasts are department-specific

## Data Model

### DemandForecast Entity

```java
@Entity
@Table(name = "demand_forecasts")
public class DemandForecast {
    private UUID id;
    private Company company;
    private Department department;
    private LocalDate startDate;
    private LocalDate endDate;
    private Map<String, Object> roleDemands;  // JSONB: role-based demands
    private Boolean isActive;
}
```

## API Endpoints

**Base**: `/api/v1/companies/{companyId}/departments/{deptId}/demand-forecasts`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/demand-forecasts` | read:demand-forecast | List forecasts (paginated) |
| GET | `/demand-forecasts/{id}` | read:demand-forecast | Get forecast by ID |
| POST | `/demand-forecasts` | create:demand-forecast | Create forecast |
| PUT | `/demand-forecasts/{id}` | update:demand-forecast | Update forecast |
| DELETE | `/demand-forecasts/{id}` | delete:demand-forecast | Delete forecast |

## Example Forecast

```json
{
  "startDate": "2024-01-01",
  "endDate": "2024-01-31",
  "roleDemands": {
    "nurse": { "min": 5, "max": 8, "optimal": 6 },
    "supervisor": { "min": 1, "max": 2, "optimal": 1 },
    "technician": { "min": 2, "max": 4, "optimal": 3 }
  }
}
```

## Related Documentation

- [Schedule Module](./09-schedule-module.md) - Uses forecasts for optimization
- [Department Module](./04-department-module.md) - Forecast container
- See [`docs_suggestion.md`](../docs_suggestion.md) for comprehensive details
