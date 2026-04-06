---
name: integration-testing
description: Use when unit tests pass and you need to verify components work together, or when setting up integration/E2E tests for a feature
---

# Integration & E2E Testing

Verify components work together after unit tests pass.

## Testing Pyramid

```
        /\
       /  \      E2E (few) - Full user flows, slowest
      /----\     
     /      \    Integration (some) - Component interactions
    /--------\
   /          \  Unit (many) - Logic, fastest
  /------------\
```

| Level | Scope | Speed | Run Frequency |
|-------|-------|-------|---------------|
| Unit | Single function/class | ms | Every save |
| Integration | Component interactions | seconds | Before merge |
| E2E | Full user flow | minutes | Before deploy |

## Environment Strategy

### Testcontainers (Recommended)

```java
@SpringBootTest
@Testcontainers
class OrderServiceIntegrationTest {

    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @ServiceConnection
    static RedisContainer redis = new RedisContainer(DockerImageName.parse("redis:7"));
}
```

Spring Boot 3.1+ auto-configures connection from `@ServiceConnection`.

### Docker Compose (Alternative)

```bash
docker-compose -f docker-compose.test.yml up -d
mvn verify -Pintegration
docker-compose -f docker-compose.test.yml down -v
```

## Data Isolation

**Each test must be independent.**

```java
@BeforeEach
void setup() {
    database.clean();
    // or: transaction.rollback() after each test
}

@AfterEach
void cleanup() {
    database.truncateAllTables();
}
```

**Never rely on test execution order.**

## Mock Strategy

| When to Mock | When to Use Real |
|--------------|------------------|
| External APIs (rate limits, cost) | Database |
| Email/SMS services | Message queues |
| Third-party auth | Cache (Redis) |
| Flaky external services | Internal services |

**Rule:** Mock slow/flaky/costly external deps. Use real internal deps.

## Test Structure

### Integration Test

```java
@SpringBootTest
class OrderServiceIntegrationTest {

    @Autowired OrderService orderService;
    @Autowired InventoryRepository inventoryRepository;

    @Test
    void shouldDeductInventory_WhenOrderCreated() {
        // Given - setup inventory in real database
        inventoryRepository.save(new Inventory("product-1", 10));
        CreateOrderRequest request = new CreateOrderRequest("product-1", 2);

        // When - call real service with real database
        Order order = orderService.createOrder(request);

        // Then - verify both order and inventory in real database
        assertThat(order.getStatus()).isEqualTo(OrderStatus.CONFIRMED);
        assertThat(inventoryRepository.findByProductId("product-1").getQuantity())
            .isEqualTo(8); // 10 - 2 = 8
    }
}
```

**What this tests:** OrderService correctly coordinates order creation AND inventory deduction.

### E2E Test

```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
class OrderE2ETest {

    @Autowired WebTestClient webTestClient;

    @Test
    void shouldCreateOrder_WhenUserSubmitsRequest() {
        webTestClient.post()
            .uri("/api/orders")
            .bodyValue(new CreateOrderRequest("product-1", 2))
            .exchange()
            .expectStatus().isCreated()
            .expectBody()
            .jsonPath("$.status").isEqualTo("CONFIRMED");
    }
}
```

## Running Strategy

| Trigger | What to Run |
|---------|-------------|
| Every save | Unit tests only |
| Before commit | Unit + changed integration |
| Before merge | All tests |
| Before deploy | All + E2E |

```bash
# Fast feedback - unit only
mvn test

# Integration tests
mvn verify -Pintegration

# E2E tests
mvn verify -Pe2e
```

## Red Flags

- **Tests share state** → Each test must be isolated
- **Order-dependent tests** → Fix immediately
- **No cleanup** → Will cause flaky tests
- **Mocking everything** → That's not integration testing
- **Skipping E2E for critical flows** → High risk

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Tests pass locally, fail in CI | Use Testcontainers for consistency |
| Slow tests | Reduce scope, move to unit tests |
| Flaky tests | Fix data isolation, avoid race conditions |
| Mocking database | Use real DB with transactions |
| Too many E2E tests | Push down to integration/unit |

## Checklist

Before marking integration tests complete:

- [ ] Each test is isolated (no shared state)
- [ ] Data cleaned up after each test
- [ ] External services mocked (or controlled)
- [ ] Real database/message queue used
- [ ] Tests pass in CI environment
- [ ] No order dependencies
