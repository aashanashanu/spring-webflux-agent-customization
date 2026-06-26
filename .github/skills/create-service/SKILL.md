---
name: create-service
description: Implement service/application logic for Spring Boot and WebFlux with architecture-aware boundaries.
---

# Skill: Create Service

## Templates
### Service interface
```java
public interface OrderService {
    Mono<OrderResponse> create(OrderCreateRequest request);
    Mono<OrderResponse> getById(String id);
}
```

### Service implementation
```java
@Service
public class OrderServiceImpl implements OrderService {
    private final OrderRepository repository;

    public OrderServiceImpl(OrderRepository repository) {
        this.repository = repository;
    }

    @Override
    public Mono<OrderResponse> getById(String id) {
        return repository.findById(id)
            .switchIfEmpty(Mono.error(new ResourceNotFoundException("Order not found")))
            .map(this::toResponse);
    }
}
```

## Best practices
- Constructor injection only.
- Keep methods single-purpose and side-effect aware.
- Translate low-level errors with `onErrorMap`.

## Implementation guidance
- Layered: business logic in service.
- Hexagonal: service may represent application service that uses ports.
- DDD: service coordinates aggregates and domain services.
- Event-driven: publish domain events after state transitions.

## Example
- Add `cancel(orderId)` with state transition checks and event emission.

## Validation criteria
- Public methods return reactive types.
- Domain exceptions used for not found/conflict/rule violations.
- No `.block()` or thread sleeps.
