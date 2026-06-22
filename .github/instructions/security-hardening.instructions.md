---
description: "Use when configuring Spring Security for WebFlux, defining route protection, authentication, authorization, or applying HTTP security headers."
applyTo: "**/*SecurityConfig*.java,**/*Security*.java,**/application*.yml,**/application*.yaml"
---

# Spring Security Hardening Instructions

## Security Config Structure
- Use `SecurityWebFilterChain` bean (reactive) — do NOT use servlet `HttpSecurity`.
- Annotate config class with `@EnableWebFluxSecurity` and `@EnableReactiveMethodSecurity`.
- Define a single `@Bean SecurityWebFilterChain` per application context.

## Route Authorization
- **Deny by default**: end chain with `.anyExchange().authenticated()`.
- Explicitly permit public endpoints only:
  ```java
  .pathMatchers(HttpMethod.GET, "/api/v1/public/**").permitAll()
  .pathMatchers("/actuator/health", "/actuator/info").permitAll()
  ```
- Document every `permitAll()` with a comment explaining why it is public.
- Protect admin routes with explicit role checks: `.hasRole("ADMIN")`.

## CSRF
- Disable CSRF for stateless REST APIs:
  ```java
  .csrf(CsrfSpec::disable)
  ```
- Add a comment: `// CSRF disabled - stateless JWT/token API, no cookie-based sessions`.

## HTTP Security Headers
Apply the following headers in every response:
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Cache-Control: no-store, no-cache, must-revalidate`
- `Referrer-Policy: no-referrer`
- `Permissions-Policy: geolocation=(), microphone=()`
- `Strict-Transport-Security: max-age=31536000; includeSubDomains` (production only)

Use `ServerHttpSecurity.headers(...)` to apply these.

## Authentication
- Prefer JWT-based stateless authentication via a `ServerSecurityContextRepository` or custom `AuthenticationWebFilter`.
- Validate token expiry, signature, and issuer on every request.
- Never store raw passwords; use `PasswordEncoder` (BCrypt, strength ≥ 12).

## Error Behavior
- Return generic `401 UNAUTHORIZED` / `403 FORBIDDEN` messages — never expose role names or internal structure.
- Configure `ServerAuthenticationEntryPoint` and `ServerAccessDeniedHandler` to return the standard error payload defined in `exception-handling.instructions.md`.

## Secrets & Config
- Never hardcode secrets, passwords, or keys in `application.yml` or Java source.
- Use environment variables or a secrets manager (Vault, AWS Secrets Manager).
- Mark sensitive properties in config with `# SENSITIVE — load from env` comment.
