# Module 3: DBMS (MySQL, PostgreSQL)

> **Scope:** SQL Optimization, JPA/Hibernate, Caching, Indexing, N+1 Problem, Connection Pooling, PostgreSQL
> **Questions:** 20 | **Critical:** 5 | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

---

### Q1. 🔴 🌐 What is the N+1 query problem in Hibernate/JPA? How do you detect and fix it?

**The N+1 problem occurs when loading a parent entity triggers N additional queries to lazily fetch associated collections — resulting in 1 query for parents + N queries for their children, which is catastrophic for performance at scale.**

**Example:**
```java
// Entity with lazy relationship
@Entity
public class Order {
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY) // LAZY is the JPA default for collections
    private List<OrderItem> items;
}

// N+1 in action:
List<Order> orders = orderRepository.findAll();  // 1 query: SELECT * FROM orders
for (Order order : orders) {
    order.getItems().size();  // N queries: SELECT * FROM order_items WHERE order_id = ?
}
// Total: 1 + N queries (if N = 1000 orders → 1001 queries!)
```

**Solutions (Ordered by Preference):**

```java
// 1. JOIN FETCH (best for known use cases)
@Query("SELECT o FROM Order o JOIN FETCH o.items WHERE o.status = :status")
List<Order> findByStatusWithItems(@Param("status") OrderStatus status);
// Result: 1 query with JOIN

// 2. @EntityGraph (declarative, reusable)
@EntityGraph(attributePaths = {"items", "items.product"})
List<Order> findByStatus(OrderStatus status);
// Result: 1 query with LEFT JOIN

// 3. @BatchSize (transparent batching — best for general-purpose)
@Entity
public class Order {
    @OneToMany(mappedBy = "order")
    @BatchSize(size = 50)  // Loads items in batches of 50 parent IDs
    private List<OrderItem> items;
}
// Result: 1 + ceil(N/50) queries — much better than 1 + N

// 4. Global batch size in application.yml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 50

// 5. DTO Projection (best for read-only views)
@Query("""
    SELECT new com.app.dto.OrderSummary(o.id, o.status, COUNT(i), SUM(i.price))
    FROM Order o LEFT JOIN o.items i
    WHERE o.status = :status
    GROUP BY o.id, o.status
    """)
List<OrderSummary> findOrderSummaries(@Param("status") OrderStatus status);
```

**Detection Tools:**
- **Hibernate statistics:** `spring.jpa.properties.hibernate.generate_statistics=true`
- **p6spy / datasource-proxy:** Logs actual SQL with execution count.
- **Hypersistence Optimizer:** Detects N+1 at test time.

**⚠️ Pitfalls:**
- `JOIN FETCH` with pagination (`Pageable`) applies pagination in memory — Hibernate fetches ALL rows then paginates in Java. Use `@BatchSize` or a two-query approach instead.
- `FetchType.EAGER` does NOT fix N+1 — it just makes the N queries happen automatically at load time.
- Multiple `JOIN FETCH` on collections produces a Cartesian product — use `@BatchSize` for multiple collections.

---

### Q2. 🔴 🌐 How do database indexes work? When should you create them, and what are the trade-offs?

**An index is a separate data structure (typically a B-tree in PostgreSQL) that maintains a sorted copy of selected columns, enabling O(log n) lookups instead of O(n) full table scans — but each index adds write overhead and storage cost.**

**B-Tree Index Internal Structure:**
```
                 [50]
               /      \
          [20, 35]    [70, 85]
         /  |  \     /   |   \
     [10] [25] [40] [60] [75] [90]
      ↓     ↓    ↓    ↓    ↓    ↓
    rows  rows  rows  rows  rows  rows
```

**PostgreSQL Index Types:**

| Index Type | Use Case | Example |
|-----------|----------|---------|
| B-Tree (default) | Equality, range, sorting | `WHERE price > 100` |
| Hash | Equality only | `WHERE email = 'x@y.com'` |
| GIN | Full-text search, arrays, JSONB | `WHERE tags @> '{java}'` |
| GiST | Geometric, range, full-text | `WHERE location <@> point(...)` |
| BRIN | Large tables with natural ordering | Time-series data |

```sql
-- Composite index (column order matters!)
CREATE INDEX idx_orders_status_created 
ON orders (status, created_at DESC);

-- Partial index (index only active orders — much smaller)
CREATE INDEX idx_orders_active 
ON orders (customer_id, created_at) 
WHERE status = 'ACTIVE';

-- Covering index (includes all needed columns — index-only scan)
CREATE INDEX idx_orders_covering 
ON orders (customer_id) 
INCLUDE (total_amount, status);

-- Expression index
CREATE INDEX idx_users_email_lower 
ON users (LOWER(email));

-- EXPLAIN ANALYZE to verify index usage
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) 
SELECT * FROM orders WHERE status = 'ACTIVE' AND created_at > NOW() - INTERVAL '7 days';
```

