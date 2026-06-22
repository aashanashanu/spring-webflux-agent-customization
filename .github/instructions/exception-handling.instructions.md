---
description: "Use when creating or modifying global exception handlers, custom exception classes, error response models, or HTTP error status mappings."
applyTo: "**/*Exception*.java,**/*ExceptionHandler*.java,**/*ErrorHandler*.java,**/*Error*.java,**/*Advice*.java,**/exception/**/*.java,**/error/**/*.java"
---

# Global Exception Handling Instructions

## Standard Error Response Model

Every API error response MUST use this exact shape:

```java
public record ApiErrorResponse(
    Instant timestamp,
    String path,
    int status,
    String error,      // HTTP reason phrase, e.g. "Not Found"
    String code,       // Application error code, e.g. "RESOURCE_NOT_FOUND"
    String message,    // Human-friendly, safe message — no internal details
    String traceId     // From MDC/Reactor context
) {}
```

## Global Exception Handler

- Implement as a `@Component` class extending `AbstractErrorWebExceptionHandler`
  OR implementing `ErrorWebExceptionHandler` directly for WebFlux.
- Register with `@Order(-2)` to override Spring Boot's default error handler.
- Keep handler non-blocking — return `Mono<Void>` using reactive writes.

```java
@Component
@Order(-2)
public class GlobalExceptionHandler extends AbstractErrorWebExceptionHandler {
    // implement getRoutingFunction() to route to handler methods
}
```

## Required Domain Exception Classes

Create the following in `<base-package>.exception`:

| Class                          | Extends               | Usage                                  |
|--------------------------------|-----------------------|----------------------------------------|
| `ResourceNotFoundException`    | `RuntimeException`    | Entity/document not found by id/query  |
| `DuplicateResourceException`   | `RuntimeException`    | Unique constraint violated             |
| `BusinessRuleException`        | `RuntimeException`    | Domain rule violation                  |
| `ExternalServiceException`     | `RuntimeException`    | Downstream/integration call failure    |

Each must carry: `message`, `code` (application code string), and optional `details` map.

## HTTP Status Mapping Table

| Exception Class                          | HTTP Status                   | `code` Value                |
|------------------------------------------|-------------------------------|-----------------------------|
| `MethodArgumentNotValidException`        | 400 BAD_REQUEST               | `VALIDATION_ERROR`          |
| `ConstraintViolationException`           | 400 BAD_REQUEST               | `VALIDATION_ERROR`          |
| `WebExchangeBindException`               | 400 BAD_REQUEST               | `REQUEST_BINDING_ERROR`     |
| `IllegalArgumentException`               | 400 BAD_REQUEST               | `BAD_REQUEST`               |
| `ServerWebInputException`                | 400 BAD_REQUEST               | `MALFORMED_REQUEST`         |
| `AuthenticationException`               | 401 UNAUTHORIZED              | `UNAUTHORIZED`              |
| `AccessDeniedException`                 | 403 FORBIDDEN                 | `ACCESS_DENIED`             |
| `ResourceNotFoundException`             | 404 NOT_FOUND                 | `RESOURCE_NOT_FOUND`        |
| `DuplicateResourceException`            | 409 CONFLICT                  | `RESOURCE_CONFLICT`         |
| `BusinessRuleException`                 | 422 UNPROCESSABLE_ENTITY      | `BUSINESS_RULE_VIOLATION`   |
| `UnsupportedMediaTypeStatusException`   | 415 UNSUPPORTED_MEDIA_TYPE    | `UNSUPPORTED_MEDIA_TYPE`    |
| `ResponseStatusException`               | use `getStatusCode()`         | `RESPONSE_STATUS_EXCEPTION` |
| `TimeoutException`                      | 504 GATEWAY_TIMEOUT           | `UPSTREAM_TIMEOUT`          |
| `Throwable` (fallback)                  | 500 INTERNAL_SERVER_ERROR     | `INTERNAL_SERVER_ERROR`     |

## Logging Rules in Handler

- 4xx errors → log at `WARN` level with: `traceId`, `path`, `status`, `code`, `message`.
- 5xx errors → log at `ERROR` level with: `traceId`, `path`, `status`, `code`, full exception stack.
- **Never** include stack traces, class names, or internal messages in the client response body.
- For `INTERNAL_SERVER_ERROR` responses, use a fixed safe message: `"An unexpected error occurred. Please try again or contact support."`.

## Security-Sensitive Error Behavior

- `AuthenticationException` → do not reveal whether the account exists; use generic message.
- `AccessDeniedException` → do not reveal what permissions are required.
- Validation errors → DO include field-level detail (field name + constraint message) — safe to expose.

## Validation Error Detail (400 only)

For validation exceptions, include `violations` array in the response:

```json
{
  "timestamp": "2026-06-21T10:15:30Z",
  "path": "/api/v1/orders",
  "status": 400,
  "error": "Bad Request",
  "code": "VALIDATION_ERROR",
  "message": "Request validation failed",
  "traceId": "abc123",
  "violations": [
    { "field": "quantity", "message": "must be greater than 0" },
    { "field": "productId", "message": "must not be blank" }
  ]
}
```
