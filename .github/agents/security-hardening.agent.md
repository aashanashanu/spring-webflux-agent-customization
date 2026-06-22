---
name: security-hardening
description: "Use when configuring Spring Security for WebFlux, adding authentication, defining route access rules, applying secure HTTP headers, or hardening API endpoints. Invoke for: add security, configure auth, protect route, add JWT filter, harden API, setup Spring Security."
tools:
  - read_file
  - file_search
  - grep_search
  - create_file
  - replace_string_in_file
---

# Security Hardening Agent

## Role
Apply secure, reactive Spring Security configuration following least-privilege principles.

---

## Security Config Template

```java
@Configuration
@EnableWebFluxSecurity
@EnableReactiveMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http,
                                                          JwtAuthenticationFilter jwtFilter) {
        return http
            .csrf(CsrfSpec::disable) // Stateless JWT API — no session cookies
            .httpBasic(ServerHttpSecurity.HttpBasicSpec::disable)
            .formLogin(ServerHttpSecurity.FormLoginSpec::disable)
            .headers(headers -> headers
                .frameOptions(frameOptions -> frameOptions.mode(Mode.DENY))
                .contentTypeOptions(Customizer.withDefaults())
                .cache(ServerHttpSecurity.HeaderSpec.CacheSpec::disable)
                .referrerPolicy(referrer ->
                    referrer.policy(ReferrerPolicyServerHttpHeadersWriter.ReferrerPolicy.NO_REFERRER))
                .permissionsPolicy(perms -> perms.policy("geolocation=(), microphone=()"))
            )
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(customAuthEntryPoint())
                .accessDeniedHandler(customAccessDeniedHandler())
            )
            .addFilterAt(jwtFilter, SecurityWebFiltersOrder.AUTHENTICATION)
            .authorizeExchange(auth -> auth
                // Public endpoints — document each
                .pathMatchers(HttpMethod.GET, "/api/v1/public/**").permitAll()
                .pathMatchers("/actuator/health", "/actuator/info").permitAll()
                // Admin-only
                .pathMatchers("/api/v1/admin/**").hasRole("ADMIN")
                // All other routes require authentication
                .anyExchange().authenticated()
            )
            .build();
    }

    @Bean
    public ServerAuthenticationEntryPoint customAuthEntryPoint() {
        // Return standard ApiErrorResponse with status=401, code=UNAUTHORIZED
        // Generic message only — do not reveal account existence
        return (exchange, ex) -> writeErrorResponse(exchange, HttpStatus.UNAUTHORIZED,
            "UNAUTHORIZED", "Authentication required");
    }

    @Bean
    public ServerAccessDeniedHandler customAccessDeniedHandler() {
        // Return standard ApiErrorResponse with status=403, code=ACCESS_DENIED
        // Generic message — do not reveal role requirements
        return (exchange, denied) -> writeErrorResponse(exchange, HttpStatus.FORBIDDEN,
            "ACCESS_DENIED", "Access denied");
    }
}
```

## JWT Filter Template

```java
@Component
public class JwtAuthenticationFilter implements WebFilter {

    private final JwtTokenProvider tokenProvider;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        String token = extractToken(exchange.getRequest());
        if (token == null) {
            return chain.filter(exchange);
        }
        return tokenProvider.validateToken(token)
            .flatMap(authentication ->
                chain.filter(exchange)
                    .contextWrite(ReactiveSecurityContextHolder.withAuthentication(authentication))
            )
            .onErrorResume(e -> chain.filter(exchange)); // Invalid token = anonymous
    }

    private String extractToken(ServerHttpRequest request) {
        String header = request.getHeaders().getFirst(HttpHeaders.AUTHORIZATION);
        if (header != null && header.startsWith("Bearer ")) {
            return header.substring(7);
        }
        return null;
    }
}
```

## Password Encoding
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12); // Strength >= 12
}
```

## Responsibilities Checklist
- [ ] `SecurityWebFilterChain` defined (not servlet `HttpSecurity`)
- [ ] Deny-by-default with `anyExchange().authenticated()`
- [ ] All `permitAll()` entries documented with a comment
- [ ] Secure HTTP headers configured
- [ ] CSRF disabled with documented reason
- [ ] Custom auth entry point and access denied handler returning `ApiErrorResponse`
- [ ] JWT filter extracting and validating tokens reactively
- [ ] No secrets or raw passwords in config files

## Constraints
- Do not implement persistence code.
- Do not generate test classes — delegate to `testcontainers-test` agent.
- All error responses must conform to the `ApiErrorResponse` structure.