**When to Create:**
- ✅ Columns in `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY` clauses.
- ✅ Foreign key columns (PostgreSQL does NOT auto-index FK columns).
- ✅ High-selectivity columns (many distinct values relative to row count).
- ❌ Small tables (< 1000 rows) — sequential scan is faster.
- ❌ Columns with very low selectivity (e.g., `boolean` flags — unless partial index).
- ❌ Write-heavy tables where index maintenance exceeds read benefit.

**⚠️ Pitfalls:**
- Composite index `(A, B)` supports queries on `A` and `(A, B)` but NOT on `B` alone — leftmost prefix rule.
- Functions on indexed columns bypass the index: `WHERE UPPER(name) = 'JOHN'` → create expression index.
- Too many indexes slow down `INSERT/UPDATE/DELETE` — each write must update all indexes.
- `VACUUM` and `ANALYZE` must run regularly (autovacuum) — stale statistics → bad query plans.

---

### Q3. 🔴 🌐 What is the difference between `FetchType.LAZY` and `FetchType.EAGER` in JPA? What are the best practices?

**`LAZY` loads associated entities on first access (via proxy); `EAGER` loads them immediately with the parent — the universal best practice is to default everything to `LAZY` and explicitly fetch what you need per use case, because `EAGER` loading is a global setting you can never turn off.**

**Defaults:**

| Relationship | JPA Default |
|-------------|-------------|
| `@ManyToOne` | **EAGER** ← dangerous |
| `@OneToOne` | **EAGER** ← dangerous |
| `@OneToMany` | LAZY |
| `@ManyToMany` | LAZY |

```java
// ✅ Best practice: ALWAYS override to LAZY
@Entity
public class Order {
    @ManyToOne(fetch = FetchType.LAZY)  // Override EAGER default
    @JoinColumn(name = "customer_id")
    private Customer customer;

    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;
}

// ✅ Fetch what you need per use case
public interface OrderRepository extends JpaRepository<Order, Long> {
    // Use case 1: List orders (no items needed)
    List<Order> findByStatus(OrderStatus status);

    // Use case 2: Order detail page (items needed)
    @EntityGraph(attributePaths = {"items", "customer"})
    Optional<Order> findWithDetailsById(Long id);
}
```

**⚠️ Pitfalls:**
- `LazyInitializationException` — accessing a lazy association outside an active persistence context (session). Fix: use `@Transactional` on the service method, or fetch eagerly for that use case.
- `@OneToOne` lazy loading doesn't work on the non-owning side — Hibernate can't proxy `null`. Use `@MapsId` or restructure to `@ManyToOne`.
- `EAGER` on `@ManyToOne` causes Hibernate to execute a `LEFT JOIN` for EVERY query on that entity — even when you don't need the association.

---

### Q4. 🔴 🏢 How do you optimize slow SQL queries in PostgreSQL? Walk through your investigation process.

**Optimizing slow queries follows a systematic process: capture the slow query → analyze with `EXPLAIN ANALYZE` → identify the bottleneck (sequential scan, nested loop, sort spill) → apply the targeted fix (index, rewrite, partitioning) → verify the improvement.**

**Step-by-Step Investigation:**

```sql
-- Step 1: Enable slow query logging
ALTER SYSTEM SET log_min_duration_statement = 500; -- Log queries > 500ms
SELECT pg_reload_conf();

-- Step 2: EXPLAIN ANALYZE the slow query
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) 
SELECT o.*, c.name 
FROM orders o 
JOIN customers c ON o.customer_id = c.id 
WHERE o.status = 'ACTIVE' AND o.created_at > '2025-01-01'
ORDER BY o.created_at DESC 
LIMIT 50;

-- Step 3: Read the plan — look for:
-- ❌ Seq Scan on large tables (missing index)
-- ❌ Nested Loop with high row estimates (consider Hash Join)
-- ❌ Sort with "Sort Method: external merge" (spilling to disk)
-- ❌ Rows Removed by Filter: high number (index not selective enough)
```

**Common Optimizations:**

