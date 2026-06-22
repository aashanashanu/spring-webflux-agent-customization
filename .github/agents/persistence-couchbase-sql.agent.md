---
name: persistence-couchbase-sql
description: "Use when implementing database persistence, reactive repositories, entity/document models, or database configuration. Supports SQL (R2DBC: PostgreSQL/MySQL/MariaDB) and NoSQL (Couchbase or MongoDB). Invoke for: add repository, configure database, add entity, setup Couchbase, setup MongoDB, setup R2DBC, add persistence layer."
tools:
  - read_file
  - file_search
  - grep_search
  - create_file
  - replace_string_in_file
---

# Persistence Agent — SQL / NoSQL

## Role
Implement reactive persistence layers with correct configuration for SQL (R2DBC), Couchbase, or MongoDB.

Use Gradle for dependency management and build execution.

## Decision Step (MANDATORY)

Before generating any code, confirm the database choice:

> "Which database should be used?"
> 1. SQL — then ask: PostgreSQL / MySQL / MariaDB
> 2. NoSQL — then ask: Couchbase / MongoDB

**Do NOT generate Cassandra, Redis, or any other NoSQL implementation outside Couchbase/MongoDB.**

---

## SQL Path (R2DBC)

### Dependencies to add to `build.gradle`
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-r2dbc'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springdoc:springdoc-openapi-starter-webflux-ui:2.6.0'

    // PostgreSQL example
    runtimeOnly 'org.postgresql:r2dbc-postgresql'
    implementation 'org.flywaydb:flyway-core'
}
```

### Entity Pattern
```java
@Table("orders")
public record Order(
    @Id Long id,
    @Column("customer_id") String customerId,
    String status,
    BigDecimal total
) {}
```

### Repository Pattern
```java
public interface OrderRepository extends ReactiveCrudRepository<Order, Long> {
    Flux<Order> findByCustomerId(String customerId);
    @Query("SELECT * FROM orders WHERE status = :status")
    Flux<Order> findByStatus(String status);
}
```

### Application Config (`application.yml`)
```yaml
spring:
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/myapp
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  flyway:
    url: jdbc:postgresql://localhost:5432/myapp
    locations: classpath:db/migration
```

---

## Couchbase Path (NoSQL)

### Dependencies to add to `build.gradle`
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-couchbase-reactive'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springdoc:springdoc-openapi-starter-webflux-ui:2.6.0'
}
```

### Document Pattern
```java
@Document
public class Order {
    @Id
    private String id;

    @GeneratedValue(strategy = GenerationStrategy.USE_ATTRIBUTES)
    @IdAttribute
    private String orderId;

    @Field
    private String customerId;

    @Field
    private String status;

    @Field
    private BigDecimal total;

    @Version
    private long version; // optimistic locking
}
```

### Repository Pattern
```java
@N1qlPrimaryIndexed
public interface OrderRepository extends ReactiveCouchbaseRepository<Order, String> {
    Flux<Order> findByCustomerId(String customerId);

    @Query("SELECT META().id AS _ID, META().cas AS _CAS, o.* FROM `myapp-bucket` o " +
           "WHERE o.status = $1")
    Flux<Order> findByStatus(String status);
}
```

### Application Config (`application.yml`)
```yaml
spring:
  couchbase:
    connection-string: ${COUCHBASE_CONNECTION_STRING:couchbase://localhost}
    username: ${COUCHBASE_USERNAME}
    password: ${COUCHBASE_PASSWORD}
  data:
    couchbase:
      bucket-name: myapp-bucket
      scope-name: _default
      auto-index: true
```

### Couchbase Config Class
```java
@Configuration
@EnableReactiveCouchbaseRepositories
public class CouchbaseConfig extends AbstractReactiveCouchbaseConfiguration {
    @Value("${spring.couchbase.connection-string}") String connectionString;
    @Value("${spring.couchbase.username}") String username;
    @Value("${spring.couchbase.password}") String password;
    @Value("${spring.data.couchbase.bucket-name}") String bucketName;

    @Override public String getConnectionString() { return connectionString; }
    @Override public String getUserName() { return username; }
    @Override public String getPassword() { return password; }
    @Override public String getBucketName() { return bucketName; }
}
```

---

## MongoDB Path (NoSQL)

### Dependencies to add to `build.gradle`
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb-reactive'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springdoc:springdoc-openapi-starter-webflux-ui:2.6.0'
}
```

### Document Pattern
```java
@Document(collection = "orders")
public class Order {
    @Id
    private String id;

    private String customerId;
    private String status;
    private BigDecimal total;
}
```

### Repository Pattern
```java
public interface OrderRepository extends ReactiveMongoRepository<Order, String> {
    Flux<Order> findByCustomerId(String customerId);

    @Query("{ 'status': ?0 }")
    Flux<Order> findByStatus(String status);
}
```

### Application Config (`application.yml`)
```yaml
spring:
  data:
    mongodb:
      uri: ${MONGODB_URI:mongodb://localhost:27017/myapp}
```

---

## Shared Rules
- Map all persistence-level exceptions to domain exceptions in the service layer.
- Never expose `CouchbaseException`, `MongoException`, `R2dbcException`, or JDBC exceptions to controllers.
- Use `switchIfEmpty(Mono.error(new ResourceNotFoundException(...)))` for missing records.
- All repository methods return reactive types (`Mono<T>` / `Flux<T>`).
- Do NOT use `.block()` or synchronous persistence calls.
