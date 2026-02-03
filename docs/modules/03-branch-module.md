# Branch Module

## Overview

The Branch module manages physical and virtual locations within companies. Branches represent office locations, warehouses, retail stores, or virtual divisions with geographic coordinates and operational details.

**Location**: [`modules/branch/`](../../src/main/java/com/horaion/app/modules/branch/)

## Key Features

- **Multi-location Support**: Manage multiple branch types (HEADQUARTERS, REGIONAL, SATELLITE, WAREHOUSE, RETAIL, VIRTUAL)
- **Geographic Data**: Latitude/longitude coordinates for mapping
- **Opening Hours**: Flexible JSONB storage for variable schedules
- **Services**: Customizable service offerings per branch
- **Soft Delete**: Safe deletion with referential integrity

## Data Model

### Branch Entity

```java
@Entity
@Table(name = "branches")
public class Branch {
    private UUID id;
    private Company company;
    private String name;
    private String code;                    // Unique within company
    private BranchType type;
    private String addressLine1;
    private String addressLine2;
    private String city;
    private String state;
    private String country;
    private String postalCode;
    private BigDecimal latitude;
    private BigDecimal longitude;
    private String phoneNumber;
    private String emailAddress;
    private Employee manager;
    private String timezone;
    private Map<String, Object> openingHours;  // JSONB
    private Map<String, Object> services;      // JSONB
    private Boolean isHeadquarters;
    private Boolean isActive;
}
```

### Branch Types

```java
public enum BranchType {
    HEADQUARTERS,   // Main office
    REGIONAL,       // Regional office
    SATELLITE,      // Satellite office
    WAREHOUSE,      // Warehouse/distribution center
    RETAIL,         // Retail location
    VIRTUAL         // Virtual/remote branch
}
```

## API Endpoints

**Base**: `/api/v1/companies/{companyId}/branches`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| GET | `/branches` | read:branch | List branches by company (paginated) |
| GET | `/branches/{id}` | read:branch | Get branch by ID |
| GET | `/branches/code/{code}` | read:branch | Get by branch code |
| GET | `/branches/active` | read:branch | Get active branches only |
| GET | `/branches/search?name={term}` | read:branch | Search by name |
| POST | `/branches` | create:branch | Create new branch |
| PUT | `/branches/{id}` | update:branch | Update branch |
| DELETE | `/branches/{id}` | delete:branch | Delete branch (hard) |
| DELETE | `/branches/{id}/soft` | delete:branch | Soft delete branch |

## Related Documentation

- [Company Module](./02-company-module.md) - Parent organization
- [Department Module](./04-department-module.md) - Organizational units within branches
- [Employee Module](./05-employee-module.md) - Branch employees
- See [`docs_suggestion.md`](../docs_suggestion.md) segments for comprehensive details
