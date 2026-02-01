---
description: Rules for writing reliable tests for Jmix applications
globs: ["**/test/**"]
alwaysApply: false
---

# Jmix Testing Rules

## Scope
Applies to `src/test/java/**`

## Test Levels
- **Unit:** Pure logic, no Spring context
- **Layered:** `@SpringBootTest` + `@MockitoBean` for mocks if need to mock services
- **E2E:** `@SpringBootTest(webEnvironment = RANDOM_PORT)` + TestContainers

## Service Test Pattern
```java
@SpringBootTest
@ExtendWith(AuthenticatedAsAdmin.class)
class MyServiceTest {
    @Autowired DataManager dataManager;
    @MockitoBean PaymentGateway paymentGateway;
    
    @AfterEach
    void tearDown() {
        dataManager.remove(createdEntities);
    }
}
```

## E2E Test Pattern
```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
@Testcontainers
class MyApiE2ETest {
    @Autowired WebTestClient webClient;  // OK for test clients
    // Do NOT @Autowired internal services (breaks isolation)
}
``` 

Or Selenide / Selenium tests for smoke testing you application

## Authentication
- Use `SystemAuthenticator.begin("user")/end()` via JUnit Extension
- Do NOT use `@WithUserDetails`

## Cleanup
- Always in `@AfterEach`, never at end of test method

## Assertions
- Use AssertJ (`assertThat`)
- Follow AAA: Arrange → Act → Assert

## Forbidden
- `@Transactional` on tests (Jmix DataManager manages its own)
- `@MockBean` (deprecated, use `@MockitoBean`)
- `@WithUserDetails` (use `SystemAuthenticator`)
- `@Autowired` internal services in E2E (use WebTestClient/RestAssured)
- Cleanup at end of test method
- `System.out.println`
- Hardcoded UUIDs
- Over arranging after asserts, re-asserts etc
