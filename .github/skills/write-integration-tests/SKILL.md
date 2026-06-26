---
name: write-integration-tests
description: Create integration tests with WebTestClient, Testcontainers, and contract assertions.
---

# Skill: Write Integration Tests

## Templates
```java
@Testcontainers
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class OrderControllerIT {
    @Autowired WebTestClient client;

    @Test
    void should_createOrder_when_validRequest() {
        client.post().uri("/api/v1/orders")
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue("{\"customerId\":\"c1\",\"total\":10}")
            .exchange()
            .expectStatus().isCreated();
    }
}
```

## Best practices
- Use real infrastructure for integration paths.
- Validate both happy and error paths.
- Assert standardized error response contracts.

## Validation criteria
- Includes 2xx, 4xx, and security scenarios.
- Testcontainers setup matches selected data platform.
- WebTestClient assertions verify status and key payload fields.
