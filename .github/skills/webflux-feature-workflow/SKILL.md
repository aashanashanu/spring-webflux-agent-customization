---
name: webflux-feature-workflow
description: "Use when building a new Spring Boot WebFlux feature module end-to-end — covers database selection (SQL, Couchbase, or MongoDB), controller, service, repository, DTO, logging, Spring Security route protection, global exception handling, and Testcontainers integration tests."
---

# WebFlux Feature Workflow

This skill guides a complete feature implementation across all layers of a Spring Boot WebFlux service.

---

## Step 0 — Baseline Project Setup

- Use **Gradle** build management (`build.gradle` or `build.gradle.kts`).
- Ensure the project includes:
  - `org.springframework.boot:spring-boot-starter-actuator`
  - `org.springdoc:springdoc-openapi-starter-webflux-ui`
- Ensure `application.yml` has safe defaults:
  - `management.endpoints.web.exposure.include: health,info,metrics`
  - `management.endpoint.health.show-details: when_authorized`
- Ensure API documentation endpoints are available:
  - OpenAPI JSON: `/v3/api-docs`
  - Swagger UI: `/swagger-ui.html`

---

## Step 1 — Confirm Database Choice

Ask the developer before generating any persistence code:

> "Which database will this feature use?"
> 1. **SQL** — choose engine: PostgreSQL / MySQL / MariaDB (R2DBC)
> 2. **NoSQL** — choose engine: Couchbase (reactive Spring Data Couchbase) / MongoDB (reactive Spring Data MongoDB)

Store the answer and apply consistently throughout the remaining steps.

---

## Step 2 — Generate Feature Skeleton

Based on the resource name provided, generate:

### Controller (`<Resource>Controller.java`)
- `@RestController`, `@RequestMapping("/api/v1/<resources>")`
- CRUD methods returning `Mono<ResponseEntity<T>>` / `Flux<T>`
- `@Valid` on request body params; `@Validated` at class level

### Service (`<Resource>Service.java` + `<Resource>ServiceImpl.java`)
- Interface + implementation pattern
- Return `Mono<T>` / `Flux<T>`
- Throw `ResourceNotFoundException`, `DuplicateResourceException` as needed
- Map persistence exceptions to domain exceptions

### Repository
- **SQL path**: `interface <Resource>Repository extends ReactiveCrudRepository<T, ID>`
  with R2DBC; add custom `@Query` methods as needed
- **Couchbase path**: `interface <Resource>Repository extends ReactiveCouchbaseRepository<T, String>`
  with `@N1qlPrimaryIndexed`; add `@Query` for N1QL queries
- **MongoDB path**: `interface <Resource>Repository extends ReactiveMongoRepository<T, String>`
  with derived query methods and `@Query` for custom Mongo queries as needed

### Entity / Document
- **SQL**: `@Table` annotated record/class with `@Id`
- **Couchbase**: `@Document` annotated class with `@Id` (String key), `@Field` annotations,
  `@GeneratedValue(strategy = GenerationStrategy.USE_ATTRIBUTES)`
- **MongoDB**: `@Document(collection = "<resources>")` annotated class with `@Id` (String key)
  and Spring Data MongoDB field mappings as needed

### DTOs
- Separate `<Resource>CreateRequest`, `<Resource>UpdateRequest`, `<Resource>Response` records
- Field-level validation annotations on request DTOs

### Mapper (`<Resource>Mapper.java`)
- MapStruct `@Mapper(componentModel = "spring")` or manual mapper
- Convert entity ↔ DTO without exposing internals

---

## Step 3 — Observability

- Add / verify `RequestResponseLoggingFilter` exists:
  - Log: method, path, traceId, status, latency
  - Mask sensitive fields
- Add / verify `logback-spring.xml` rolling policy:
  - Daily + size-based rotation (`50MB` per file)
  - `maxHistory=30`, `totalSizeCap=2GB`
  - JSON structured output for non-local profiles

---

## Step 4 — Security

- Add route authorization rule for the new resource in `SecurityWebFilterChain`:
  - Public endpoints → `.pathMatchers(...).permitAll()` with comment
  - Protected endpoints → `.authenticated()` or `.hasRole(...)`
- Ensure secure HTTP headers are configured
- Document any role requirements in the controller Javadoc

---

## Step 5 — Exception Handling

- Ensure `GlobalExceptionHandler` exists and handles all mappings from the exception-handling instructions
- Create or verify domain exception classes:
  - `ResourceNotFoundException`
  - `DuplicateResourceException`
  - `BusinessRuleException`
- Ensure `ApiErrorResponse` record exists as the standard error payload
- Add error response serialization for validation violations (field + message array)

---

## Step 6 — Testcontainers Integration Tests

Generate `<Resource>IT.java`:

### SQL path
```java
@Container static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine").withReusable(true);
```

### Couchbase path
```java
@Container static CouchbaseContainer couchbase = new CouchbaseContainer("couchbase/server:7.6")
    .withBucket(BucketDefinition.of("myapp-bucket")).withReusable(true);
```

### MongoDB path
```java
@Container static MongoDBContainer mongo = new MongoDBContainer("mongo:7.0").withReuse(true);
```

Test methods to generate for each resource:
- `should_createResource_when_validRequest` → `201 Created`
- `should_returnResource_when_existsById` → `200 OK` + body assertion
- `should_return404_when_resourceNotFound` → assert `code: RESOURCE_NOT_FOUND`
- `should_return400_when_validationFails` → assert `code: VALIDATION_ERROR` + `violations`
- `should_return409_when_duplicateResource` → assert `code: RESOURCE_CONFLICT`
- `should_return401_when_unauthenticated` → assert `code: UNAUTHORIZED`
- `should_return403_when_insufficientRole` → assert `code: ACCESS_DENIED`

---

## Step 7 — Verify

Before completing:
- [ ] No blocking calls (`.block()`) in reactive path
- [ ] All controller methods return reactive types
- [ ] `traceId` present in all error responses
- [ ] Sensitive fields masked in logs
- [ ] Security route config updated
- [ ] All required test cases implemented
- [ ] `logback-spring.xml` rotation policy in place