```sql
-- Fix 1: Add targeted composite index
CREATE INDEX CONCURRENTLY idx_orders_status_created 
ON orders (status, created_at DESC) 
WHERE status = 'ACTIVE';  -- Partial index for common filter

-- Fix 2: Rewrite correlated subquery as JOIN
-- SLOW:
SELECT * FROM orders o WHERE EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.order_id = o.id AND oi.quantity > 100
);
-- FAST:
SELECT DISTINCT o.* FROM orders o 
JOIN order_items oi ON oi.order_id = o.id 
WHERE oi.quantity > 100;

-- Fix 3: Pagination with keyset instead of OFFSET
-- SLOW (OFFSET scans and discards rows):
SELECT * FROM orders ORDER BY created_at DESC LIMIT 20 OFFSET 10000;
-- FAST (keyset pagination):
SELECT * FROM orders 
WHERE created_at < '2025-06-15T10:30:00' 
ORDER BY created_at DESC LIMIT 20;

-- Fix 4: Use MATERIALIZED VIEW for complex aggregations
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT date_trunc('day', created_at) as day, 
       SUM(total_amount) as revenue, 
       COUNT(*) as order_count
FROM orders 
GROUP BY 1;
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_sales;
```

**⚠️ Pitfalls:**
- `EXPLAIN` without `ANALYZE` shows estimates, not actuals — always use `ANALYZE` for real investigation.
- `CREATE INDEX CONCURRENTLY` — always use in production to avoid locking the table.
- `SELECT *` prevents index-only scans — select only needed columns.
- PostgreSQL query planner relies on statistics — run `ANALYZE table_name` after bulk data changes.

---

### Q5. 🔴 🏢 What is connection pooling and how does HikariCP work? How do you size the pool?

**Connection pooling maintains a cache of reusable database connections to avoid the expensive overhead of creating new connections (TCP handshake + authentication) per request — HikariCP is the default in Spring Boot and the fastest Java connection pool.**

**How HikariCP Works:**
1. Pre-creates a pool of connections (`minimumIdle` to `maximumPoolSize`).
2. When a thread requests a connection, it borrows one from the pool.
3. When the thread closes the connection (via `try-with-resources`), it's returned to the pool, not actually closed.
4. Connections are validated using test queries or JDBC4 `isValid()`.
5. Idle connections are evicted after `idleTimeout`.

**Pool Sizing Formula:**
```
Optimal pool size = (number of CPU cores × 2) + effective_spindle_count
```
For SSD-backed PostgreSQL: **pool size ≈ CPU cores × 2 + 1** (typically 10–20 connections for most apps).

```yaml
spring:
  datasource:
    url: jdbc:postgresql://db:5432/myapp
    hikari:
      maximum-pool-size: 20        # Max connections (DON'T go too high!)
      minimum-idle: 5              # Minimum idle connections maintained
      idle-timeout: 300000         # 5 min — close idle connections
      max-lifetime: 1800000        # 30 min — recycle connections before DB timeout
      connection-timeout: 5000     # 5 sec — wait for connection before exception
      leak-detection-threshold: 60000  # Warn if connection held > 60 sec
      pool-name: OrderServicePool
```

```java
// Monitor pool metrics with Micrometer
@Bean
public MeterBinder hikariMetrics(DataSource dataSource) {
    HikariDataSource hds = (HikariDataSource) dataSource;
    return new HikariCPMetrics(hds.getHikariPoolMXBean());
    // Exposes: hikari.pool.active, hikari.pool.idle, hikari.pool.pending
}
```

**⚠️ Pitfalls:**
- **More connections ≠ better performance.** PostgreSQL degrades beyond ~100 connections due to process-per-connection architecture. Use PgBouncer for connection multiplexing at scale.
- `max-lifetime` must be shorter than the database's `wait_timeout` — otherwise the pool serves stale connections.
- **Connection leak** — forgetting to close a connection in application code exhausts the pool. Enable `leak-detection-threshold`.
- **@Transactional too broadly** — a long-running method holds a connection for its entire duration. Keep transactions short.

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

---

### Q6. 🏢 What caching strategies do you use in production? How does Spring Cache + Redis work?

**Production caching typically combines L1 in-process cache (Caffeine) for hot data with L2 distributed cache (Redis) for shared state — Spring's `@Cacheable` abstraction lets you swap implementations without changing business logic.**

**Caching Strategies:**

| Strategy | Description | Use Case |
|---------|-------------|----------|
| **Cache-Aside** | App checks cache, falls back to DB, populates cache | Most common for read-heavy |
| **Write-Through** | Write to cache and DB simultaneously | Strong consistency needed |
| **Write-Behind** | Write to cache, async flush to DB | Write-heavy with eventual consistency |
| **Read-Through** | Cache auto-loads from DB on miss | Transparent caching |

