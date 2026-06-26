---
name: write-unit-tests
description: Create deterministic unit tests for services, mappers, and utility components.
---

# Skill: Write Unit Tests

## Templates
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock OrderRepository repository;
    @InjectMocks OrderServiceImpl service;

    @Test
    void should_returnOrder_when_exists() {
        when(repository.findById("1")).thenReturn(Mono.just(new OrderEntity()));
        StepVerifier.create(service.getById("1"))
            .expectNextCount(1)
            .verifyComplete();
    }
}
```

## Best practices
- Test behavior, not implementation details.
- Keep tests fast and isolated.
- Use StepVerifier for reactive assertions.

## Validation criteria
- Positive and negative scenarios covered.
- Reactive completion/error signals asserted.
- Naming convention `should_<outcome>_when_<condition>` used.
