---
description: Rules for creating enumerations with EnumClass interface for Jmix
globs: ["**/entity/**"]
alwaysApply: false
---

# Jmix Enum Rules

## Scope
Applies to enums in `src/main/java/**/entity/`

## Enum Pattern
```java
import io.jmix.core.metamodel.datatype.EnumClass;
import org.springframework.lang.Nullable;

public enum OrderStatus implements EnumClass<String> {
    NEW("NEW"),
    PROCESSING("PROCESSING"),
    DONE("DONE");
    
    private final String id;
    
    OrderStatus(String id) { this.id = id; }
    
    @Override
    public String getId() { return id; }
    
    @Nullable
    public static OrderStatus fromId(String id) {
        for (OrderStatus s : values()) {
            if (s.getId().equals(id)) return s;
        }
        return null;
    }
}
```

## Required
- Implement `EnumClass<String>` or `EnumClass<Integer>`
- `getId()` method returning stored value
- Static `fromId()` with `@Nullable` return (or use `EnumUtils`)
- Import: `org.springframework.lang.Nullable`

## Alternative: EnumUtils.fromId()
Instead of custom `fromId()` method, use Jmix utility:
```java
import io.jmix.core.metamodel.datatype.EnumUtils;

// EnumUtils.fromId(TargetEnum.class, storedStringValue)
OrderStatus status = EnumUtils.fromId(OrderStatus.class, this.status);
```

## Entity Field Pattern
```java
@Column(name = "STATUS")
private String status; // Store ID, not enum!

public OrderStatus getStatus() {
    return status == null ? null : OrderStatus.fromId(status);
}

public void setStatus(OrderStatus status) {
    this.status = status == null ? null : status.getId();
}
```

## Liquibase Column Type
- `EnumClass<String>` → `varchar(50)`
- `EnumClass<Integer>` → `int`

## Messages Format
```properties
com.company.project.entity/OrderStatus=Order Status
com.company.project.entity/OrderStatus.NEW=New
com.company.project.entity/OrderStatus.PROCESSING=Processing
com.company.project.entity/OrderStatus.DONE=Done
```

## Forbidden
- Plain enums without `EnumClass` (breaks UI localization)
- `@Convert` / `AttributeConverter` (use EnumClass instead)
- Storing enum `.name()` directly (refactoring breaks data)
- Missing `@Nullable` on `fromId()` return
