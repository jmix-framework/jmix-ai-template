---
name: testing
description: Writing reliable tests for Jmix — Unit, Layered (Integration), and E2E tests with proper authentication and cleanup.
---

# Testing

## When to Use
- When writing tests for Jmix services
- When testing with authentication context
- When implementing E2E tests

## Test Pyramid: 3 Levels

### 1. Unit Tests (No Spring Context)
```java
class PriceCalculatorTest {
    private final PriceCalculator calculator = new PriceCalculator();

    @ParameterizedTest
    @CsvSource({"100, 10, 90", "100, 0, 100"})
    void shouldApplyDiscount(int price, int discount, int expected) {
        assertThat(calculator.applyDiscount(price, discount)).isEqualTo(expected);
    }
}
```

### 2. Layered Tests (Integration with Mocks)
```java
@SpringBootTest
class OrderServiceLayeredTest {

    @Autowired OrderService orderService;
    @MockitoBean PaymentGateway paymentGateway;  // NOT @MockBean!

    @Test
    void shouldProcessPayment() {
        when(paymentGateway.charge(any())).thenReturn(true);
        orderService.checkout(order);
        verify(paymentGateway).charge(any());
    }
}
```

### 3. E2E Tests (Full Stack)
```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
@Testcontainers
class OrderApiE2ETest {

    @Autowired WebTestClient webClient;  // OK for test clients
    // Do NOT @Autowired internal services!

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @Test
    void shouldCreateOrderViaRest() {
        webClient.post().uri("/api/orders")
            .bodyValue(orderRequest)
            .exchange()
            .expectStatus().isCreated();
    }
}
```

## Authentication in Tests
Use `SystemAuthenticator` via JUnit Extension:

```java
public class AuthenticatedAsAdmin implements BeforeEachCallback, AfterEachCallback {
    
    @Override
    public void beforeEach(ExtensionContext context) {
        var systemAuthenticator = SpringExtension.getApplicationContext(context)
            .getBean(SystemAuthenticator.class);
        systemAuthenticator.begin("admin");
    }

    @Override
    public void afterEach(ExtensionContext context) {
        var systemAuthenticator = SpringExtension.getApplicationContext(context)
            .getBean(SystemAuthenticator.class);
        systemAuthenticator.end();
    }
}

// Usage:
@SpringBootTest
@ExtendWith(AuthenticatedAsAdmin.class)
class MyServiceTest { ... }
```

## Cleanup — Always in @AfterEach
```java
@AfterEach
void tearDown() {
    dataManager.remove(createdEntities);
}
```

## Checklist
- [ ] Choose level: Unit → Layered → E2E
- [ ] NO `@Transactional` on tests
- [ ] Cleanup in `@AfterEach`
- [ ] Use `@MockitoBean` (not `@MockBean`)
- [ ] Use `SystemAuthenticator` for auth
- [ ] AssertJ assertions

## Forbidden
- `@Transactional` on test classes/methods
- `@MockBean` (deprecated, use `@MockitoBean`)
- `@WithUserDetails` (use `SystemAuthenticator`)
- `@Autowired` internal services in E2E
- Cleanup at end of test method