```java
// Spring Cache with Redis
@Configuration
@EnableCaching
public class CacheConfig {
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30))
            .serializeValuesWith(SerializationPair.fromSerializer(
                new GenericJackson2JsonRedisSerializer()))
            .disableCachingNullValues();

        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .withCacheConfiguration("products", 
                config.entryTtl(Duration.ofHours(1)))  // Per-cache TTL
            .withCacheConfiguration("user-sessions", 
                config.entryTtl(Duration.ofMinutes(15)))
            .build();
    }
}

@Service
public class ProductService {
    @Cacheable(value = "products", key = "#id", unless = "#result == null")
    public Product findById(Long id) {
        return productRepository.findById(id).orElse(null);
    }

    @CachePut(value = "products", key = "#product.id")
    public Product update(Product product) {
        return productRepository.save(product);
    }

    @CacheEvict(value = "products", key = "#id")
    public void delete(Long id) {
        productRepository.deleteById(id);
    }

    @CacheEvict(value = "products", allEntries = true)
    @Scheduled(fixedRate = 3600000)  // Hourly full eviction
    public void evictAllProducts() {}
}
```

**⚠️ Pitfalls:**
- **Cache invalidation is the hardest problem.** Stale data causes bugs. Use TTL + event-driven eviction.
- **Self-invocation bypasses cache proxy** — calling `this.findById()` skips the cache.
- **Don't cache user-specific data in shared cache** without proper key isolation.
- **Thundering herd** — when a popular cache key expires, hundreds of requests hit the DB simultaneously. Use distributed locks or `@Cacheable` with `sync = true`.

---

### Q7. 🏬 What is the difference between `@Query`, `@NamedQuery`, and derived query methods in Spring Data JPA?

**Spring Data JPA offers three query strategies: derived query methods (auto-generated from method names for simple queries), `@Query` (inline JPQL/native SQL for complex queries), and `@NamedQuery` (entity-level pre-defined queries for reusability) — derived methods for simple, `@Query` for complex, `@NamedQuery` rarely in modern code.**

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    // 1. Derived query — generated from method name
    List<Order> findByStatusAndCreatedAtAfter(OrderStatus status, Instant after);
    // → SELECT o FROM Order o WHERE o.status = ?1 AND o.createdAt > ?2

    Optional<Order> findFirstByCustomerIdOrderByCreatedAtDesc(Long customerId);

    long countByStatus(OrderStatus status);

    // 2. @Query — JPQL (type-safe, entity-based)
    @Query("""
        SELECT o FROM Order o 
        JOIN FETCH o.items 
        WHERE o.customer.id = :customerId 
        AND o.status IN :statuses
        ORDER BY o.createdAt DESC
        """)
    List<Order> findCustomerOrdersWithItems(
        @Param("customerId") Long customerId, 
        @Param("statuses") Collection<OrderStatus> statuses);

    // 3. @Query — Native SQL (when you need PostgreSQL-specific features)
    @Query(value = """
        SELECT o.* FROM orders o 
        WHERE o.metadata @> '{"priority": "high"}'::jsonb 
        AND o.created_at > NOW() - INTERVAL '7 days'
        """, nativeQuery = true)
    List<Order> findHighPriorityRecentOrders();

    // 4. Modifying query
    @Modifying
    @Query("UPDATE Order o SET o.status = :status WHERE o.id IN :ids")
    int bulkUpdateStatus(@Param("ids") List<Long> ids, @Param("status") OrderStatus status);
}
```

**⚠️ Pitfalls:**
- `@Modifying` queries bypass the persistence context — call `@Modifying(clearAutomatically = true)` or manually clear the entity manager.
- Derived queries with 3+ conditions become unreadable — switch to `@Query`.
- Native queries return `Object[]` by default — use `@SqlResultSetMapping` or `Tuple` for type safety.

---

### Q8. 🌐 How does Hibernate's first-level cache (persistence context) and second-level cache work?

**The first-level cache (L1) is the persistence context scoped to a single session/transaction — it guarantees identity (same entity = same object); the second-level cache (L2) is shared across sessions and requires explicit configuration with providers like Ehcache or Caffeine.**

| Feature | L1 Cache | L2 Cache |
|---------|----------|----------|
| Scope | Session/Transaction | SessionFactory (global) |
| Enabled by | Default (always on) | Explicit config |
| Eviction | Session close | TTL / capacity-based |
| Identity | Guaranteed (same PK = same reference) | Not guaranteed |

```java
// L2 cache configuration
@Entity
@Cacheable                                    // JPA annotation
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE, region = "products")  // Hibernate
public class Product {
    @Id
    private Long id;
    private String name;

