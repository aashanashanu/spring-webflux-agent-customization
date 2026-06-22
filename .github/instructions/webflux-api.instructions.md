---
description: "Use when implementing or modifying WebFlux REST controllers, handlers, services, or DTO classes."
applyTo: "**/*Controller.java,**/*Handler.java,**/*Service.java,**/*ServiceImpl.java,**/dto/**/*.java,**/model/**/*.java"
---

# WebFlux REST API Instructions

## Controller Rules
- Annotate with `@RestController` + `@RequestMapping("/api/v1/<resource>")`.
- Mark input parameters with `@Valid`; add `@Validated` at class level for path/query params.
- Return `Mono<ResponseEntity<T>>` for single resources.
- Return `Flux<T>` for streaming responses; `Mono<ResponseEntity<List<T>>>` for paginated lists.
- Use `HttpStatus` constants — never hardcode integer status codes.
- Propagate domain exceptions up and let the global exception handler produce HTTP responses.

## Service Layer Rules
- All public methods must return `Mono<T>` or `Flux<T>`.
- Throw domain exceptions (e.g., `ResourceNotFoundException`, `DuplicateResourceException`) instead of returning null or empty.
- Map persistence exceptions to domain exceptions in the service — never expose Couchbase/R2DBC/JDBC exceptions to the controller.
- Do not use `.block()`, `.blockFirst()`, or `.blockLast()` anywhere.

## DTO Rules
- Use Java records or `@Value`-annotated classes for immutability.
- Add field-level validation annotations (`@NotNull`, `@NotBlank`, `@Size`, `@Pattern`, etc.).
- Separate request DTOs from response DTOs.
- Never expose internal entity or document classes directly.

## Reactive Patterns
- Use `flatMap` for async chaining, `map` for synchronous transformation.
- Use `switchIfEmpty` + `Mono.error(...)` to handle missing resources.
- Use `.onErrorMap` to translate low-level exceptions to domain exceptions.
- Prefer `zipWith`/`zip` over nested `flatMap` for parallel operations.

## Versioning & Routes
- Version all routes: `/api/v1/...`
- Use plural nouns for resource paths: `/api/v1/orders`, `/api/v1/users`.
- Use HTTP methods semantically: `GET` (read), `POST` (create), `PUT/PATCH` (update), `DELETE` (delete).
