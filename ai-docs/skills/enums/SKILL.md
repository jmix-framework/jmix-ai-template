---
name: enums
description: Creating enumerations with EnumClass interface for proper Jmix integration and localization.
---

# Enums

## When to Use
- When creating enumeration for entity fields
- When you need localized enum display in UI

## Required Imports
```java
import io.jmix.core.metamodel.datatype.EnumClass;
import org.springframework.lang.Nullable;
```

## Enum Template
```java
public enum OrderStatus implements EnumClass<String> {

    NEW("NEW"),
    CONFIRMED("CONFIRMED"),
    PROCESSING("PROCESSING"),
    SHIPPED("SHIPPED"),
    DELIVERED("DELIVERED"),
    CANCELLED("CANCELLED");

    private final String id;

    OrderStatus(String id) {
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    @Nullable
    public static OrderStatus fromId(String id) {
        for (OrderStatus status : values()) {
            if (status.getId().equals(id)) {
                return status;
            }
        }
        return null;
    }
}
```

## Alternative: EnumUtils.fromId()
```java
import io.jmix.core.metamodel.datatype.EnumUtils;

// Instead of custom fromId():
OrderStatus status = EnumUtils.fromId(OrderStatus.class, storedValue);
```

## Entity Field Pattern
```java
@Column(name = "STATUS")
private String status;  // Store ID, not enum!

public OrderStatus getStatus() {
    return status == null ? null : OrderStatus.fromId(status);
}

public void setStatus(OrderStatus status) {
    this.status = status == null ? null : status.getId();
}
```

## Liquibase Column
```xml
<!-- For EnumClass<String> -->
<column name="STATUS" type="varchar(50)"/>

<!-- For EnumClass<Integer> -->
<column name="STATUS" type="int"/>
```

## Messages (Localization)
```properties
com.company.project.entity/OrderStatus=Order Status
com.company.project.entity/OrderStatus.NEW=New
com.company.project.entity/OrderStatus.CONFIRMED=Confirmed
com.company.project.entity/OrderStatus.PROCESSING=Processing
com.company.project.entity/OrderStatus.SHIPPED=Shipped
com.company.project.entity/OrderStatus.DELIVERED=Delivered
com.company.project.entity/OrderStatus.CANCELLED=Cancelled
```

## Checklist
- [ ] Implements `EnumClass<String>` or `EnumClass<Integer>`
- [ ] `getId()` method
- [ ] Static `fromId()` with `@Nullable`
- [ ] Messages for localization
- [ ] Entity field stores ID (not enum directly)

## Forbidden
- Plain enums without `EnumClass`
- `@Convert` / `AttributeConverter`
- Storing enum `.name()` directly
- Missing `@Nullable` on `fromId()` return
