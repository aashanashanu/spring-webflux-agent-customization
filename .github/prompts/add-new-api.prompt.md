---
agent: agent
description: "Guided prompt for adding a new versioned REST API endpoint to a Spring Boot WebFlux service. Enforces DTO design, validation, inline documentation, and reactive best practices."
tools:
  - read_file
  - file_search
  - grep_search
  - create_file
  - replace_string_in_file
  - multi_replace_string_in_file
---

# Add New API Endpoint

You are implementing a new REST API endpoint in a **Spring Boot WebFlux** service.  
Follow every rule below exactly. Do not skip steps or make assumptions — ask if anything is unclear.

---

## Step 1 — Gather Requirements

Before writing any code, confirm the following with the developer:

1. **Resource name** — e.g. `Order`, `Payment`, `Product` (singular PascalCase)
2. **HTTP operations** — which of: `GET (list)`, `GET (by ID)`, `POST (create)`, `PUT (full update)`, `PATCH (partial update)`, `DELETE`
3. **Request fields** — field names, types, required/optional, validation constraints
4. **Response fields** — which entity fields are safe to expose in the response
5. **Access control** — public or authenticated? Any specific role requirement?
6. **API version** — default to `v1` unless specified otherwise

---

## Step 2 — API Versioning Contract

- All routes MUST be versioned: `/api/v{n}/<resource-plural>` (e.g. `/api/v1/orders`)
- Place the version in `@RequestMapping` at class level, never per-method
- If a breaking change is needed, create a new versioned controller class (e.g. `OrderV2Controller`)
- Never change an existing versioned endpoint's response contract; add a new version

```java
// Good — versioned at class level
@RestController
@RequestMapping("/api/v1/orders")
@Validated
public class OrderController { ... }
```

---

## Step 3 — DTO Design Rules

Create **separate** DTO classes for each purpose. Never share request and response DTOs.

### Request DTOs

- Use Java **records** for immutability
- Every field that the client sends must have at least one validation annotation
- Use `@NotNull`, `@NotBlank`, `@Size`, `@Min`, `@Max`, `@Pattern`, `@Email`, `@Valid` (for nested objects)
- Add a Javadoc comment on the record and on any non-obvious field

```java
/**
 * Request payload for creating a new order.
 * All fields are mandatory unless marked with @Nullable.
 */
public record CreateOrderRequest(

    /** Customer identifier — must be a non-blank UUID string. */
    @NotBlank(message = "customerId is required")
    @Pattern(regexp = "^[0-9a-fA-F\\-]{36}$", message = "customerId must be a valid UUID")
    String customerId,

    /** Total amount — must be greater than zero. */
    @NotNull(message = "amount is required")
    @Positive(message = "amount must be positive")
    BigDecimal amount
) {}
```

### Response DTOs

- Only expose fields safe for the client (never internal IDs, version counters, audit metadata unless needed)
- Use records or immutable classes
- Add a Javadoc on the record listing which API version introduced it

```java
/**
 * Response payload for an Order resource.
 * @since API v1
 */
public record OrderResponse(
    String id,
    String customerId,
    String status,
    BigDecimal amount,
    Instant createdAt
) {}
```

---

## Step 4 — Controller Implementation Rules

- Class-level annotations: `@RestController`, `@RequestMapping("/api/v1/<resources>")`, `@Validated`
- Method-level annotations: `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`
- Always use `@Valid` on `@RequestBody` parameters
- Return types:
  - Single resource → `Mono<ResponseEntity<ResourceResponse>>`
  - Collection → `Flux<ResourceResponse>` or `Mono<ResponseEntity<List<ResourceResponse>>>`
  - Create → `Mono<ResponseEntity<ResourceResponse>>` with `ResponseEntity.created(URI)` and `201`
  - Delete → `Mono<ResponseEntity<Void>>` with `204 NO_CONTENT`
- Add an `@Operation` annotation (SpringDoc) on each method with `summary` and `description`
- Add `@ApiResponse` annotations for all expected status codes (200, 201, 400, 401, 403, 404 etc.)
- **Never** put business logic in the controller — delegate to the service layer

