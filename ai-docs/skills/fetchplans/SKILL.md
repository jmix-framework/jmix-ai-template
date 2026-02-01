---
name: fetchplans
description: Configuring what data to load — built-in plans (_base, _local), custom plans, N+1 problem prevention, and fetch modes.
---

# Fetch Plans

## When to Use
- When loading entities with relationships
- When optimizing data loading (N+1 problem)
- When you need specific attributes loaded

## Built-in Fetch Plans

| Name | Constant | Description |
|------|----------|-------------|
| `_local` | `FetchPlan.LOCAL` | All local attributes (non-references) |
| `_instance_name` | `FetchPlan.INSTANCE_NAME` | ONLY `@InstanceName` field(s) |
| `_base` | `FetchPlan.BASE` | `_local` + `_instance_name` + embedded |

**Best Practice:** Use `_base` by default until performance issues arise.

## ⚠️ CUBA Migration Warning

**CRITICAL:** CUBA's `_minimal` ≠ Jmix's `_instance_name`!

```java
// CUBA: _minimal loaded ALL local attributes
// Jmix: _instance_name loads ONLY @InstanceName field!

// Migration mapping:
// CUBA _minimal  →  Jmix _base (to get all local attrs)
// CUBA _local    →  Jmix _local
// CUBA view      →  Jmix fetchPlan
```

## XML Definition
```xml
<!-- In fetch-plans.xml -->
<fetch-plan entity="Order" name="order-with-lines" extends="_base">
    <property name="customer" fetchPlan="_instance_name" fetchMode="JOIN"/>
    <property name="lines" fetchPlan="_base" fetchMode="BATCH"/>
</fetch-plan>

<!-- Inline in view XML -->
<instance id="orderDc" class="Order">
    <fetchPlan extends="_base">
        <property name="customer" fetchPlan="_instance_name"/>
        <property name="lines" fetchPlan="_base"/>
    </fetchPlan>
    <loader/>
</instance>
```

## Programmatic (FetchPlans bean)
```java
@Autowired
private FetchPlans fetchPlans;

FetchPlan plan = fetchPlans.builder(Order.class)
    .addFetchPlan(FetchPlan.BASE)
    .add("customer", builder -> builder.addFetchPlan(FetchPlan.INSTANCE_NAME))
    .add("lines", FetchPlan.BASE)
    .build();

List<Order> orders = dataManager.load(Order.class)
    .query("select o from Order o")
    .fetchPlan(plan)
    .list();
```

## JmixDataRepository
```java
// FetchPlan as LAST parameter
List<Order> findByStatus(OrderStatus status, FetchPlan fetchPlan);

// Or by name
List<Order> findByCustomer(Customer customer, String fetchPlanName);
```

## Fetch Modes

| Mode | Description | Use For |
|------|-------------|---------|
| `AUTO` | Framework decides | Default |
| `JOIN` | Single query with SQL JOIN | to-one references |
| `BATCH` | Separate query with IN clause | collections |

## N+1 Problem Prevention
```java
// ❌ BAD: N+1 queries
List<Order> orders = dataManager.load(Order.class).all().list();
for (Order o : orders) {
    o.getCustomer().getName();  // Each access = extra query!
}

// ✅ GOOD: Single query with JOIN
FetchPlan plan = fetchPlans.builder(Order.class)
    .addFetchPlan(FetchPlan.BASE)
    .add("customer", FetchPlan.INSTANCE_NAME)
    .build();
List<Order> orders = dataManager.load(Order.class).fetchPlan(plan).list();
```

## List View Considerations
- Don't load collections in list views unless displaying them
- Use `_instance_name` for references in columns
- Think carefully about nested collections (n×n×n problem)

## Forbidden
- `FetchType.EAGER` on entity relationships
- Loading without fetch plan in loops
- Deep nesting (>3 levels)
- Using fetch plans as security mechanism
