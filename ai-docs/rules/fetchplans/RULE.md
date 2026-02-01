---
description: Rules for configuring fetch plans and data loading in Jmix
alwaysApply: false
---

# Jmix Fetch Plans Rules

## Scope
Applies to data loading operations

## Built-in Plans
- `_local` / `FetchPlan.LOCAL` — all local attributes (non-references)
- `_instance_name` / `FetchPlan.INSTANCE_NAME` — ONLY @InstanceName field(s)!
- `_base` / `FetchPlan.BASE` — `_local` + `_instance_name` + embedded (use by default)

## ⚠️ CUBA Migration
CUBA `_minimal` ≠ Jmix `_instance_name`!
- CUBA `_minimal` → use Jmix `_base` (loaded all local attrs)
- Jmix `_instance_name` loads ONLY @InstanceName field

## Beans
- `FetchPlans` — builder for creating fetch plans programmatically
- `FetchPlanRepository` — retrieve named plans from XML

## Fetch Modes
- `AUTO` — framework decides (default)
- `JOIN` — single query with SQL JOIN (to-one)
- `BATCH` — separate query (collections)

## XML Definition (fetch-plans.xml)
```xml
<fetch-plan entity="Order" name="order-with-lines" extends="_base">
    <property name="customer" fetchPlan="_instance_name" fetchMode="JOIN"/>
    <property name="lines" fetchPlan="_base" fetchMode="BATCH"/>
</fetch-plan>
```

## Programmatic (Constructor Injection)
```java
private final FetchPlans fetchPlans;
private final DataManager dataManager;

public MyService(FetchPlans fetchPlans, DataManager dataManager) {
    this.fetchPlans = fetchPlans;
    this.dataManager = dataManager;
}
```

## JmixDataRepository
```java
List<Order> findByStatus(OrderStatus status, FetchPlan fetchPlan);
```

## Best Practices
- List views: `_instance_name` for references
- Detail views: `_base` or custom with collections
- Always `FetchType.LAZY` on entities
- Max depth: 2-3 levels
- Check logs for N+1 queries
- Only load what's displayed in UI

## Virtual Lists
- Plan buffering strategy for infinite scroll
- Use `firstResult`/`maxResults` for pagination
- Think about lazy loading before implementation

## Forbidden
- `FetchType.EAGER` on entities
- Loading without fetch plan in loops
- Deep nesting (>3 levels) without reason
- Using fetch plans as security mechanism
