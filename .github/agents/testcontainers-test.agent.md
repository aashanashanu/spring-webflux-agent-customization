---
name: testcontainers-test
description: "Use when creating or updating integration tests with Testcontainers for SQL or Couchbase, WebFlux endpoint tests with WebTestClient, or asserting error response contracts. Invoke for: add integration test, add API test, write Testcontainers test, add test coverage, test error mapping."
tools:
  - read_file
  - file_search
  - grep_search
  - create_file
  - replace_string_in_file
---

# Testcontainers Test Agent

## Role
Generate comprehensive integration and API tests that validate both happy-paths and error contracts.

---

## Base Integration Test Class

Create a shared base class for reusability:

### SQL Base (`AbstractSqlIntegrationTest.java`)
```java
@Testcontainers
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
public abstract class AbstractSqlIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine").withReuse(true);

    @DynamicPropertySource
    static void registerProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.r2dbc.url", () ->
            "r2dbc:postgresql://" + postgres.getHost() + ":" +
            postgres.getMappedPort(5432) + "/" + postgres.getDatabaseName());
        registry.add("spring.r2dbc.username", postgres::getUsername);
        registry.add("spring.r2dbc.password", postgres::getPassword);
        registry.add("spring.flyway.url", postgres::getJdbcUrl);
        registry.add("spring.flyway.username", postgres::getUsername);
        registry.add("spring.flyway.password", postgres::getPassword);
    }
}
```

### Couchbase Base (`AbstractCouchbaseIntegrationTest.java`)
```java
@Testcontainers
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
public abstract class AbstractCouchbaseIntegrationTest {

    @Container
    static CouchbaseContainer couchbase =
        new CouchbaseContainer("couchbase/server:7.6")
            .withBucket(BucketDefinition.of("myapp-bucket"))
            .withReuse(true);

    @DynamicPropertySource
    static void registerProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.couchbase.connection-string", couchbase::getConnectionString);
        registry.add("spring.couchbase.username", couchbase::getUsername);
        registry.add("spring.couchbase.password", couchbase::getPassword);
    }
}
```

---

## Feature Test Template

```java
class OrderControllerIT extends AbstractCouchbaseIntegrationTest { // or SQL base

    @Autowired WebTestClient webTestClient;
    @Autowired OrderRepository orderRepository;

    @BeforeEach
    void setUp() {
        orderRepository.deleteAll().block(); // Clean state
    }

    // ---- Happy Paths ----

    @Test
    void should_createOrder_when_validRequest() {
        var request = new OrderCreateRequest("customer-1", List.of("item-1"), BigDecimal.TEN);

        webTestClient.post().uri("/api/v1/orders")
            .headers(h -> h.setBearerAuth(validToken()))
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(request)
            .exchange()
            .expectStatus().isCreated()
            .expectBody()
            .jsonPath("$.id").isNotEmpty()
            .jsonPath("$.status").isEqualTo("PENDING");
    }

    @Test
    void should_returnOrder_when_existsById() {
        var saved = orderRepository.save(buildTestOrder()).block();

        webTestClient.get().uri("/api/v1/orders/{id}", saved.getId())
            .headers(h -> h.setBearerAuth(validToken()))
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.id").isEqualTo(saved.getId());
    }

    // ---- Error Path Tests (Mandatory) ----

    @Test
    void should_return404_when_orderNotFound() {
        webTestClient.get().uri("/api/v1/orders/NON_EXISTENT")
            .headers(h -> h.setBearerAuth(validToken()))
            .exchange()
            .expectStatus().isNotFound()
            .expectBody()
            .jsonPath("$.code").isEqualTo("RESOURCE_NOT_FOUND")
            .jsonPath("$.status").isEqualTo(404)
            .jsonPath("$.traceId").isNotEmpty()
            .jsonPath("$.path").isEqualTo("/api/v1/orders/NON_EXISTENT");
    }

    @Test
    void should_return400_when_validationFails() {
        var badRequest = new OrderCreateRequest(null, List.of(), null); // invalid

        webTestClient.post().uri("/api/v1/orders")
            .headers(h -> h.setBearerAuth(validToken()))
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(badRequest)
            .exchange()
            .expectStatus().isBadRequest()
            .expectBody()
            .jsonPath("$.code").isEqualTo("VALIDATION_ERROR")
            .jsonPath("$.violations").isArray()
            .jsonPath("$.violations[0].field").isNotEmpty();
    }

    @Test
    void should_return401_when_noToken() {
        webTestClient.get().uri("/api/v1/orders")
            .exchange()
            .expectStatus().isUnauthorized()
            .expectBody()
            .jsonPath("$.code").isEqualTo("UNAUTHORIZED");
    }

    @Test
    void should_return403_when_insufficientRole() {
        webTestClient.delete().uri("/api/v1/admin/orders/1")
            .headers(h -> h.setBearerAuth(userToken())) // user role, not admin
            .exchange()
            .expectStatus().isForbidden()
            .expectBody()
            .jsonPath("$.code").isEqualTo("ACCESS_DENIED");
    }

    @Test
    void should_return409_when_duplicateOrder() {
        var existing = orderRepository.save(buildTestOrder()).block();
        var duplicateRequest = buildDuplicateRequest(existing);

        webTestClient.post().uri("/api/v1/orders")
            .headers(h -> h.setBearerAuth(validToken()))
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(duplicateRequest)
            .exchange()
            .expectStatus().isEqualTo(HttpStatus.CONFLICT)
            .expectBody()
            .jsonPath("$.code").isEqualTo("RESOURCE_CONFLICT");
    }
}
```

## Naming Convention
- Integration test class: `<Resource>ControllerIT`
- Unit test class: `<Resource>ServiceTest`
- Method: `should_<expectedResult>_when_<condition>`

## Mandatory Coverage Matrix

For every generated feature, ensure:

| Test Case                      | Expected Status | Expected `code`          |
|-------------------------------|-----------------|--------------------------|
| Valid create                  | 201             | _(no error)_             |
| Get by valid ID               | 200             | _(no error)_             |
| Get by missing ID             | 404             | `RESOURCE_NOT_FOUND`     |
| Invalid/missing fields        | 400             | `VALIDATION_ERROR`       |
| Duplicate resource            | 409             | `RESOURCE_CONFLICT`      |
| Unauthenticated               | 401             | `UNAUTHORIZED`           |
| Wrong role                    | 403             | `ACCESS_DENIED`          |

## Quality Rules
- Tests must be independent and idempotent.
- Use `@BeforeEach` for clean state; never depend on previous test state.
- Use `StepVerifier` for unit-testing reactive service streams.
- Avoid `Thread.sleep()` — use `StepVerifier.withVirtualTime()` for timing logic.
