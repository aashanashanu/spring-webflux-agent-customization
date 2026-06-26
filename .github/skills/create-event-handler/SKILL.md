---
name: create-event-handler
description: Implement event handlers for asynchronous workflows in enterprise and AI-agent applications.
---

# Skill: Create Event Handler

## Templates
### Event handler service
```java
@Component
public class OrderCreatedEventHandler {
    public Mono<Void> handle(OrderCreatedEvent event) {
        return Mono.fromRunnable(() -> {
            // process event
        });
    }
}
```

## Best practices
- Make handlers idempotent.
- Use deduplication keys when available.
- Separate deserialization, validation, and processing.

## Implementation guidance
- Layered: handler delegates to service.
- Hexagonal: handler is adapter feeding application port.
- Event-driven: include dead-letter and retry strategy.
- DDD: map events to aggregate/application actions.

## Example
- Handle `PaymentCaptured` event to mark order as paid.

## Validation criteria
- Handler processing is idempotent.
- Failure path is explicit (retry, DLQ, or park).
- Observability hooks (trace/log/metric) are included.
