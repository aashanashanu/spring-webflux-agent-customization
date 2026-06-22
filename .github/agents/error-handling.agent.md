---
name: error-handling
description: "Use when implementing or updating global exception handling, custom exception classes, error response models, or HTTP error status mappings. Invoke for: add exception handler, create custom exception, add error response, map exception to status, add global error handling."
tools:
  - read_file
  - file_search
  - grep_search
  - create_file
  - replace_string_in_file
---

# Error Handling Agent

## Role
Create and maintain the centralized exception handling layer: error model, domain exceptions, and global handler.

---

## File Structure to Generate

```
src/main/java/<base-package>/
├── exception/
│   ├── ResourceNotFoundException.java
│   ├── DuplicateResourceException.java
│   ├── BusinessRuleException.java
│   └── ExternalServiceException.java
├── error/
│   ├── ApiErrorResponse.java
│   ├── ApiViolation.java
│   └── GlobalExceptionHandler.java
```

---

## ApiErrorResponse Record

```java
package com.example.error;

import java.time.Instant;
import java.util.List;

public record ApiErrorResponse(
    Instant timestamp,
    String path,
    int status,
    String error,
    String code,
    String message,
    String traceId,
    List<ApiViolation> violations  // non-null only for VALIDATION_ERROR
) {
    public static ApiErrorResponse of(int status, String error, String code,
                                      String message, String path, String traceId) {
        return new ApiErrorResponse(Instant.now(), path, status, error, code, message, traceId, null);
    }

    public static ApiErrorResponse withViolations(String path, String traceId,
                                                   List<ApiViolation> violations) {
        return new ApiErrorResponse(Instant.now(), path, 400, "Bad Request",
            "VALIDATION_ERROR", "Request validation failed", traceId, violations);
    }
}
```

## ApiViolation Record

```java
public record ApiViolation(String field, String message) {}
```

---

## Domain Exception Classes

```java
// ResourceNotFoundException.java
public class ResourceNotFoundException extends RuntimeException {
    private final String code = "RESOURCE_NOT_FOUND";
    public ResourceNotFoundException(String message) { super(message); }
    public String getCode() { return code; }
}

// DuplicateResourceException.java
public class DuplicateResourceException extends RuntimeException {
    private final String code = "RESOURCE_CONFLICT";
    public DuplicateResourceException(String message) { super(message); }
    public String getCode() { return code; }
}

// BusinessRuleException.java
public class BusinessRuleException extends RuntimeException {
    private final String code;
    public BusinessRuleException(String message, String code) {
        super(message);
        this.code = code;
    }
    public String getCode() { return code; }
}

// ExternalServiceException.java
public class ExternalServiceException extends RuntimeException {
    private final String code = "EXTERNAL_SERVICE_ERROR";
    public ExternalServiceException(String message, Throwable cause) { super(message, cause); }
    public String getCode() { return code; }
}
```

---

## GlobalExceptionHandler

