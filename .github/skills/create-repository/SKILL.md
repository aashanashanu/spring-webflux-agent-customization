---
name: create-repository
description: Implement reactive persistence repositories for SQL/NoSQL while matching project conventions.
---

# Skill: Create Repository

## Inputs
- Storage choice: SQL (R2DBC), Couchbase, or MongoDB.
- Entity/document and identifier strategy.
- Existing naming and package conventions.

## Templates
### R2DBC repository
```java
public interface OrderRepository extends ReactiveCrudRepository<OrderEntity, Long> {
    Flux<OrderEntity> findByCustomerId(String customerId);
}
```

### Mongo repository
```java
public interface OrderRepository extends ReactiveMongoRepository<OrderDocument, String> {
    Flux<OrderDocument> findByStatus(String status);
}
```

### Couchbase repository
```java
public interface OrderRepository extends ReactiveCouchbaseRepository<OrderDocument, String> {
    Flux<OrderDocument> findByCustomerId(String customerId);
}
```

## Best practices
- Keep repositories persistence-focused.
- Do not place business logic in repositories.
- Use explicit query methods only when necessary.

## Implementation guidance
- Layered: repository called from service.
- Hexagonal: repository is adapter implementing outbound port.
- DDD: repositories align to aggregate roots.
- Event-driven: support idempotent reads/writes for consumer flows.

## Validation criteria
- Repository signatures are reactive.
- Chosen technology matches project storage stack.
- Query names and model classes match conventions.
