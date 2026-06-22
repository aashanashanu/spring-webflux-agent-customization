---
description: "Global engineering policy for Spring Boot WebFlux services with SQL/Couchbase/MongoDB, Gradle build management, Spring Actuator, Swagger/OpenAPI, Spring Security hardening, request-response logging, log rotation, global exception handling, and Testcontainers."
applyTo: "**/*"
---

# Spring Boot WebFlux — Global Copilot Policy

## 1. Architecture & Coding Standards

- Follow layered architecture: `Controller → Service → Repository`.
- Use **Gradle** as the project build tool (`build.gradle` or `build.gradle.kts`); do not generate Maven `pom.xml`.
- Use reactive types (`Mono<T>`, `Flux<T>`) throughout the call chain.
- **Never** call blocking APIs (`.block()`, `Thread.sleep()`, JDBC) in a reactive request path.
- Prefer constructor injection and immutable DTOs (records or `@Value` objects).
- All request/response models must carry validation annotations (`@NotNull`, `@Valid`, etc.).
- Keep controllers thin — orchestration only; push business logic to the service layer.
- Use explicit, descriptive names and small, single-purpose methods.
- Add comments only for non-obvious logic; avoid redundant comments.
- Version API routes (`/api/v1/...`).

## 1.1 Platform Baseline (Required)

- Include Spring Boot Actuator dependency and enable endpoint exposure config.
- Include Swagger/OpenAPI for WebFlux using `springdoc-openapi-starter-webflux-ui`.
- Expose OpenAPI docs at `/v3/api-docs` and Swagger UI at `/swagger-ui.html`.
- Keep actuator endpoint exposure minimal by default (`health`, `info`, `metrics`).

## 2. Database Decision Rule

Before generating **any** repository, entity, or persistence configuration code:
1. Ask the developer: **SQL or NoSQL?**
2. For **SQL**: generate reactive SQL stack (R2DBC), prompt for engine choice:
   - PostgreSQL
   - MySQL
   - MariaDB
3. For **NoSQL**: prompt for engine choice:
  - Couchbase (Spring Data Couchbase reactive)
  - MongoDB (Spring Data Reactive MongoDB)
  - Do NOT suggest or generate Cassandra, Redis, or any other NoSQL alternative.
4. Apply the chosen engine in application config, repository, Testcontainers, and CI config.

## 3. REST Controller Conventions

- Annotate with `@RestController` and `@RequestMapping("/api/v1/<resource>")`.
- Use `@Validated` at class level and `@Valid` on request body parameters.
- Return `Mono<ResponseEntity<T>>` for single resource responses.
- Return `Flux<T>` or `Mono<ResponseEntity<List<T>>>` for collections.
- Use `HttpStatus` constants; never hardcode integer status codes.
- Propagate domain exceptions; let the global handler map them to HTTP responses.

## 4. Request/Response Logging

- Implement a `WebFilter` for request/response logging.
- Log for every request: method, path, query params, correlation/trace ID, response status, latency.
- **Mask** the following fields in all logged payloads: `password`, `authorization`, `token`,
  `secret`, `creditCard`, `ssn`, and any field annotated `@Sensitive`.
- Do not log raw request/response bodies by default; enable via feature flag only.
- Attach `traceId` and `spanId` from MDC or Reactor context to every log record.

## 5. Logback XML — Rolling Policy

- Use `logback-spring.xml` (not `logback.xml`) to leverage Spring profiles.
- Configure a `RollingFileAppender` with:
  - `TimeBasedRollingPolicy`: daily rotation, pattern `app-%d{yyyy-MM-dd}.%i.log.gz`.
  - `SizeAndTimeBasedFNATP`: max file size `50MB`.
  - `maxHistory`: `30` days.
  - `totalSizeCap`: `2GB`.
- Console appender active only for `local` and `test` profiles.
- File appender active for `dev`, `staging`, `prod`.
- Log level defaults: `ROOT=WARN`, `com.yourcompany=INFO`; override per profile.

## 6. Spring Security Hardening

- Use `SecurityWebFilterChain` (reactive security); do NOT use servlet `HttpSecurity`.
- Deny-by-default: all routes require authentication unless explicitly permitted.
- Permitted public routes must be listed explicitly and documented.
- Apply the following headers: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`,
  `Cache-Control: no-store`, `Strict-Transport-Security` (prod only).
- Disable CSRF for stateless REST APIs; document the decision.
- Do not log raw credentials or auth tokens.
- Return sanitized, non-leaking error messages for auth/authz failures.

## 7. Global Exception Handling

- Implement **one** centralized `@Component` that extends `DefaultErrorWebExceptionHandler`
  or implements `ErrorWebExceptionHandler` for WebFlux.
- Every handled exception returns the following standardized error payload:

```json
{
  "timestamp": "2026-06-21T10:15:30Z",
  "path": "/api/v1/resource/id",
  "status": 404,
  "error": "Not Found",
  "code": "RESOURCE_NOT_FOUND",
  "message": "Human-friendly, safe message",
  "traceId": "abc123"
}
```

- HTTP status mapping (must be implemented):

| Exception                              | HTTP Status | code                      |
|----------------------------------------|-------------|---------------------------|
| `MethodArgumentNotValidException`      | 400         | `VALIDATION_ERROR`        |
| `ConstraintViolationException`         | 400         | `VALIDATION_ERROR`        |
| `WebExchangeBindException`             | 400         | `REQUEST_BINDING_ERROR`   |
| `IllegalArgumentException`             | 400         | `BAD_REQUEST`             |
| `ServerWebInputException`              | 400         | `MALFORMED_REQUEST`       |
| `AuthenticationException`             | 401         | `UNAUTHORIZED`            |
| `AccessDeniedException`               | 403         | `ACCESS_DENIED`           |
| `ResourceNotFoundException`           | 404         | `RESOURCE_NOT_FOUND`      |
| `DuplicateResourceException`          | 409         | `RESOURCE_CONFLICT`       |
| `UnsupportedMediaTypeStatusException` | 415         | `UNSUPPORTED_MEDIA_TYPE`  |
| `ResponseStatusException`             | embedded    | `RESPONSE_STATUS_EXCEPTION` |
| `TimeoutException`                    | 504         | `UPSTREAM_TIMEOUT`        |
| `Throwable` (catch-all)               | 500         | `INTERNAL_SERVER_ERROR`   |

- 4xx → log as `WARN`; 5xx → log as `ERROR`; always include `traceId` and `path`.
- **Never** leak stack traces, class names, or internal messages to the client.

## 8. Testcontainers

- All integration tests use Testcontainers; no mocked persistence in integration tests.
- Annotate integration test classes with `@Testcontainers` and `@SpringBootTest(webEnvironment = RANDOM_PORT)`.
- For **SQL**: use the matching SQL container and R2DBC test datasource.
- For **NoSQL (Couchbase)**: use `CouchbaseContainer` from `org.testcontainers:couchbase`.
- For **NoSQL (MongoDB)**: use `MongoDBContainer` from `org.testcontainers:mongodb`.
- Test **both** happy-path and error-path (validate HTTP status codes and error `code` fields).
- Reuse containers across tests using `@Container static` and `Reusable=true` where practical.

## 9. General Quality Rules

- No magic constants — use named constants or enums.
- Avoid duplicated logic; extract shared utilities.
- Reactive pipelines must handle error operators (`.onErrorMap`, `.onErrorReturn`, etc.).
- All public service methods return reactive types.
- Never expose internal entity classes directly from controllers; use DTOs.
