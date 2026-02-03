# Rule Module

## Overview

The Rule module enables dynamic business rule creation using templated sentences with configurable field options. Rules are defined at the department level and support both static and dynamic field options through a provider-based architecture.

**Location**: [`modules/rule/`](../../src/main/java/com/horaion/app/modules/rule/)

**Comprehensive Documentation**: See [`../rule-module.md`](../rule-module.md) for complete technical details.

## Key Features

- **Template-Based Sentences**: Rules with placeholder variables (e.g., `{roles}`, `{shifts}`)
- **Field Configuration**: Define field types, validation, and data sources
- **Provider System**: Dynamic options from databases or APIs
- **Field Templates**: Reusable field configurations
- **Answer Storage**: User-submitted values as JSON

## Quick Example

### Sentence Template
```
"There {requirement_type} be {quantity_type} {number_of_employees} employees with roles {roles} assigned to {shifts} shift"
```

### Field Types
- **Static Fields**: Options in `sourceConfig.option` (e.g., "must", "should", "may")
- **Dynamic Fields**: Options from providers (e.g., `roleOptionProvider`, `shiftOptionProvider`)

### Result
> "There **must** be **at least** **2** employees with roles **Nurse, Supervisor** assigned to **Morning** shift"

## API Endpoints

| Endpoint Group | Description |
|----------------|-------------|
| **Rules** | CRUD operations for rules |
| **Rule Fields** | Manage field definitions |
| **Rule Answers** | Store and retrieve user answers |
| **Rule Field Templates** | Reusable field configurations |

**See**: [`../rule-module.md`](../rule-module.md) for complete API reference and implementation details.

## Provider System

### Available Providers

- **roleOptionProvider** / **role_list**: Fetches employee roles from database
- **shiftOptionProvider** / **shift_list**: Fetches shifts for department
- **employeeOptionProvider**: Fetches employees with branch filtering

### Adding Custom Providers

```java
@Component("customProvider")
public class CustomOptionProvider implements FieldOptionProvider {
    @Override
    public Map<String, Object> getOptions(UUID companyId, UUID branchId, 
                                         UUID departmentId, Map<String, Object> sourceConfig) {
        // Fetch and return options
    }
}
```

## Related Documentation

- **[Complete Rule Module Documentation](../rule-module.md)** - Comprehensive technical guide
- [Department Module](./04-department-module.md) - Rule container
- [Schedule Module](./09-schedule-module.md) - Uses rules for generation