```java
/**
 * REST controller for the Order resource.
 * Exposes CRUD operations under /api/v1/orders.
 */
@RestController
@RequestMapping("/api/v1/orders")
@Validated
@Tag(name = "Orders", description = "Order management API")
public class OrderController {

    private final OrderService orderService;

    // Constructor injection — prefer over @Autowired
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    /**
     * Creates a new order.
     *
     * @param request validated creation payload
     * @return 201 Created with the new order resource, or 400/422 on invalid input
     */
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @Operation(summary = "Create order", description = "Creates a new order for the given customer.")
    @ApiResponse(responseCode = "201", description = "Order created successfully")
    @ApiResponse(responseCode = "400", description = "Validation error — see 'violations' in response body")
    @ApiResponse(responseCode = "401", description = "Unauthenticated")
    public Mono<ResponseEntity<OrderResponse>> createOrder(@Valid @RequestBody CreateOrderRequest request) {
        // Delegate entirely to service — no business logic here
        return orderService.create(request)
            .map(order -> ResponseEntity
                .created(URI.create("/api/v1/orders/" + order.id()))
                .body(order));
    }
}
```

---

## Step 5 — Service Layer Rules

- Define an interface (`OrderService`) and an implementation (`OrderServiceImpl`)
- All public methods must return `Mono<T>` or `Flux<T>`
- Throw domain exceptions from the service, not HTTP exceptions:
  - Missing resource → `throw new ResourceNotFoundException("Order", id)`
  - Duplicate → `throw new DuplicateResourceException("Order", field, value)`
- Chain reactive operators correctly; do NOT call `.block()`
- Add Javadoc on all public interface methods

```java
public interface OrderService {
    /** Creates an order and returns the persisted representation. */
    Mono<OrderResponse> create(CreateOrderRequest request);

    /** Returns the order by ID or signals ResourceNotFoundException if not found. */
    Mono<OrderResponse> findById(String id);
}
```

---

## Step 6 — Inline Comment Standards

Add comments only for non-obvious logic. Specifically required:

| Location | Required comment |
|----------|-----------------|
| `permitAll()` route in SecurityConfig | Explain WHY the route is public |
| `CSRF disabled` | Always add the standard comment |
| `switchIfEmpty(Mono.error(...))` | Explain the missing resource scenario |
| `.onErrorMap(...)` | Explain the exception being translated |
| Sensitive field masking | Comment that masking is intentional |
| Custom `@Query` on repository | Describe what the query does |

---

## Step 7 — Security Route Registration

After creating the controller, update `SecurityConfig.java`:

- Protected resource → add `.pathMatchers("/api/v1/<resources>/**").authenticated()`
- Public resource → add `.pathMatchers(HttpMethod.GET, "/api/v1/<resources>").permitAll()` with an explanatory comment
- Actuator and Swagger/OpenAPI paths must already be permitted — do not remove them:
  ```java
  .pathMatchers("/actuator/health", "/actuator/info").permitAll()
  .pathMatchers("/v3/api-docs/**", "/swagger-ui/**", "/swagger-ui.html").permitAll()
  ```

---

## Step 8 — Checklist Before Completing

Verify every item before declaring the task done:

- [ ] Route is versioned (`/api/v1/...`)
- [ ] Separate Request and Response DTOs created as records
- [ ] All request fields have validation annotations with messages
- [ ] Controller uses `@Valid` on `@RequestBody`
- [ ] Controller methods return reactive types (`Mono`/`Flux`)
- [ ] No business logic in the controller
- [ ] Service interface and implementation created
- [ ] Service methods return `Mono`/`Flux`, no `.block()` calls
- [ ] Domain exceptions thrown (not HTTP exceptions)
- [ ] `@Operation` and `@ApiResponse` annotations added to each controller method
- [ ] `SecurityConfig` updated with route authorization rule
- [ ] Inline comments added at required locations
- [ ] Integration test stub generated (delegate to `testcontainers-test` agent if needed)
