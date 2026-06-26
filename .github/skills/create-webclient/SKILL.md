---
name: create-webclient
description: Build resilient external REST integrations with WebClient in Spring WebFlux.
---

# Skill: Create WebClient

## Templates
### WebClient bean
```java
@Configuration
public class ClientConfig {
    @Bean
    WebClient partnerClient(WebClient.Builder builder) {
        return builder.baseUrl("https://partner.example.com").build();
    }
}
```

### Integration call
```java
public Mono<PartnerResponse> getPartnerOrder(String id) {
    return partnerClient.get()
        .uri(uriBuilder -> uriBuilder.path("/orders/{id}").build(id))
        .retrieve()
        .bodyToMono(PartnerResponse.class)
        .timeout(Duration.ofSeconds(3))
        .retryWhen(Retry.backoff(2, Duration.ofMillis(200)));
}
```

## Best practices
- Apply timeout and bounded retries.
- Map HTTP errors to domain/integration exceptions.
- Propagate correlation IDs via headers.

## Implementation guidance
- Use shared builder configuration for observability and auth.
- Align retry behavior with idempotency and business semantics.

## Example
- Add downstream payment status fetch with fallback path.

## Validation criteria
- Timeout and retry policy are explicit.
- Errors are translated and not leaked raw.
- Correlation headers propagated.
