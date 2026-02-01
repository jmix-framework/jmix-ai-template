---
description: Rules for configuring access control with Resource Roles and Row-Level Roles
globs: ["**/security/**"]
alwaysApply: false
---

# Jmix Security Rules

## Scope
Applies to `src/main/java/**/security/`

## Resource Role Pattern
```java
import io.jmix.security.model.SecurityScope;
import io.jmix.securityflowui.role.annotation.MenuPolicy; // Note package!
import io.jmix.securityflowui.role.annotation.ViewPolicy; // Note package!

@ResourceRole(name = "Role Name", code = RoleName.CODE, scope = SecurityScope.UI)
public interface RoleName {
    // ...
```

## Policies
- `@EntityPolicy` — CRUD operations
- `@EntityAttributePolicy` — Attribute access (MODIFY/VIEW). Hidden by omission.
- `@ViewPolicy` — Screen access (`io.jmix.securityflowui...`)
- `@MenuPolicy` — Menu item visibility (`io.jmix.securityflowui...`)
- `@SpecificPolicy` — Custom permissions

## Row-Level Security
- `@RowLevelRole` + `@JpqlRowLevelPolicy`
- Use `{E}` for entity alias
- Variables: `:current_user_username`

## Permission Checks in Code
- Use `AccessManager` with contexts (e.g., `EntityOperationContext`, `SpecificOperationAccessContext`).
- Call `accessManager.applyRegisteredConstraints(ctx)`.
- Check `ctx.isPermitted()`.
- DO NOT invent methods like `isEntityOpPermitted`.
