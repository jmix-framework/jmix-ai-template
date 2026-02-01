---
name: security
description: Configuring access control — Resource Roles, Row-Level Roles, Entity/View/Menu policies, and programmatic permission checks.
---

# Security (Roles & Policies)

## When to Use
- When creating new roles
- When configuring entity/view/menu access
- When implementing row-level security

## Resource Role Template
```java
import io.jmix.security.model.EntityPolicyAction;
import io.jmix.security.model.EntityAttributePolicyAction;
import io.jmix.security.model.SecurityScope;
import io.jmix.security.role.annotation.*;
import io.jmix.securityflowui.role.annotation.MenuPolicy;
import io.jmix.securityflowui.role.annotation.ViewPolicy;

@ResourceRole(name = "Sales Manager", code = SalesManagerRole.CODE, scope = SecurityScope.UI)
public interface SalesManagerRole {

    String CODE = "app_SalesManager"; // Always use prefix!

    @EntityPolicy(entityClass = Customer.class, actions = EntityPolicyAction.ALL)
    @EntityAttributePolicy(entityClass = Customer.class, attributes = "*", action = EntityAttributePolicyAction.MODIFY)
    void customer();

    @ViewPolicy(viewIds = {"Customer.list", "Customer.detail"})
    void views();

    @MenuPolicy(menuIds = {"Customer.list"})
    void menu();
}
```

## Entity Policies
```java
// All operations
@EntityPolicy(entityClass = Customer.class, actions = EntityPolicyAction.ALL)

// Specific operations
@EntityPolicy(entityClass = Order.class, actions = {
    EntityPolicyAction.CREATE,
    EntityPolicyAction.READ,
    EntityPolicyAction.UPDATE
})

// Read-only
@EntityPolicy(entityClass = Product.class, actions = EntityPolicyAction.READ)
```

## Attribute Policies
```java
// Full access
@EntityAttributePolicy(entityClass = Customer.class, attributes = "*", action = EntityAttributePolicyAction.MODIFY)

// View only
@EntityAttributePolicy(entityClass = Customer.class, attributes = "*", action = EntityAttributePolicyAction.VIEW)

// Hidden attributes — simply don't include them (additive system)
```

## Row-Level Security
```java
@RowLevelRole(name = "Own Orders Only", code = "app_OwnOrdersOnly")
public interface OwnOrdersOnlyRole {

    @JpqlRowLevelPolicy(
        entityClass = Order.class,
        where = "{E}.createdBy = :current_user_username"
    )
    void orderPolicy();
}
```

## Permission Checks in Code
```java
@Autowired
private AccessManager accessManager;

public void checkAccess() {
    EntityOperationContext ctx = new EntityOperationContext(Order.class, EntityOp.UPDATE);
    accessManager.applyRegisteredConstraints(ctx);
    
    if (!ctx.isPermitted()) {
        throw new AccessDeniedException("Cannot update Order");
    }
}
```

## Checklist
- [ ] Role code with prefix (`app_`)
- [ ] Entity policies (CRUD level)
- [ ] Attribute policies (modify/view)
- [ ] View policies for screens
- [ ] Menu policies for menu items
