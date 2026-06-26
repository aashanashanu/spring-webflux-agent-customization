---
name: create-api
description: Build or evolve Spring Boot/WebFlux REST APIs with architecture-aware endpoint design, DTO contracts, and validation.
---

# Skill: Create API

## When to use
- Add new endpoint sets.
- Evolve existing API versions.
- Add resource handlers aligned to existing architecture style.

## Inputs
- Resource name and version.
- Operations (GET/POST/PUT/PATCH/DELETE).
- Architecture style (Layered/Hexagonal/Event-Driven/DDD).
- Existing project conventions from architecture review.

## Templates
### Controller skeleton
```java
@RestController
@RequestMapping("/api/v1/orders")
@Validated
public class OrderController {
    private final OrderService service;

    public OrderController(OrderService service) {
        this.service = service;
    }

    @PostMapping
    public Mono<ResponseEntity<OrderResponse>> create(@Valid @RequestBody OrderCreateRequest request) {
        return service.create(request)
            .map(response -> ResponseEntity.status(HttpStatus.CREATED).body(response));
    }
}
```

### DTO skeleton
```java
public record OrderCreateRequest(
    @NotBlank String customerId,
    @NotNull @Positive BigDecimal total
) {}

public record OrderResponse(String id, String customerId, BigDecimal total, String status) {}
```

## Best practices
- Keep controllers thin and reactive.
- Version routes explicitly.
- Use dedicated request and response DTOs.
- Surface domain exceptions; let global handler map HTTP responses.

## Implementation guidance
- Layered: controller calls service directly.
- Hexagonal: controller calls inbound port.
- DDD: controller delegates to application service, not aggregate internals.
- Event-driven: pair synchronous endpoint with asynchronous event publication where required.

## Example
- Create `/api/v1/payments` with POST and GET by id using `Mono<ResponseEntity<...>>` return types.

## Validation criteria
- All controller methods return `Mono`/`Flux`.
- No blocking calls.
- Request DTOs contain validation annotations.
- Endpoint path and naming follow project conventions.
