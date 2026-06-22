---
description: "Use when adding or modifying logback configuration, request-response logging filters, log rotation policy, or structured log output."
applyTo: "**/logback*.xml,**/*LoggingFilter.java,**/*WebFilter.java,**/src/main/resources/logback*.xml"
---

# Logging & Observability Instructions

## Logback XML Setup
- Use `logback-spring.xml` (Spring profile-aware, NOT `logback.xml`).
- Define a `RollingFileAppender` named `FILE` with:
  ```xml
  <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
    <fileNamePattern>logs/app-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
    <maxFileSize>50MB</maxFileSize>
    <maxHistory>30</maxHistory>
    <totalSizeCap>2GB</totalSizeCap>
  </rollingPolicy>
  ```
- Console appender active only under Spring profiles: `local`, `test`.
- File appender active under: `dev`, `staging`, `prod`.
- Root log level: `WARN`; application packages: `INFO`; override per profile.
- Use JSON encoder for structured logging in non-local profiles (e.g., Logstash encoder).

## Request/Response Logging Filter
- Implement as a `WebFilter` bean with `@Order(Ordered.HIGHEST_PRECEDENCE + 1)`.
- Log on each request:
  - HTTP method, path, query string
  - Correlation/trace ID (from `X-Trace-Id` header or generated UUID if absent)
  - Client IP (masked last octet for GDPR compliance)
- Log on each response:
  - HTTP status code
  - Latency in milliseconds
  - Trace ID

## Sensitive Field Masking
- Mask or redact the following fields in any logged payload:
  `password`, `authorization`, `token`, `secret`, `apiKey`, `creditCard`, `cvv`, `ssn`, `pin`
- Use a regex-based masking utility applied before writing log statements.
- Do NOT log raw request bodies by default; enable only via explicit feature flag (`logging.request-body.enabled=true`).

## Trace ID Propagation
- Extract `X-Trace-Id` from incoming request headers; generate one if absent.
- Populate into MDC: `MDC.put("traceId", traceId)`.
- Propagate through Reactor context using `Context.of("traceId", traceId)`.
- Include `traceId` in all log statements and error responses.

## Log Level Guidelines
- `TRACE`: internal reactive pipeline steps (disabled in prod).
- `DEBUG`: service method entry/exit (disabled in prod).
- `INFO`: significant business events (request received, entity created/updated).
- `WARN`: 4xx errors, recoverable issues, retried operations.
- `ERROR`: 5xx errors, unhandled exceptions, connectivity failures.
