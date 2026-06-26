---
name: create-kafka-consumer
description: Implement Kafka consumer flows with reliability and observability controls.
---

# Skill: Create Kafka Consumer

## Templates
```java
@KafkaListener(topics = "orders.created", groupId = "order-service")
public void onMessage(String payload) {
    // deserialize, validate, process, ack
}
```

## Best practices
- Idempotent message handling.
- Explicit retry and dead-letter strategy.
- Schema evolution compatibility checks.

## Implementation guidance
- Validate payload before side effects.
- Track processing latency and failure metrics.
- Preserve ordering assumptions only where required.

## Validation criteria
- Consumer group/topic naming follows conventions.
- Error handling and DLQ strategy present.
- Tests include malformed and duplicate events.
