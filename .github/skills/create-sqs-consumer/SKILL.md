---
name: create-sqs-consumer
description: Implement Amazon SQS consumer workflows with visibility timeout and redrive handling.
---

# Skill: Create SQS Consumer

## Templates
```java
public Mono<Void> consume(Message message) {
    return Mono.fromRunnable(() -> {
        // parse payload, process, acknowledge/delete
    });
}
```

## Best practices
- Align visibility timeout with processing budget.
- Use DLQ redrive policies.
- Ensure idempotent processing.

## Implementation guidance
- Use structured envelope metadata.
- Track retry count and poison message handling.
- Include correlation IDs for trace continuity.

## Validation criteria
- Queue, DLQ, and retry strategy documented.
- Duplicate message handling verified.
- Failure telemetry included.
