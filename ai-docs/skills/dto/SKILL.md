---
name: dto
description: Creating DTOs — Jmix UI DTOs with @JmixEntity/@JmixId vs plain POJOs/Records for REST APIs, and validation patterns.
---

# DTOs (Data Transfer Objects)

## When to Use
- When creating non-persistent data structures for UI
- When building REST API request/response objects
- When you need to aggregate data from multiple entities

## When to Use What

| Use Case | Pattern |
|----------|---------|
| Jmix UI (dataGrid, forms) | `@JmixEntity` + `@JmixId` |
| REST API request/response | Plain POJO or Java Record |
| Internal service layer | Plain POJO or Java Record |

## Java Records (REST/immutable)
```java
public record OrderRequest(UUID customerId, List<LineItem> items) {}
public record CustomerSummary(String name, int orderCount, BigDecimal total) {}
```

## Jmix UI DTO Template
```java
@JmixEntity(name = "app_CustomerSummaryDto")  // Always use prefix!
public class CustomerSummaryDto {

    @JmixId
    @JmixGeneratedValue
    private UUID id;

    @InstanceName
    private String customerName;

    private Integer orderCount;
    private BigDecimal totalAmount;

    // Getters and setters
}
```

## Key Annotations

| Annotation | Purpose |
|------------|---------|
| `@JmixEntity` | Registers DTO in Jmix metadata |
| `@JmixId` | Marks ID field (NOT `@Id`!) |
| `@JmixGeneratedValue` | Auto-generates UUID |
| `@InstanceName` | Display name in UI |
| `@JmixProperty(mandatory=true)` | Required non-persistent attribute |

## Creating UI DTOs
```java
// CORRECT — use DataManager
CustomerSummaryDto dto = dataManager.create(CustomerSummaryDto.class);

// WRONG — new breaks ID generation
CustomerSummaryDto dto = new CustomerSummaryDto(); // ❌
```

## Validation
```java
import jakarta.validation.constraints.*;

public class OrderRequest {
    @NotNull(message = "Customer is required")
    private UUID customerId;

    @NotBlank @Size(min = 3, max = 50)
    private String orderNumber;

    @Valid  // Validate nested
    private List<LineItemDto> items;
}

// In service:
@Validated
public class OrderService {
    public void createOrder(@Valid OrderRequest request) { ... }
}
```

## Messages
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
- `new Dto()` for UI DTOs (use `dataManager.create()`)
