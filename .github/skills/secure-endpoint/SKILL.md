---
name: secure-endpoint
description: Apply endpoint security patterns for Spring Security WebFlux and enterprise APIs.
---

# Skill: Secure Endpoint

## Templates
### Route policy snippet
```java
.authorizeExchange(auth -> auth
    .pathMatchers("/api/v1/public/**").permitAll()
    .pathMatchers("/api/v1/admin/**").hasRole("ADMIN")
    .anyExchange().authenticated())
```

### Security headers snippet
```java
.headers(headers -> headers
    .contentTypeOptions(Customizer.withDefaults())
    .frameOptions(frame -> frame.mode(ServerHttpSecurity.HeaderSpec.FrameOptionsSpec.Mode.DENY)))
```

## Best practices
- Deny-by-default authorization.
- Principle of least privilege.
- Generic 401/403 response messages.

## Implementation guidance
- Preserve existing auth model in existing projects.
- Apply endpoint-specific scopes/roles for new APIs.
- Coordinate with observability for auth events and traceability.

## Validation criteria
- Public endpoints are explicit and justified.
- Sensitive routes require auth.
- Security headers are applied consistently.