    @Cache(usage = CacheConcurrencyStrategy.READ_ONLY)  // Cache the collection too
    @OneToMany(mappedBy = "product")
    private List<Review> reviews;
}
```

```yaml
# application.yml
spring:
  jpa:
    properties:
      hibernate:
        cache:
          use_second_level_cache: true
          use_query_cache: true  # Also cache JPQL query results
          region:
            factory_class: org.hibernate.cache.jcache.JCacheRegionFactory
        javax:
          cache:
            provider: org.ehcache.jsr107.EhcacheCachingProvider
```

**Cache Concurrency Strategies:**
- `READ_ONLY` — immutable entities (reference data).
- `READ_WRITE` — mutable entities with strict consistency.
- `NONSTRICT_READ_WRITE` — eventual consistency (rare updates).
- `TRANSACTIONAL` — JTA transactions (XA — rarely used).

**⚠️ Pitfalls:**
- **Query cache invalidation is table-level** — ANY update to the `products` table invalidates ALL cached queries on `Product`. Only useful for rarely-changing reference data.
- L1 cache can cause memory issues for batch operations — use `entityManager.clear()` periodically.
- L2 cache does NOT cache associations by default — annotate each `@OneToMany`/`@ManyToMany` with `@Cache`.

---

### Q9. 🏬 What are JPA Projections and when should you use them over entities?

**JPA Projections retrieve only the columns you need instead of full entities — reducing memory consumption, avoiding unnecessary joins, and bypassing the persistence context overhead. Use projections for read-only views and DTOs; use entities only when you need to modify data.**

```java
// 1. Interface-based projection (Spring Data generates proxy)
public interface OrderSummary {
    Long getId();
    OrderStatus getStatus();
    BigDecimal getTotalAmount();

    @Value("#{target.customer.name}")  // SpEL for derived properties
    String getCustomerName();
}

List<OrderSummary> findByStatus(OrderStatus status);  // Returns projections, not entities

// 2. Class-based (DTO) projection — most performant
public record OrderDTO(Long id, OrderStatus status, BigDecimal totalAmount, String customerName) {}

@Query("""
    SELECT new com.app.dto.OrderDTO(o.id, o.status, o.totalAmount, c.name) 
    FROM Order o JOIN o.customer c 
    WHERE o.status = :status
    """)
List<OrderDTO> findOrderDTOs(@Param("status") OrderStatus status);

// 3. Tuple projection (ad hoc)
@Query("SELECT o.id as id, o.status as status, COUNT(i) as itemCount " +
       "FROM Order o LEFT JOIN o.items i GROUP BY o.id, o.status")
List<Tuple> findOrderStats();
```

**Performance Comparison (1000 rows):**

| Approach | Heap Usage | Query Complexity | Dirty Checking |
|----------|-----------|-----------------|----------------|
| Entity (`findAll()`) | 100% | Full SELECT * | Yes (overhead) |
| DTO Projection | ~30% | SELECT specific cols | No |
| Interface Projection | ~40% | SELECT specific cols | No (proxy overhead) |

**⚠️ Pitfall:** Interface projections generate dynamic proxies — for high-throughput scenarios, class-based projections (records) are faster.

---

### Q10. 🏬 How do you handle database migrations in production? Compare Flyway and Liquibase.

**Database migrations should be version-controlled, incremental, and applied automatically at startup — Flyway (SQL-based, simple, convention-driven) and Liquibase (XML/YAML/JSON, more flexible, rollback support) are the two main tools, with Flyway being more popular in Spring Boot projects.**

| Feature | Flyway | Liquibase |
|---------|--------|-----------|
| Format | SQL files (+ Java) | XML, YAML, JSON, SQL |
| Learning curve | Low (just write SQL) | Higher (DSL to learn) |
| Rollback | Paid feature | Built-in |
| Diff/generate | Paid | Free (diffChangeLog) |
| Spring Boot | Auto-configured | Auto-configured |

```sql
-- Flyway: db/migration/V1__create_orders.sql
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    customer_id BIGINT NOT NULL REFERENCES customers(id),
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING',
    total_amount DECIMAL(19,4) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_orders_customer_status ON orders (customer_id, status);

-- V2__add_shipping_address.sql
ALTER TABLE orders ADD COLUMN shipping_address_id BIGINT REFERENCES addresses(id);

-- V3__add_order_metadata.sql  (non-breaking: nullable column, no lock on large tables)
ALTER TABLE orders ADD COLUMN metadata JSONB;
CREATE INDEX idx_orders_metadata ON orders USING GIN (metadata);
```

```yaml
# Spring Boot auto-runs Flyway on startup
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true  # For existing databases without migration history
    validate-on-migrate: true
