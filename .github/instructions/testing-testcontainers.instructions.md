---
description: "Use when creating or modifying integration tests, unit tests, Testcontainers setup for SQL or Couchbase, or WebFlux test client tests."
applyTo: "**/src/test/**/*.java,**/*Test.java,**/*IT.java,**/*IntegrationTest.java"
---

# Testing & Testcontainers Instructions

## Test Types
- **Unit tests**: test service logic in isolation; mock repository and external dependencies.
- **Integration tests**: use Testcontainers with a real database container; no mocked persistence.
- **API tests**: use `WebTestClient` to validate controller behavior end-to-end.

## Integration Test Setup
```java
@Testcontainers
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class OrderServiceIT {
    // ...
}
```

## SQL Testcontainers (when SQL branch chosen)
```java
@Container
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
    .withReusable(true);

@DynamicPropertySource
static void properties(DynamicPropertyRegistry registry) {
    registry.add("spring.r2dbc.url", () -> "r2dbc:postgresql://" +
        postgres.getHost() + ":" + postgres.getMappedPort(5432) + "/" + postgres.getDatabaseName());
    registry.add("spring.r2dbc.username", postgres::getUsername);
    registry.add("spring.r2dbc.password", postgres::getPassword);
}
```

## Couchbase Testcontainers (when NoSQL branch chosen)
```java
@Container
static CouchbaseContainer couchbase = new CouchbaseContainer("couchbase/server:7.6")
    .withBucket(BucketDefinition.of("myapp-bucket"))
    .withReusable(true);

@DynamicPropertySource
static void properties(DynamicPropertyRegistry registry) {
    registry.add("spring.couchbase.connection-string", couchbase::getConnectionString);
    registry.add("spring.couchbase.username", couchbase::getUsername);
    registry.add("spring.couchbase.password", couchbase::getPassword);
}
```

## Mandatory Test Coverage
For each new feature, provide:

1. **Happy-path test**: valid input → expected 2xx response + body assertions.
2. **Validation error test**: invalid input → assert `400` status and `code: VALIDATION_ERROR` in body.
3. **Not found test**: missing resource → assert `404` status and `code: RESOURCE_NOT_FOUND`.
4. **Conflict test** (where applicable): duplicate → assert `409` status and `code: RESOURCE_CONFLICT`.
5. **Auth failure test**: unauthenticated request → assert `401`; unauthorized role → assert `403`.

## WebTestClient Pattern
```java
webTestClient.post()
    .uri("/api/v1/orders")
    .contentType(MediaType.APPLICATION_JSON)
    .bodyValue(requestDto)
    .exchange()
    .expectStatus().isCreated()
    .expectBody()
    .jsonPath("$.id").isNotEmpty()
    .jsonPath("$.status").isEqualTo("PENDING");
```

## Error Response Assertions
```java
webTestClient.get()
    .uri("/api/v1/orders/INVALID_ID")
    .exchange()
    .expectStatus().isNotFound()
    .expectBody()
    .jsonPath("$.code").isEqualTo("RESOURCE_NOT_FOUND")
    .jsonPath("$.traceId").isNotEmpty()
    .jsonPath("$.path").isEqualTo("/api/v1/orders/INVALID_ID");
```

## Naming Convention
- Unit test class: `<ClassName>Test`
- Integration test class: `<ClassName>IT`
- Test method: `should_<expectedBehavior>_when_<condition>` (snake_case with underscores).

## Quality Rules
- Tests must not depend on each other (independent, idempotent).
- Use `@BeforeEach` for setup and `@AfterEach` for cleanup.
- Avoid `Thread.sleep()` in tests; use Awaitility for async assertions.
