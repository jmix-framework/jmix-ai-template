---
name: services
description: Creating business logic services — DataManager usage, transactions, bean naming conventions, and Jmix Data Repositories.
---

# Services

## When to Use
- When implementing business logic
- When working with DataManager or transactions
- When creating service beans

## Service Template

### Recommended: Constructor Injection
```java
@Service("app_OrderService")  // Always use prefix!
public class OrderService {

    private final DataManager dataManager;

    public OrderService(DataManager dataManager) {
        this.dataManager = dataManager;
    }

    public Order createOrder() {
        return dataManager.create(Order.class);
    }
}
```

### Allowed: Field Injection (CUBA Style)
```java
@Service
public class LegacyStyleService {
    @Autowired
    private DataManager dataManager;
}
```

## DataManager Operations (Primary API)

```java
// Load by ID
Customer customer = dataManager.load(Customer.class)
    .id(customerId)
    .fetchPlan("customer-full")
    .one();

// Load List with Query
List<Order> orders = dataManager.load(Order.class)
    .query("select o from Order o where o.customer = :customer")
    .parameter("customer", customer)
    .list();

// Save
dataManager.save(order, customer);

// Save with SaveContext
SaveContext saveContext = new SaveContext();
saveContext.saving(order);
saveContext.removing(deletedLines);
dataManager.save(saveContext);

// Remove
dataManager.remove(entity);
```

## Advanced: EntityManager (Bulk/Native)

```java
@Service
public class BulkOperationService {

    @PersistenceContext
    private EntityManager entityManager;

    @Transactional  // REQUIRED!
    public void archiveOldOrders() {
        entityManager.createQuery(
            "update Order o set o.status = :status where o.date < :cutoff")
            .setParameter("status", OrderStatus.ARCHIVED)
            .setParameter("cutoff", LocalDate.now().minusYears(1))
            .executeUpdate();
    }
}
```

## Transactions

```java
@Transactional
public void complexOperation() {
    Order order = dataManager.save(newOrder);
    processPayment(order);
}

@Transactional(readOnly = true)
public List<Order> getReport() {
    return dataManager.load(Order.class).all().list();
}
```

## TransactionTemplate (Programmatic)

```java
@Service
public class ManualTxService {

    private final TransactionTemplate transactionTemplate;

    public ManualTxService(PlatformTransactionManager txManager) {
        this.transactionTemplate = new TransactionTemplate(txManager);
    }

    public void executeInTransaction() {
        transactionTemplate.execute(status -> {
            // Code in transaction
            if (somethingFails()) {
                status.setRollbackOnly();
            }
            return null;
        });
    }
}
```

## Bean Naming Conventions

| Stereotype | Usage | Example |
|------------|-------|---------|
| `@Service` | Business logic | `@Service("app_OrderService")` |
| `@Component` | Infrastructure, Utils | `@Component("app_OrderListener")` |
| `@Bean` | Config methods | `@Bean("app_CustomMapper")` |

## Jmix Data Repositories

```java
public interface UserRepository extends JmixDataRepository<User, UUID> {
    
    List<User> findByActiveTrue();
    
    // FetchPlan as LAST parameter
    List<User> findByLastName(String lastName, FetchPlan fetchPlan);
}
```

## Checklist
- [ ] `@Service` for business logic
- [ ] Bean name prefix applied (`app_`)
- [ ] `DataManager` for CRUD
- [ ] `EntityManager` only for bulk/native with `@Transactional`
- [ ] Business logic in services, not views

## Forbidden
- `EntityManager` without `@Transactional`
- UI components in services
- Raw `tm.begin()` — use `@Transactional` or `TransactionTemplate`