```

**⚠️ Pitfalls:**
- **Never modify an applied migration** — Flyway checksums detect changes and fail. Create a new migration instead.
- **`ALTER TABLE` on large tables locks the table in PostgreSQL** — use `ALTER TABLE ... ADD COLUMN ... DEFAULT` (PostgreSQL 11+ doesn't rewrite table for nullable columns).
- **Always test migrations against a copy of production data** — what works on empty tables may lock production for hours.

---

### Q11. 🌐 What is optimistic vs. pessimistic locking in JPA?

**Optimistic locking detects conflicts at commit time using a version column (no DB locks held), while pessimistic locking acquires actual database row locks to prevent concurrent access — use optimistic for high-concurrency web apps, pessimistic for critical financial operations.**

```java
// Optimistic locking — @Version column
@Entity
public class Account {
    @Id
    private Long id;
    
    @Version  // Hibernate adds WHERE version = ? to UPDATE
    private Long version;
    
    private BigDecimal balance;
}

// What happens on concurrent update:
// Thread A: UPDATE account SET balance=900, version=2 WHERE id=1 AND version=1 → succeeds
// Thread B: UPDATE account SET balance=800, version=2 WHERE id=1 AND version=1 → 0 rows updated
// Thread B throws: OptimisticLockException → retry or report conflict

// Pessimistic locking — actual DB lock
@Query("SELECT a FROM Account a WHERE a.id = :id")
@Lock(LockModeType.PESSIMISTIC_WRITE)  // SELECT ... FOR UPDATE
Optional<Account> findByIdForUpdate(@Param("id") Long id);

// With timeout to avoid indefinite blocking
@QueryHints(@QueryHint(name = "jakarta.persistence.lock.timeout", value = "3000"))
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Account> findByIdWithLockTimeout(@Param("id") Long id);
```

| Feature | Optimistic | Pessimistic |
|---------|-----------|-------------|
| Mechanism | Version check at commit | DB row lock at read |
| Contention | Handles well (retry) | Can deadlock |
| Performance | Better for read-heavy | Better for write-heavy critical |
| Use case | Web apps, REST APIs | Financial transactions, inventory |

**⚠️ Pitfalls:**
- Optimistic locking requires retry logic — `OptimisticLockException` must be caught and retried.
- Pessimistic locking order must be consistent — always lock tables/rows in the same order to avoid deadlocks.
- `PESSIMISTIC_READ` (`FOR SHARE`) allows concurrent reads but blocks writes.

---

### Q12. 🏢 How do you handle bulk inserts/updates efficiently in JPA?

**JPA's default entity-by-entity persistence is extremely slow for bulk operations — use batch inserts with `hibernate.jdbc.batch_size`, `@Id` generation strategy compatible with batching (SEQUENCE with allocation), and periodic flush-clear cycles.**

```java
// application.yml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 50
          order_inserts: true     # Group inserts by entity type
          order_updates: true     # Group updates by entity type

// Entity — SEQUENCE strategy (required for batching)
@Entity
public class Event {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "event_seq")
    @SequenceGenerator(name = "event_seq", sequenceName = "event_id_seq", allocationSize = 50)
    private Long id;
    // IDENTITY strategy DISABLES batching — Hibernate must INSERT one-by-one to get generated IDs
}

// Batch insert with flush-clear cycle
@Transactional
public void bulkInsert(List<EventDTO> events) {
    int batchSize = 50;
    for (int i = 0; i < events.size(); i++) {
        entityManager.persist(mapToEntity(events.get(i)));
        if (i > 0 && i % batchSize == 0) {
            entityManager.flush();   // Execute batch INSERT
            entityManager.clear();   // Free memory — detach managed entities
        }
    }
}

// For truly large volumes, bypass JPA entirely
@Autowired
private JdbcTemplate jdbcTemplate;

