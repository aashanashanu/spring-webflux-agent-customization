---
name: create-eventbridge-integration
description: Integrate AWS EventBridge publishers/consumers for event-driven enterprise workflows.
---

# Skill: Create EventBridge Integration

## Templates
```java
public Mono<Void> publishEvent(String detailType, String source, String payloadJson) {
    return Mono.fromRunnable(() -> {
        // build PutEventsRequest and submit
    });
}
```

## Best practices
- Use explicit event source and detail type versioning.
- Validate event envelope contracts.
- Handle partial publish failures.

## Implementation guidance
- Define routing rules and targets per domain capability.
- Ensure retry/backoff strategy for transient AWS failures.
- Preserve trace metadata in event detail.

## Validation criteria
- Event contract documented and versioned.
- Publish error handling exists.
- Integration tests cover success and failure paths.
