---
agent: agent
description: "Guided prompt for diagnosing and fixing bugs in a Spring Boot WebFlux service. Enforces root-cause analysis, minimal-impact fixes, reactive safety, test coverage, and regression checks."
tools:
  - read_file
  - file_search
  - grep_search
  - semantic_search
  - replace_string_in_file
  - multi_replace_string_in_file
---

# Bug Fix

You are diagnosing and fixing a bug in a **Spring Boot WebFlux** service.  
Follow every phase below. Do not apply a fix until the root cause is confirmed.

---

## Phase 1 — Understand the Bug

Before touching any code, gather the following:

1. **Symptom** — what is the observed incorrect behaviour?
   - Wrong HTTP status returned?
   - Wrong response body / field values?
   - Exception thrown unexpectedly (or not thrown when it should be)?
   - Reactive stream completes without emitting a value?
   - Performance issue (timeout, slow response)?

2. **Reproduction steps** — exact request (method, path, body, headers) that triggers the bug

3. **Expected behaviour** — what should happen according to the API contract?

4. **Actual behaviour** — what is currently happening (log output, error payload, status code)?

5. **Scope** — is this isolated to one endpoint/service or does it affect multiple layers?

---

## Phase 2 — Locate the Root Cause

Use the following investigative order. Work top-down through the reactive call chain:

```
Request
  → WebFilter (logging, auth)
  → SecurityWebFilterChain
  → Controller (validation, mapping)
  → Service (business logic, error handling)
  → Repository (query, data mapping)
  → Global Exception Handler (error response shaping)
Response
```

### Checklist — common WebFlux bug patterns

| Pattern | What to look for |
|---------|-----------------|
| Missing resource not signalled | `findById` returns `Mono.empty()` instead of `Mono.error(ResourceNotFoundException)` |
| Wrong HTTP status | Exception not mapped in `GlobalExceptionHandler`; check the status-code table |
| Validation silently skipped | Missing `@Valid` on `@RequestBody`; missing `@Validated` on controller class |
| Reactive pipeline never executes | `Mono`/`Flux` constructed but not subscribed (return chain broken) |
| Blocking call in reactive path | `.block()`, `.blockFirst()`, JDBC call inside a reactive operator |
| Error swallowed | `.onErrorReturn(null)` or empty `onErrorResume` eating the signal |
| Sensitive field leak | Exception message or stack trace reaching the client response body |
| Thread context loss | MDC/traceId not propagated — missing `contextWrite` or Reactor Context setup |
| Incorrect error code | `code` field in error response doesn't match the exception-to-code mapping table |
| Security bypass | Route not covered by `SecurityWebFilterChain`; open `permitAll()` added by mistake |

### Investigation commands

Search for the relevant files before making assumptions:

```
file_search: **/GlobalExceptionHandler*.java
file_search: **/*Controller.java
file_search: **/*Service*.java
grep_search: <method or field name from the stack trace>
```

---

## Phase 3 — Apply the Fix

### Fix constraints (MUST follow all of these)

- **Minimal change** — fix only what is broken; do not refactor surrounding code unless it is directly causing the bug
- **No new blocking calls** — never introduce `.block()`, `Thread.sleep()`, or JDBC inside a reactive pipeline
- **Preserve existing API contract** — do not change response field names, types, or HTTP status codes for existing callers
- **Maintain error payload contract** — all error responses must keep the standard shape:
  ```json
  {
    "timestamp": "...",
    "path": "...",
    "status": 404,
    "error": "Not Found",
    "code": "RESOURCE_NOT_FOUND",
    "message": "...",
    "traceId": "..."
  }
  ```
- **Do not leak internals** — exception messages, class names, or stack traces must never reach the client
- **Preserve security rules** — do not change `SecurityWebFilterChain` route rules except to fix a security bug

### Fix procedure

1. State the root cause in one sentence before writing any code
2. Identify the exact file(s) and line(s) to change
3. Apply the fix using `replace_string_in_file` or `multi_replace_string_in_file`
4. Add an inline comment on the changed line(s) explaining what was wrong and why the fix is correct:
   ```java
   // FIX: previously returned Mono.empty() causing the downstream pipeline to complete
   // without a value — changed to signal ResourceNotFoundException so GlobalExceptionHandler
   // maps it to 404 RESOURCE_NOT_FOUND.
   return repository.findById(id)
       .switchIfEmpty(Mono.error(new ResourceNotFoundException("Order", id)));
   ```

---

## Phase 4 — Verify the Fix

### Regression check

Before declaring the fix complete, confirm:

- [ ] The specific endpoint/flow that was broken now behaves correctly
- [ ] Adjacent endpoints in the same controller are not affected
- [ ] The global exception handler still maps all exceptions correctly
- [ ] No `.block()` or synchronous call was introduced
- [ ] The fix does not change any field name or HTTP status that existing callers depend on

### Test coverage

If an integration test for this scenario does not exist, create one:

```java
@Test
void should_return404_when_orderNotFound() {
    webTestClient.get()
        .uri("/api/v1/orders/NON_EXISTENT_ID")
        .headers(h -> h.setBearerAuth(validToken))
        .exchange()
        .expectStatus().isNotFound()
        .expectBody()
        .jsonPath("$.code").isEqualTo("RESOURCE_NOT_FOUND")
        .jsonPath("$.traceId").isNotEmpty();
}
```

Test naming convention: `should_<expectedOutcome>_when_<condition>`

If a test already exists for this scenario, check why it did not catch the bug and fix the test accordingly.

---

## Phase 5 — Security Review

After every bug fix, confirm:

- [ ] No stack trace, internal class name, or raw exception message is exposed to the client
- [ ] No hardcoded credential, token, or secret was introduced
- [ ] No `permitAll()` route was added or changed without explicit justification
- [ ] Sensitive log fields (password, token, authorization) remain masked
- [ ] If the bug was a security issue, escalate the fix description and add a regression test that proves the vulnerability is closed

---

## Phase 6 — Fix Summary

Provide a concise summary in the following format:

```
Root cause: <one sentence>
Files changed: <list>
What changed: <brief description of each change>
Tests added/updated: <yes/no + test method name>
Regression risk: <none / low / medium — with brief explanation>
```