public void bulkInsertWithJdbc(List<EventDTO> events) {
    jdbcTemplate.batchUpdate(
        "INSERT INTO events (name, timestamp, data) VALUES (?, ?, ?::jsonb)",
        events, 1000, (ps, event) -> {
            ps.setString(1, event.name());
            ps.setTimestamp(2, Timestamp.from(event.timestamp()));
            ps.setString(3, event.dataJson());
        });
}
```

**⚠️ Pitfalls:**
- `GenerationType.IDENTITY` (auto-increment) **disables JDBC batching** in Hibernate — Hibernate must execute each INSERT individually to retrieve the generated ID.
- Without `entityManager.clear()`, the persistence context grows unbounded → `OutOfMemoryError`.
- `saveAll()` in Spring Data JPA does batch correctly IF `batch_size` is set AND the ID strategy supports it.

---

## 🟢 GOOD TO KNOW (Questions 13–20)

---

### Q13. 🏢 What is database partitioning and when should you use it in PostgreSQL?

**Partitioning splits a large table into smaller physical sub-tables (partitions) while presenting a single logical table — enabling faster queries on partition keys, efficient bulk deletion, and parallel query execution across partitions.**

```sql
-- Range partitioning by date (most common)
CREATE TABLE events (
    id BIGSERIAL,
    event_type VARCHAR(50),
    payload JSONB,
    created_at TIMESTAMPTZ NOT NULL,
    PRIMARY KEY (id, created_at)  -- Partition key must be in PK
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2025_q1 PARTITION OF events 
    FOR VALUES FROM ('2025-01-01') TO ('2025-04-01');
CREATE TABLE events_2025_q2 PARTITION OF events 
    FOR VALUES FROM ('2025-04-01') TO ('2025-07-01');

-- Automatic partition creation (pg_partman extension)
-- Or application-level partition creation
```

**⚠️ Pitfall:** Queries without the partition key in the `WHERE` clause scan ALL partitions — always filter on the partition column.

---

### Q14. 🌐 What is the difference between `INNER JOIN`, `LEFT JOIN`, and `CROSS JOIN`?

**`INNER JOIN` returns only matching rows from both tables; `LEFT JOIN` returns all rows from the left table with NULLs for unmatched right-side rows; `CROSS JOIN` produces the Cartesian product of all rows — each serves a distinct querying need.**

```sql
-- INNER JOIN: only customers WITH orders
SELECT c.name, o.id FROM customers c INNER JOIN orders o ON c.id = o.customer_id;

-- LEFT JOIN: ALL customers, even without orders (o.id is NULL for them)
SELECT c.name, o.id FROM customers c LEFT JOIN orders o ON c.id = o.customer_id;

-- CROSS JOIN: every combination (rarely used, expensive)
SELECT p.name, c.name FROM products p CROSS JOIN colors c;  -- 100 products × 10 colors = 1000 rows
```

**⚠️ Pitfall:** Putting a filter on the RIGHT table in the `WHERE` clause converts a `LEFT JOIN` to an `INNER JOIN`. Use the `ON` clause instead: `LEFT JOIN orders o ON c.id = o.customer_id AND o.status = 'ACTIVE'`.

---

### Q15. 🏢 How does PostgreSQL's MVCC (Multi-Version Concurrency Control) work?

**MVCC allows concurrent readers and writers to operate without blocking each other by maintaining multiple versions of each row — each transaction sees a consistent snapshot of data based on its start time, with old versions cleaned up by VACUUM.**

**How It Works:**
1. Each row version has `xmin` (transaction that created it) and `xmax` (transaction that deleted/updated it).
2. `SELECT` sees rows where `xmin < current_txn` AND (`xmax` is empty OR `xmax > current_txn`).
3. `UPDATE` creates a NEW row version with new `xmin`, marks old version with `xmax`.
4. `DELETE` marks the row with `xmax` — doesn't physically remove it.
5. `VACUUM` reclaims space from dead row versions.

**Isolation Levels:**

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|-------------------|-------------|
| Read Committed (default) | ❌ | ✅ possible | ✅ possible |
| Repeatable Read | ❌ | ❌ | ❌ (in PostgreSQL) |
| Serializable | ❌ | ❌ | ❌ |

**⚠️ Pitfall:** Without regular `VACUUM`, dead tuples accumulate → table bloat → degraded performance. Always ensure autovacuum is enabled and tuned.

---

### Q16. 🌐 What is the difference between `HAVING` and `WHERE` in SQL?

**`WHERE` filters rows before aggregation (GROUP BY); `HAVING` filters groups after aggregation — `WHERE` operates on individual row values, `HAVING` operates on aggregate function results.**

```sql
-- WHERE: filter individual rows
-- HAVING: filter aggregated groups
SELECT customer_id, COUNT(*) as order_count, SUM(total) as total_spent
FROM orders
WHERE status = 'COMPLETED'          -- Filter rows BEFORE grouping
GROUP BY customer_id
HAVING SUM(total) > 1000            -- Filter groups AFTER aggregation
ORDER BY total_spent DESC;
```

**⚠️ Pitfall:** Putting filterable conditions in `HAVING` instead of `WHERE` forces the DB to aggregate all rows first — always push filters as far left (into `WHERE`) as possible.

---

### Q17. 🏬 How do you implement soft deletes in JPA?

**Soft deletes mark records as deleted with a flag/timestamp instead of physically removing them — enabling audit trails and data recovery, implemented in Hibernate using `@SQLDelete` and `@Where` (or `@SQLRestriction` in Hibernate 6.3+).**

```java
@Entity
@SQLDelete(sql = "UPDATE orders SET deleted_at = NOW() WHERE id = ?")
@SQLRestriction("deleted_at IS NULL")  // Hibernate 6.3+ (replaces @Where)
public class Order {
    @Id
    private Long id;
    private Instant deletedAt;
}

// orderRepository.deleteById(1L) → executes UPDATE, not DELETE
// orderRepository.findAll() → automatically adds WHERE deleted_at IS NULL
```

**⚠️ Pitfall:** Soft deletes complicate unique constraints — you need partial unique indexes: `CREATE UNIQUE INDEX idx_email_active ON users(email) WHERE deleted_at IS NULL;`

---

### Q18. 🏬 What are window functions in SQL and when do you use them?

**Window functions perform calculations across a set of rows related to the current row — unlike `GROUP BY`, they don't collapse rows, enabling ranking, running totals, and moving averages while preserving individual row data.**

```sql
-- Ranking: top 3 products per category
SELECT name, category, revenue,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) as rank,
    RANK() OVER (PARTITION BY category ORDER BY revenue DESC) as rank_with_ties,
    SUM(revenue) OVER (PARTITION BY category) as category_total,
    revenue::numeric / SUM(revenue) OVER (PARTITION BY category) * 100 as pct_of_category
