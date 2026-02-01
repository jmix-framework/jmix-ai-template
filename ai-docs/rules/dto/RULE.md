---
description: Rules for creating DTOs for API responses, reports, and non-persistent data
globs: ["**/dto/**"]
alwaysApply: false
---

# Jmix DTO Rules

## Scope
Applies to `src/main/java/**/dto/`

## When to Use What

| Use Case | Pattern |
|----------|---------|
| Jmix UI (dataGrid, forms) | `@JmixEntity` + `@JmixId` |
| REST API request/response | Plain POJO or Java Record |
| Internal service layer | Plain POJO or Java Record |

## Jmix UI DTO Pattern
```java
@JmixEntity(name = "app_CustomerSummaryDto")
public class CustomerSummaryDto {
    @JmixId
    @JmixGeneratedValue
    private UUID id;
    
    @InstanceName
    private String displayField;
    
    // getters/setters
}
```

## REST API DTO (Plain POJO / Record)
```java
// Java Record (immutable, recommended)
public record OrderRequest(UUID customerId, List<LineItem> items) {}

// Or plain POJO
public class OrderRequest {
    private UUID customerId;
    // getters/setters
}
```

## Required Annotations (UI DTOs only)
- `@JmixEntity(name = "app_DtoName")` — with prefix
- `@JmixId` on ID field (NOT `@Id` — that's for JPA entities!)
- `@InstanceName` on display field
- `@JmixProperty(mandatory=true)` for required transient fields

## Creating UI DTOs
- Use `dataManager.create(Dto.class)` (NOT `new Dto()`)

## Validation (Jakarta Bean Validation)
```java
import jakarta.validation.constraints.*;

@NotNull(message = "Required")
private UUID customerId;

@NotBlank @Size(min = 3, max = 50)
private String orderNumber;

@Valid // For nested DTOs
private List<LineItemDto> items;
```

Validate in service:
- `@Validated` on class + `@Valid` on param (auto)
- Or inject `Validator` and call `validator.validate(dto)`

## Messages Format
```properties
com.company.project.dto/CustomerSummaryDto=Customer Summary
com.company.project.dto/CustomerSummaryDto.customerName=Customer
```

## Lombok
- **Allowed on DTOs** (unlike entities)
- Prefer Records for immutable DTOs

## Forbidden
- `@Entity` / `@Table` on DTOs
- Persisting DTOs to database
- Using `new Dto()` for UI DTOs (breaks ID generation)