```java
@Component
@Order(-2)
public class GlobalExceptionHandler extends AbstractErrorWebExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    private final ObjectMapper objectMapper;

    public GlobalExceptionHandler(ErrorAttributes errorAttributes,
                                   WebProperties webProperties,
                                   ApplicationContext applicationContext,
                                   ServerCodecConfigurer codecConfigurer,
                                   ObjectMapper objectMapper) {
        super(errorAttributes, webProperties.getResources(), applicationContext);
        setMessageWriters(codecConfigurer.getWriters());
        setMessageReaders(codecConfigurer.getReaders());
        this.objectMapper = objectMapper;
    }

    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
        return RouterFunctions.route(RequestPredicates.all(), this::renderErrorResponse);
    }

    private Mono<ServerResponse> renderErrorResponse(ServerRequest request) {
        Throwable error = getError(request);
        String path = request.path();
        String traceId = request.exchange().getAttribute("traceId");
        if (traceId == null) traceId = UUID.randomUUID().toString();

        return buildErrorResponse(error, path, traceId)
            .flatMap(response ->
                ServerResponse.status(response.status())
                    .contentType(MediaType.APPLICATION_JSON)
                    .bodyValue(response)
            );
    }

    private Mono<ApiErrorResponse> buildErrorResponse(Throwable ex, String path, String traceId) {
        return Mono.fromCallable(() -> {
            if (ex instanceof WebExchangeBindException e) {
                log.warn("Validation error traceId={} path={}", traceId, path);
                List<ApiViolation> violations = e.getBindingResult().getFieldErrors().stream()
                    .map(fe -> new ApiViolation(fe.getField(), fe.getDefaultMessage()))
                    .toList();
                return ApiErrorResponse.withViolations(path, traceId, violations);
            }
            if (ex instanceof ConstraintViolationException e) {
                log.warn("Constraint violation traceId={} path={}", traceId, path);
                List<ApiViolation> violations = e.getConstraintViolations().stream()
                    .map(cv -> new ApiViolation(
                        cv.getPropertyPath().toString(), cv.getMessage()))
                    .toList();
                return ApiErrorResponse.withViolations(path, traceId, violations);
            }
            if (ex instanceof IllegalArgumentException || ex instanceof ServerWebInputException) {
                log.warn("Bad request traceId={} path={} message={}", traceId, path, ex.getMessage());
                return ApiErrorResponse.of(400, "Bad Request",
                    ex instanceof ServerWebInputException ? "MALFORMED_REQUEST" : "BAD_REQUEST",
                    ex.getMessage(), path, traceId);
            }
            if (ex instanceof AuthenticationException) {
                log.warn("Unauthorized traceId={} path={}", traceId, path);
                return ApiErrorResponse.of(401, "Unauthorized", "UNAUTHORIZED",
                    "Authentication required", path, traceId);
            }
            if (ex instanceof AccessDeniedException) {
                log.warn("Forbidden traceId={} path={}", traceId, path);
                return ApiErrorResponse.of(403, "Forbidden", "ACCESS_DENIED",
                    "Access denied", path, traceId);
            }
            if (ex instanceof ResourceNotFoundException e) {
                log.warn("Not found traceId={} path={} message={}", traceId, path, ex.getMessage());
                return ApiErrorResponse.of(404, "Not Found", e.getCode(),
                    ex.getMessage(), path, traceId);
            }
            if (ex instanceof DuplicateResourceException e) {
                log.warn("Conflict traceId={} path={} message={}", traceId, path, ex.getMessage());
                return ApiErrorResponse.of(409, "Conflict", e.getCode(),
                    ex.getMessage(), path, traceId);
            }
            if (ex instanceof BusinessRuleException e) {
                log.warn("Business rule traceId={} path={} code={}", traceId, path, e.getCode());
                return ApiErrorResponse.of(422, "Unprocessable Entity", e.getCode(),
                    ex.getMessage(), path, traceId);
            }
            if (ex instanceof UnsupportedMediaTypeStatusException) {
                log.warn("Unsupported media type traceId={} path={}", traceId, path);
                return ApiErrorResponse.of(415, "Unsupported Media Type",
                    "UNSUPPORTED_MEDIA_TYPE", "Unsupported media type", path, traceId);
            }
            if (ex instanceof ResponseStatusException e) {
                int status = e.getStatusCode().value();
                log.warn("Response status exception traceId={} path={} status={}", traceId, path, status);
                return ApiErrorResponse.of(status,
                    HttpStatus.resolve(status) != null ? HttpStatus.resolve(status).getReasonPhrase() : "Error",
                    "RESPONSE_STATUS_EXCEPTION", e.getReason(), path, traceId);
            }
            if (ex instanceof TimeoutException) {
                log.error("Timeout traceId={} path={}", traceId, path);
                return ApiErrorResponse.of(504, "Gateway Timeout", "UPSTREAM_TIMEOUT",
                    "The request timed out. Please try again.", path, traceId);
            }
            // Fallback — 500
            log.error("Unhandled exception traceId={} path={}", traceId, path, ex);
            return ApiErrorResponse.of(500, "Internal Server Error", "INTERNAL_SERVER_ERROR",
                "An unexpected error occurred. Please try again or contact support.",
                path, traceId);
        });
    }
}
```

## Responsibilities Checklist
- [ ] `ApiErrorResponse` record created
- [ ] `ApiViolation` record created
- [ ] All 4 domain exception classes created
- [ ] `GlobalExceptionHandler` registered with `@Order(-2)`
- [ ] All 13 exception types mapped to correct status + code
- [ ] Validation violations populated for 400 VALIDATION_ERROR
- [ ] 4xx logged at WARN; 5xx at ERROR with full stack
- [ ] No stack trace or internal class name in client response
- [ ] Generic safe message for 500 responses