FROM products;

-- Running total (cumulative sum)
SELECT date, amount,
    SUM(amount) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM transactions;

-- Moving average (last 7 entries)
SELECT date, amount,
    AVG(amount) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as moving_avg_7d
FROM daily_revenue;

-- LAG/LEAD: compare with previous/next row
SELECT date, revenue,
    revenue - LAG(revenue) OVER (ORDER BY date) as day_over_day_change
FROM daily_revenue;
```

**⚠️ Pitfall:** Window functions execute AFTER `WHERE`, `GROUP BY`, and `HAVING` — you can't filter on a window function result in the same query level. Wrap in a subquery or CTE.

---

### Q19. 🏢 How do you implement read replicas with Spring Boot?

**Read replicas distribute read traffic across multiple database copies — Spring Boot supports this via `AbstractRoutingDataSource` that routes `@Transactional(readOnly = true)` queries to replicas and write queries to the primary.**

```java
@Configuration
public class DataSourceConfig {
    @Bean
    public DataSource routingDataSource() {
        var routing = new AbstractRoutingDataSource() {
            @Override
            protected Object determineCurrentLookupKey() {
                return TransactionSynchronizationManager.isCurrentTransactionReadOnly() 
                    ? "replica" : "primary";
            }
        };
        routing.setTargetDataSources(Map.of(
            "primary", primaryDataSource(),
            "replica", replicaDataSource()
        ));
        routing.setDefaultTargetDataSource(primaryDataSource());
        return routing;
    }
}

// Usage — routes automatically
@Service
public class ProductService {
    @Transactional(readOnly = true)  // → routes to replica
    public List<Product> search(String query) { return repo.search(query); }

    @Transactional  // → routes to primary
    public Product create(Product product) { return repo.save(product); }
}
```

**⚠️ Pitfall:** Replication lag means read replicas may return stale data — critical operations (read-after-write) should target the primary.

---

### Q20. 🌐 What is the difference between `EntityManager.persist()` and `EntityManager.merge()`?

**`persist()` makes a new transient entity managed and schedules an INSERT; `merge()` copies the state of a detached entity into a managed copy and schedules an INSERT or UPDATE — `persist()` is for new entities, `merge()` is for reattaching detached ones.**

```java
// persist — entity MUST be new (no ID or transient state)
Product newProduct = new Product("Widget", BigDecimal.TEN);
entityManager.persist(newProduct);  // INSERT; newProduct is now managed
newProduct.setPrice(BigDecimal.ONE); // Dirty checking will UPDATE automatically

// merge — returns a MANAGED copy (original is still detached!)
Product detached = getFromSession(); // Entity is detached
detached.setPrice(BigDecimal.ONE);
Product managed = entityManager.merge(detached); // Returns managed copy
// detached is STILL detached — changes to 'detached' after merge are NOT tracked
// Use 'managed' reference going forward
```

**⚠️ Pitfalls:**
- `persist()` with an existing ID throws `EntityExistsException` or causes duplicate key violation.
- `merge()` on a new entity (no ID) triggers an INSERT — but the returned instance is the managed one, not the original.
- Spring Data JPA's `save()` calls `persist()` if `isNew()` returns true, otherwise `merge()` — `isNew()` checks if the ID is null.
