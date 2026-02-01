---
description: Rules for creating business logic services with proper Jmix patterns
globs: ["**/service/**", "**/repository/**"]
alwaysApply: false
---

# Jmix Services Rules

## Scope
Applies to `src/main/java/**/service/` and `src/main/java/**/repository/`

## Naming Conventions
- **Bean Names:** MUST use prefix (e.g., `app_` or `mod_`).
  - Example: `@Service("app_OrderService")`
  - Example: `@Component("app_UserListener")`
  - Example: `@Bean("app_MyBean")`
- **Stereotypes:**
  - `@Service`: Pure business logic / Domain services.
  - `@Component`: Infrastructure, Listeners, Utils.

## Service Pattern
```java
@Service("app_EntityService")
public class EntityService {
    private final DataManager dataManager;
    
    public EntityService(DataManager dataManager) {
        this.dataManager = dataManager;
    }
}
```

## Data Access
### DataManager (Primary)
- Create: `dataManager.create(Entity.class)`
- Load: `dataManager.load(Entity.class).id(id).one()`
- Query: `dataManager.load(Entity.class).query("...").list()`
- Save: `dataManager.save(entity)`
- Remove: `dataManager.remove(entity)`

### JmixDataRepository (Alternative)
- Extend `JmixDataRepository<Entity, ID>`
- Pass `FetchPlan` as the **last parameter** to avoid N+1 issues.
```java
List<User> findByEmail(String email, FetchPlan fetchPlan);
```

## Transactions
- `@Transactional` for write operations
- `@Transactional(readOnly = true)` for read-only
- `TransactionTemplate` for programmatic control (when AOP fails)

## Forbidden
- `EntityManager` without `@Transactional`
- Business logic in views
- Raw `JpaRepository` (use `JmixDataRepository`)
