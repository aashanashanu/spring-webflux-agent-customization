---
name: create-kafka-producer
description: Implement Kafka producer flows with delivery guarantees and traceability.
---

# Skill: Create Kafka Producer

## Templates
```java
public Mono<Void> publishOrderCreated(OrderCreatedEvent event) {
    return Mono.fromRunnable(() -> kafkaTemplate.send("orders.created", event.getOrderId(), event));
}
```

## Best practices
- Use explicit keys for partition affinity.
- Include event version and metadata.
- Propagate correlation IDs.

## Implementation guidance
- Define event schema ownership.
- Choose at-least-once semantics with idempotent consumers.
- Add fallback path for producer errors.

## Validation criteria
- Topic and key strategy are documented.
- Publish failures are handled.
- Integration tests verify broker interaction path.
