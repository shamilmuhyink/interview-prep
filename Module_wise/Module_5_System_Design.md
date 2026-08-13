# Module 5: System Design & Architecture

> **Scope:** SOLID, Design Patterns, Scalability, Event-Driven Design, CQRS/Event Sourcing, API Design, CAP Theorem
> **Questions:** 20 | **Critical:** 5 | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

---

### Q1. 🔴 🌐 Explain the SOLID principles with real-world Java/Spring examples. Which one is violated most often?

**SOLID is a set of five object-oriented design principles that guide you toward maintainable, extensible software — the most frequently violated principle is the Single Responsibility Principle (SRP), typically through "God classes" that handle business logic, persistence, and validation all at once.**

| Principle | Definition | Violation Smell |
|-----------|-----------|----------------|
| **S** — Single Responsibility | A class should have one, and only one, reason to change | God classes, 500+ line services |
| **O** — Open/Closed | Open for extension, closed for modification | `if/else` chains on type checking |
| **L** — Liskov Substitution | Subtypes must be substitutable for their base types | Overridden method throws `UnsupportedOperationException` |
| **I** — Interface Segregation | Clients shouldn't depend on interfaces they don't use | Fat interfaces with 20+ methods |
| **D** — Dependency Inversion | Depend on abstractions, not concretions | `new` inside business logic |

```java
// ❌ Violates SRP: This service does too much
@Service
public class OrderService {
    public Order createOrder(OrderRequest req) { /* ... */ }
    public void sendConfirmationEmail(Order order) { /* ... */ }
    public BigDecimal calculateTax(Order order) { /* ... */ }
    public byte[] generateInvoicePdf(Order order) { /* ... */ }
}

// ✅ SRP Applied: Each class has one reason to change
@Service
public class OrderService {
    private final OrderRepository repo;
    private final PricingService pricingService;
    private final OrderEventPublisher eventPublisher;

    @Transactional
    public Order createOrder(OrderRequest req) {
        Order order = Order.from(req);
        order.applyPricing(pricingService.calculate(order));
        Order saved = repo.save(order);
        eventPublisher.publish(new OrderCreatedEvent(saved)); // Notification/PDF handled by listeners
        return saved;
    }
}

// ✅ Open/Closed: Add new payment types without modifying existing code
public interface PaymentProcessor {
    boolean supports(PaymentMethod method);
    PaymentResult process(PaymentRequest request);
}

@Component
public class CreditCardProcessor implements PaymentProcessor {
    public boolean supports(PaymentMethod method) { return method == PaymentMethod.CREDIT_CARD; }
    public PaymentResult process(PaymentRequest request) { /* ... */ }
}

@Component
public class PaymentService {
    private final List<PaymentProcessor> processors; // Spring injects all implementations

    public PaymentResult pay(PaymentRequest request) {
        return processors.stream()
            .filter(p -> p.supports(request.method()))
            .findFirst()
            .orElseThrow(() -> new UnsupportedPaymentException(request.method()))
            .process(request);
    }
    // Adding PayPal? Just create a new @Component. Zero changes to PaymentService.
}

// ✅ Dependency Inversion: Depend on abstraction
@Service
public class NotificationService {
    private final NotificationSender sender; // Interface — not SmtpEmailSender

    public void notify(User user, String message) {
        sender.send(user.getContactInfo(), message);
    }
}
// In tests: inject MockNotificationSender. In prod: SmtpEmailSender or SnsNotificationSender.
```

**⚠️ Pitfall:** Over-applying SOLID creates unnecessary abstraction layers. A simple CRUD service doesn't need 5 interfaces and 3 layers. Apply SOLID where complexity demands it.

---

### Q2. 🔴 🏢 Design a URL shortener like bit.ly. Walk through the system design end-to-end.

**A URL shortener maps a short code (e.g., `sho.rt/abc123`) to a long URL, requiring a high-read, low-write, globally distributed system with sub-10ms redirect latency — the core challenges are unique short code generation, storage efficiency, and analytics at scale.**

**Functional Requirements:**
- Generate a unique short URL for any long URL.
- Redirect short URL to original URL (HTTP 301/302).
- Custom aliases (optional).
- Link expiration (optional).
- Analytics (click count, geography, referrer).

**Non-Functional Requirements:**
- 100:1 read-to-write ratio.
- 100M URLs/month write, 10B redirects/month read.
- Sub-10ms redirect latency.
- 99.99% availability.

**Architecture:**

```
Client → CDN/Edge Cache → Load Balancer → API Gateway
                                            ├── Write Service (create short URL)
                                            │     └── PostgreSQL (primary store)
                                            └── Read Service (redirect)
                                                  ├── Redis Cluster (cache — 90%+ hit rate)
                                                  └── PostgreSQL (cache miss fallback)
                                            
Analytics: Kafka → Flink/Spark → ClickHouse (click analytics)
```

**Short Code Generation Strategies:**

| Strategy | Pros | Cons |
|---------|------|------|
| Base62 encoding of auto-increment ID | Simple, no collisions | Predictable (sequential), requires coordination |
| MD5/SHA256 hash → take 7 chars | Stateless | Collisions possible |
| Pre-generated key pool (KGS) | No collision, fast | Requires separate KGS service |
| Snowflake ID → Base62 | Distributed, no coordination | Longer codes |

```java
// Short code generation using Base62
@Service
public class UrlShortenerService {
    private static final String BASE62 = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
    private static final int CODE_LENGTH = 7; // 62^7 = 3.5 trillion combinations

    private final UrlRepository urlRepository;
    private final RedisTemplate<String, String> redis;

    @Transactional
    public ShortUrl create(String longUrl, Duration ttl) {
        // Check if URL already shortened (dedup)
        return urlRepository.findByLongUrl(longUrl)
            .orElseGet(() -> {
                String shortCode = generateUniqueCode();
                ShortUrl shortUrl = new ShortUrl(shortCode, longUrl, Instant.now().plus(ttl));
                urlRepository.save(shortUrl);
                redis.opsForValue().set("url:" + shortCode, longUrl, ttl);
                return shortUrl;
            });
    }

    public String resolve(String shortCode) {
        // L1: Redis cache
        String longUrl = redis.opsForValue().get("url:" + shortCode);
        if (longUrl != null) return longUrl;

        // L2: Database
        ShortUrl shortUrl = urlRepository.findByShortCode(shortCode)
            .orElseThrow(() -> new UrlNotFoundException(shortCode));

        if (shortUrl.isExpired()) throw new UrlExpiredException(shortCode);

        // Populate cache
        redis.opsForValue().set("url:" + shortCode, shortUrl.getLongUrl(), Duration.ofHours(24));
        return shortUrl.getLongUrl();
    }

    private String generateUniqueCode() {
        // Base62 encoding of Snowflake-like ID
        long id = snowflakeIdGenerator.nextId();
        StringBuilder sb = new StringBuilder();
        while (id > 0) {
            sb.append(BASE62.charAt((int) (id % 62)));
            id /= 62;
        }
        return sb.reverse().toString().substring(0, CODE_LENGTH);
    }
}
```

**Scaling Decisions:**
- **Read replicas** for PostgreSQL — redirect reads don't need strong consistency.
- **Redis Cluster** with 90%+ cache hit rate — most URLs are accessed within hours of creation.
- **CDN caching** for 301 redirects — immutable URLs can be cached at the edge.
- **Rate limiting** on the create endpoint — prevent abuse.

**⚠️ Pitfalls:**
- **301 vs 302 redirect** — 301 is cached by browsers (can't track analytics). Use 302 if you need click analytics.
- **Hash collisions** — if using hash-based generation, implement collision detection and retry.
- **Hot URLs** — a viral URL can overwhelm a single Redis shard. Use consistent hashing with replication.

---

### Q3. 🔴 🌐 What are the key design patterns used in enterprise Java applications? Give real-world examples.

**Enterprise Java relies heavily on Creational (Factory, Builder, Singleton), Structural (Adapter, Decorator, Proxy), and Behavioral (Strategy, Observer, Template Method) patterns — most Spring Boot applications use Strategy, Template Method, and Observer patterns extensively, often without developers realizing it.**

**Most Frequently Asked Patterns:**

```java
// 1. STRATEGY PATTERN — Runtime algorithm selection
// Spring naturally supports this via interface + multiple implementations
public interface ShippingCalculator {
    BigDecimal calculate(Order order);
    ShippingMethod supports();
}

@Component
public class StandardShipping implements ShippingCalculator {
    public BigDecimal calculate(Order order) { return new BigDecimal("5.99"); }
    public ShippingMethod supports() { return ShippingMethod.STANDARD; }
}

@Component
public class ExpressShipping implements ShippingCalculator {
    public BigDecimal calculate(Order order) { 
        return order.getWeight().multiply(new BigDecimal("2.50")); 
    }
    public ShippingMethod supports() { return ShippingMethod.EXPRESS; }
}

@Service
public class ShippingService {
    private final Map<ShippingMethod, ShippingCalculator> calculators;

    public ShippingService(List<ShippingCalculator> calculators) {
        this.calculators = calculators.stream()
            .collect(Collectors.toMap(ShippingCalculator::supports, Function.identity()));
    }

    public BigDecimal getShippingCost(Order order) {
        return calculators.get(order.getShippingMethod()).calculate(order);
    }
}

// 2. BUILDER PATTERN — Complex object construction
public record Notification(String to, String subject, String body, 
                           Priority priority, List<String> cc, Map<String, String> metadata) {
    public static Builder builder(String to) { return new Builder(to); }
    
    public static class Builder {
        private final String to;
        private String subject = "";
        private String body = "";
        private Priority priority = Priority.NORMAL;
        private List<String> cc = List.of();
        private Map<String, String> metadata = Map.of();

        private Builder(String to) { this.to = Objects.requireNonNull(to); }
        public Builder subject(String s) { this.subject = s; return this; }
        public Builder body(String b) { this.body = b; return this; }
        public Builder priority(Priority p) { this.priority = p; return this; }
        public Builder cc(List<String> cc) { this.cc = List.copyOf(cc); return this; }
        public Builder metadata(Map<String, String> m) { this.metadata = Map.copyOf(m); return this; }
        public Notification build() { 
            return new Notification(to, subject, body, priority, cc, metadata); 
        }
    }
}

// Usage
Notification.builder("user@example.com")
    .subject("Order Shipped")
    .body("Your order #123 has been shipped")
    .priority(Priority.HIGH)
    .build();

// 3. OBSERVER PATTERN — Spring's Event System
public record OrderCreatedEvent(Order order, Instant timestamp) {}

@Service
public class OrderService {
    private final ApplicationEventPublisher publisher;

    @Transactional
    public Order create(OrderRequest request) {
        Order order = orderRepo.save(new Order(request));
        publisher.publishEvent(new OrderCreatedEvent(order, Instant.now()));
        return order;
    }
}

// Multiple independent listeners — decoupled
@Component
public class InventoryListener {
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        inventoryService.reserve(event.order().getItems());
    }
}

@Component
public class NotificationListener {
    @Async
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        emailService.sendConfirmation(event.order());
    }
}

// 4. DECORATOR PATTERN — Layer functionality
public interface DataFetcher {
    Data fetch(String key);
}

@Component @Primary
public class CachingDataFetcher implements DataFetcher {
    private final DataFetcher delegate;
    private final Cache cache;

    public CachingDataFetcher(@Qualifier("databaseFetcher") DataFetcher delegate, Cache cache) {
        this.delegate = delegate;
        this.cache = cache;
    }

    public Data fetch(String key) {
        return cache.get(key, () -> delegate.fetch(key)); // Cache miss → delegate
    }
}
```

**⚠️ Pitfall:** Don't force patterns. If you're creating a `SingletonFactoryStrategyBridgeAdapter`, you've gone too far. Patterns should simplify, not complicate.

---

### Q4. 🔴 🏢 What is CQRS and Event Sourcing? When should you use them?

**CQRS (Command Query Responsibility Segregation) separates the write model (commands) from the read model (queries), allowing each to be optimized independently; Event Sourcing persists state as a sequence of immutable events rather than current state — together, they enable complex domain modeling, audit trails, and temporal queries at the cost of eventual consistency and increased complexity.**

```
CQRS Architecture:
                    ┌─── Command Side ──────┐
                    │  API → Command Handler │
Client ─── API ────►│  → Domain Model        │──► Event Store
 Gateway            │  → Event Publisher      │     (PostgreSQL / EventStoreDB)
                    └────────────────────────┘
                              │ events
                              ▼
                    ┌─── Projection ─────────┐
                    │  Event Consumer         │
                    │  → Read Model Updater   │──► Read DB (PostgreSQL / Elasticsearch)
                    └────────────────────────┘
                              │
Client ─── API ──── Query Side ──── Read DB ──► Response
```

```java
// Event Sourcing: Store events, not state
public sealed interface OrderEvent {
    UUID orderId();
    Instant occurredAt();
}

public record OrderCreated(UUID orderId, UUID customerId, List<LineItem> items, 
                           Instant occurredAt) implements OrderEvent {}
public record OrderItemAdded(UUID orderId, LineItem item, Instant occurredAt) implements OrderEvent {}
public record OrderShipped(UUID orderId, String trackingNumber, Instant occurredAt) implements OrderEvent {}
public record OrderCancelled(UUID orderId, String reason, Instant occurredAt) implements OrderEvent {}

// Aggregate — rebuilds state from events
public class OrderAggregate {
    private UUID id;
    private OrderStatus status;
    private List<LineItem> items = new ArrayList<>();
    private final List<OrderEvent> uncommittedEvents = new ArrayList<>();

    // Rebuild state from history
    public static OrderAggregate fromHistory(List<OrderEvent> history) {
        OrderAggregate aggregate = new OrderAggregate();
        history.forEach(aggregate::apply);
        return aggregate;
    }

    // Command → validate → emit event
    public void addItem(LineItem item) {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot add items to " + status + " order");
        }
        emit(new OrderItemAdded(id, item, Instant.now()));
    }

    private void apply(OrderEvent event) {
        switch (event) {
            case OrderCreated e -> { this.id = e.orderId(); this.status = OrderStatus.DRAFT; this.items.addAll(e.items()); }
            case OrderItemAdded e -> this.items.add(e.item());
            case OrderShipped e -> this.status = OrderStatus.SHIPPED;
            case OrderCancelled e -> this.status = OrderStatus.CANCELLED;
        }
    }

    private void emit(OrderEvent event) {
        apply(event);                          // Apply to current state
        uncommittedEvents.add(event);          // Store for persistence
    }
}

// CQRS: Separate read model (denormalized for queries)
@Component
public class OrderReadModelProjector {
    @EventListener
    public void on(OrderCreated event) {
        OrderView view = new OrderView(event.orderId(), event.customerId(),
            event.items().size(), calculateTotal(event.items()), "DRAFT");
        orderViewRepository.save(view);  // Write to read-optimized table
    }

    @EventListener
    public void on(OrderShipped event) {
        orderViewRepository.updateStatus(event.orderId(), "SHIPPED", event.trackingNumber());
    }
}
```

**When to Use CQRS/Event Sourcing:**
- ✅ Complex domains with rich business rules (DDD aggregates).
- ✅ Audit requirements (complete history of every change).
- ✅ Temporal queries ("What was the order state at 3pm yesterday?").
- ✅ Different read/write scaling needs (read replicas, Elasticsearch for queries).
- ❌ Simple CRUD applications — massive overkill.
- ❌ Teams without event-driven experience — steep learning curve.

**⚠️ Pitfalls:**
- **Eventual consistency** — the read model lags behind the write model. UIs must handle this.
- **Event schema evolution** — you can never delete old events. Use upcasters for schema migration.
- **Snapshot optimization** — replaying thousands of events is slow. Snapshot aggregate state periodically.

---

### Q5. 🔴 🏢 How would you design a system for high scalability? Walk through horizontal scaling, load balancing, and caching layers.

**A highly scalable system is built on stateless services (horizontally scaled behind load balancers), multi-tier caching (CDN → Redis → application), asynchronous processing (message queues for decoupling), and data partitioning (sharding for distributed databases).**

**Scaling Architecture:**

```
               ┌─── CDN (CloudFront) ────────────── Static assets, API caching
               │
Users ──►  Global LB (Route 53) 
               │
          ┌────┴──── Regional LB (ALB) ──────┐
          │    ┌──────┐ ┌──────┐ ┌──────┐    │
          │    │ App 1│ │ App 2│ │ App N│    │  ← Stateless, auto-scaled (K8s HPA)
          │    └──┬───┘ └──┬───┘ └──┬───┘    │
          └───────┼────────┼────────┼────────┘
                  │        │        │
          ┌───────▼────────▼────────▼────────┐
          │          Redis Cluster             │  ← L1 Cache (session, hot data)
          └───────────────┬──────────────────┘
                          │ cache miss
          ┌───────────────▼──────────────────┐
          │     PostgreSQL (Primary)          │  ← Write path
          │          ↓ replication            │
          │  Read Replica 1 │ Read Replica 2  │  ← Read path
          └──────────────────────────────────┘
          
          ┌──────────────────────────────────┐
          │   Kafka Cluster                   │  ← Async processing
          │   → Order events                  │
          │   → Notification events           │
          │   → Analytics events              │
          └──────────────────────────────────┘
```

**Key Scaling Principles:**

| Principle | Implementation |
|-----------|---------------|
| **Stateless services** | No server-side sessions. Store state in Redis/DB. |
| **Horizontal scaling** | Add more instances, not bigger instances. |
| **Caching layers** | CDN → Redis → Application cache → DB |
| **Async processing** | Queue non-critical work (emails, analytics) |
| **Database scaling** | Read replicas, sharding, connection pooling |
| **Auto-scaling** | K8s HPA based on CPU/custom metrics |

```java
// Stateless service — session in Redis, not in-memory
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 3600)
public class SessionConfig {
    @Bean
    public RedisConnectionFactory connectionFactory() {
        return new LettuceConnectionFactory("redis-cluster", 6379);
    }
}

// Rate limiting at API gateway level
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("api", r -> r.path("/api/**")
            .filters(f -> f.requestRateLimiter(c -> c
                .setRateLimiter(redisRateLimiter())
                .setKeyResolver(userKeyResolver())))
            .uri("lb://backend-service"))
        .build();
}
```

**⚠️ Pitfalls:**
- **Premature scaling** — don't distribute until you MUST. A single well-optimized PostgreSQL handles millions of rows.
- **Shared mutable state** — the #1 enemy of scalability. Avoid global locks, shared file systems.
- **Connection limits** — each app instance opens DB connections. 50 instances × 20 connections = 1000 connections. Use PgBouncer.

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

---

### Q6. 🌐 What is the CAP theorem and how does it apply to microservices?

**The CAP theorem states that a distributed system can guarantee at most two of three properties: Consistency (every read returns the latest write), Availability (every request gets a response), and Partition Tolerance (system works despite network splits) — since network partitions are inevitable, the real choice is CP (consistency over availability) vs. AP (availability over consistency).**

| System | Type | Behavior During Partition |
|--------|------|--------------------------|
| PostgreSQL (single-node) | CA | Not distributed — N/A |
| Cassandra, DynamoDB | AP | Returns stale data, resolves later |
| MongoDB (default), etcd, ZooKeeper | CP | Rejects writes until partition heals |

**In Microservices Practice:**
- **Payment service** → CP (consistency critical — no double charges)
- **Product catalog** → AP (stale price for 5 seconds is acceptable)
- **User activity feed** → AP (eventual consistency is fine)

**⚠️ Pitfall:** CAP is a spectrum, not a binary toggle. Real systems choose different guarantees for different operations within the same service.

---

### Q7. 🏢 What is Domain-Driven Design (DDD) and how do you apply it in microservice boundaries?

**DDD is a software design approach that models complex business domains by aligning code structure with business concepts — Bounded Contexts define microservice boundaries, Aggregates enforce consistency, and Ubiquitous Language ensures developers and domain experts speak the same terms.**

```java
// Aggregate Root — consistency boundary
@Entity
public class Order {  // Aggregate Root
    @Id private UUID id;
    private OrderStatus status;
    
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderLine> lines = new ArrayList<>();  // Owned by the aggregate

    // Business logic INSIDE the aggregate — not in a service
    public void addLine(Product product, int quantity) {
        if (status != OrderStatus.DRAFT) {
            throw new OrderNotModifiableException(id, status);
        }
        lines.add(new OrderLine(product.getId(), product.getPrice(), quantity));
    }

    public void submit() {
        if (lines.isEmpty()) throw new EmptyOrderException(id);
        this.status = OrderStatus.SUBMITTED;
    }
}

// Repository operates on aggregate root — never on OrderLine directly
public interface OrderRepository extends JpaRepository<Order, UUID> {}
```

**Bounded Context → Microservice Mapping:**
```
E-commerce Domain:
├── Order Context     → order-service     (Order, OrderLine, OrderStatus)
├── Inventory Context → inventory-service (Stock, Warehouse, Reservation)
├── Payment Context   → payment-service   (Payment, Refund, Transaction)
└── Shipping Context  → shipping-service  (Shipment, Carrier, TrackingEvent)

Each context has its OWN model of shared concepts:
- "Product" in Order Context = { id, name, price }
- "Product" in Inventory Context = { id, sku, warehouseLocations, stock }
```

**⚠️ Pitfall:** Don't make microservice boundaries too granular — a `LineItemService` separate from `OrderService` creates distributed monolith pain. Aggregate roots guide the right boundaries.

---

### Q8. 🌐 What is the difference between monolithic, microservices, and modular monolith architectures?

**A monolith deploys everything as one unit; microservices decompose into independently deployable services; a modular monolith structures a single deployable into well-encapsulated modules with clear boundaries — the modular monolith is often the best starting point before migrating to microservices.**

| Aspect | Monolith | Modular Monolith | Microservices |
|--------|----------|-------------------|---------------|
| Deployment | Single unit | Single unit | Independent per service |
| Data | Shared DB | Separate schemas | Separate databases |
| Communication | Method calls | Module API (interfaces) | Network (HTTP, messaging) |
| Complexity | Low | Medium | High |
| Team scaling | Hard beyond 10 devs | Good up to 30-40 | Scales with org |
| Debugging | Easy | Easy | Distributed tracing needed |

```java
// Modular Monolith with Spring Modulith
@ApplicationModule(allowedDependencies = {"order :: api", "shared"})
package com.app.inventory;  // Module can only depend on order's public API

// Module API — only this is visible to other modules
package com.app.order.api;
public interface OrderService {
    OrderDTO createOrder(CreateOrderRequest request);
}

// Module internal — hidden from other modules
package com.app.order.internal;
@Service
class OrderServiceImpl implements OrderService { /* ... */ }

// Spring Modulith verifies boundaries at test time
@Test
void verifyModuleBoundaries() {
    ApplicationModules.of(Application.class).verify();
}
```

**⚠️ Pitfall:** Teams often jump to microservices prematurely. Start with a modular monolith, extract services only when you have a concrete scaling, deployment, or team autonomy need.

---

### Q9. 🏢 How do you design an event-driven architecture? What are the key patterns?

**Event-driven architecture decouples producers and consumers through asynchronous events — with three key patterns: Event Notification (simple signal), Event-Carried State Transfer (event contains full data, enabling local caches), and Event Sourcing (events as the system of record).**

| Pattern | Event Content | Consumer Behavior | Coupling |
|---------|--------------|-------------------|---------|
| Event Notification | Minimal (just ID) | Queries source for details | Medium |
| Event-Carried State Transfer | Full entity state | Uses event data directly | Low |
| Event Sourcing | State change delta | Rebuilds state from events | Very Low |

```java
// Event-Carried State Transfer — consumer has all data it needs
public record ProductPriceChangedEvent(
    UUID productId,
    String productName,
    BigDecimal oldPrice,
    BigDecimal newPrice,
    String currency,
    Instant changedAt
) {}

// Consumer in Order Service — updates local price cache without calling Product Service
@KafkaListener(topics = "product-events")
public void onProductPriceChanged(ProductPriceChangedEvent event) {
    localProductCache.updatePrice(event.productId(), event.newPrice());
}
```

**⚠️ Pitfall:** Event ordering is only guaranteed within a partition (Kafka). Use the entity ID as the partition key to ensure events for the same entity are processed in order.

---

### Q10. 🏢 What is the Strangler Fig pattern for legacy migration?

**The Strangler Fig pattern incrementally replaces a legacy monolith by routing specific features to new microservices while the old system continues serving unchanged features — over time, the new system "strangles" the old one until it can be decommissioned.**

```
Phase 1:  Client → Proxy → Legacy Monolith (100% traffic)
Phase 2:  Client → Proxy → Legacy (80%) + New Service A (20% — /orders)
Phase 3:  Client → Proxy → Legacy (40%) + Service A + Service B + Service C
Phase 4:  Client → Proxy → New Services (100%) → Decommission legacy
```

```java
// API Gateway routing — gradually shift traffic
spring:
  cloud:
    gateway:
      routes:
        - id: new-order-service
          uri: http://new-order-service:8080
          predicates:
            - Path=/api/orders/**
            - Header=X-Feature-Flag, new-orders  # Feature flag controlled
        - id: legacy-fallback
          uri: http://legacy-monolith:8080
          predicates:
            - Path=/api/**  # Everything else stays on legacy
```

**⚠️ Pitfall:** Don't try to rewrite everything at once. Migrate one bounded context at a time, run both in parallel, and compare results before cutting over.

---

### Q11. 🏬 What is the API Gateway pattern and what problems does it solve?

**An API Gateway is a single entry point that acts as a reverse proxy for all client-to-microservice communication — it handles cross-cutting concerns (auth, rate limiting, routing, SSL termination) and provides protocol translation, request aggregation, and client-specific API facades.**

**Problems Solved:**
- **Client complexity** — one endpoint instead of 10 microservice URLs.
- **Cross-cutting concerns** — auth, logging, rate limiting in one place.
- **Protocol translation** — REST → gRPC, WebSocket → Kafka.
- **Backend for Frontend (BFF)** — different API shapes for web vs. mobile.

**⚠️ Pitfall:** The gateway can become a bottleneck. Keep it thin (routing + auth only). Don't put business logic in the gateway.

---

### Q12. 🌐 What is the difference between REST and gRPC? When do you choose one over the other?

**REST uses HTTP/1.1 with JSON for human-readable, resource-oriented APIs; gRPC uses HTTP/2 with Protocol Buffers for high-performance, strongly-typed service-to-service communication — use REST for public/external APIs and gRPC for internal microservice communication.**

| Feature | REST | gRPC |
|---------|------|------|
| Protocol | HTTP/1.1 (mostly) | HTTP/2 |
| Payload | JSON (text) | Protobuf (binary) |
| Contract | OpenAPI/Swagger (optional) | `.proto` file (mandatory) |
| Streaming | SSE, WebSocket | Built-in bidirectional streaming |
| Browser support | Native | Requires gRPC-Web proxy |
| Performance | Slower (JSON parsing) | 5-10x faster |

**⚠️ Pitfall:** gRPC requires proto schema management and code generation — adds build complexity. Only worth it for internal, high-throughput service calls.

---

## 🟢 GOOD TO KNOW (Questions 13–20)

---

### Q13. 🏬 What is the Circuit Breaker vs. Bulkhead vs. Rate Limiter pattern?

**Circuit Breaker stops calling a failing service; Bulkhead isolates failures to prevent resource exhaustion; Rate Limiter controls the rate of incoming requests — they're complementary resilience patterns used together to build fault-tolerant systems.**

| Pattern | Protects Against | Mechanism |
|---------|-----------------|-----------|
| Circuit Breaker | Cascading failures from downstream | Fail-fast when error rate exceeds threshold |
| Bulkhead | Thread/resource exhaustion | Limit concurrent calls per downstream service |
| Rate Limiter | Overload from upstream | Limit requests per time window |

---

### Q14. 🏬 What is the Saga pattern vs. Two-Phase Commit (2PC)?

**Sagas use local transactions with compensating actions for rollback (eventually consistent); 2PC coordinates atomic commits across services via a coordinator (strongly consistent) — Sagas are preferred in microservices because 2PC blocks resources and creates tight coupling.**

| Feature | Saga | 2PC |
|---------|------|-----|
| Consistency | Eventual | Strong (ACID) |
| Availability | High | Low (coordinator is SPOF) |
| Lock duration | Short (local txn) | Long (global lock) |
| Scalability | High | Low |
| Complexity | Compensations | Coordinator protocol |

---

### Q15. 🏢 How do you design an idempotent API?

**An idempotent API produces the same result regardless of how many times a request is repeated — achieved through idempotency keys, conditional operations, and client-generated IDs that prevent duplicate processing.**

```java
@PostMapping("/payments")
public ResponseEntity<Payment> createPayment(
        @RequestHeader("Idempotency-Key") String idempotencyKey,
        @RequestBody PaymentRequest request) {
    // Check if already processed
    Optional<Payment> existing = paymentRepo.findByIdempotencyKey(idempotencyKey);
    if (existing.isPresent()) return ResponseEntity.ok(existing.get());

    // Process and store with idempotency key
    Payment payment = paymentService.process(request, idempotencyKey);
    return ResponseEntity.status(HttpStatus.CREATED).body(payment);
}
```

**HTTP Method Idempotency:**
- `GET`, `PUT`, `DELETE` — idempotent by design.
- `POST` — NOT idempotent. Requires explicit handling.
- `PATCH` — depends on implementation.

---

### Q16. 🏢 What is the Outbox Pattern and why is it important?

**The Outbox Pattern ensures reliable event publishing by writing events to an outbox table in the same database transaction as the business operation — a separate process reads the outbox and publishes to the message broker, guaranteeing at-least-once delivery without distributed transactions.**

```java
@Transactional
public Order createOrder(OrderRequest request) {
    Order order = orderRepo.save(new Order(request));
    // Same transaction — atomic with order creation
    outboxRepo.save(new OutboxEvent("order-events", order.getId().toString(),
        "OrderCreated", objectMapper.writeValueAsString(new OrderCreatedEvent(order))));
    return order;
}

// Separate scheduled task reads outbox and publishes
@Scheduled(fixedRate = 1000)
public void publishOutboxEvents() {
    List<OutboxEvent> pending = outboxRepo.findUnpublished(100);
    pending.forEach(event -> {
        kafkaTemplate.send(event.getTopic(), event.getKey(), event.getPayload());
        event.markPublished();
        outboxRepo.save(event);
    });
}
```

**⚠️ Pitfall:** Use Debezium CDC (Change Data Capture) for production-grade outbox processing instead of polling — it's more efficient and has lower latency.

---

### Q17. 🏢 How do you handle distributed logging and observability?

**Distributed observability rests on three pillars: structured logging (correlated with trace IDs), distributed tracing (request flow across services), and metrics (RED/USE methods) — all feeding into centralized platforms (ELK, Grafana, Jaeger).**

```
Three Pillars:
├── Logs    → ELK Stack (Elasticsearch, Logstash, Kibana) or Loki
├── Traces  → Jaeger / Tempo (OpenTelemetry)  
└── Metrics → Prometheus + Grafana

Correlation: traceId links logs ↔ traces ↔ metrics
```

---

### Q18. 🏢 What is the Backend for Frontend (BFF) pattern?

**BFF creates dedicated API layers for each client type (web, mobile, IoT) — each BFF aggregates, transforms, and optimizes responses for its specific client's needs, avoiding a one-size-fits-all API.**

```
Mobile App  → Mobile BFF  → Microservices
Web App     → Web BFF     → Microservices
Admin       → Admin BFF   → Microservices
```

**⚠️ Pitfall:** Don't create a BFF per team — that's just a proxy. A BFF should aggregate and transform data for client-specific needs.

---

### Q19. 🏬 What is the difference between vertical and horizontal scaling?

**Vertical scaling (scaling up) adds more CPU/RAM/disk to existing servers; horizontal scaling (scaling out) adds more server instances behind a load balancer — horizontal is preferred for web applications because it offers linear scaling and eliminates single points of failure.**

| Aspect | Vertical | Horizontal |
|--------|----------|-----------|
| Implementation | Bigger server | More servers |
| Limit | Hardware ceiling | Theoretically unlimited |
| Downtime | Usually required | Zero-downtime (rolling) |
| Cost | Exponential (premium hardware) | Linear |
| State | Can keep server state | Must be stateless |

---

### Q20. 🏢 How do you design for data consistency in a distributed system?

**Distributed data consistency is achieved through strong consistency (synchronous replication, consensus protocols) for critical operations and eventual consistency (async replication, conflict resolution) for everything else — the key is choosing the right consistency level per use case, not globally.**

**Consistency Patterns:**
- **Strong Consistency** — Raft/Paxos consensus, synchronous replication. Use for: payments, account balances.
- **Eventual Consistency** — Async replication, CRDTs, last-writer-wins. Use for: social feeds, product views.
- **Causal Consistency** — Preserves cause-and-effect ordering. Use for: chat messages, collaborative editing.

```java
// Pattern: Read-your-own-writes consistency
@Service
public class UserProfileService {
    public UserProfile updateProfile(Long userId, ProfileUpdate update) {
        UserProfile saved = primaryRepo.save(update.apply(userId));  // Write to primary
        cache.put("user:" + userId, saved);                          // Immediately update cache
        return saved;                                                 // User sees their own write
    }

    public UserProfile getProfile(Long userId, boolean isOwner) {
        if (isOwner) return primaryRepo.findById(userId);    // Read from primary (strong)
        return cache.getOrFetch("user:" + userId,
            () -> replicaRepo.findById(userId));              // Read from replica (eventual)
    }
}
```

**⚠️ Pitfall:** "Eventual" has no time bound. In practice, define SLAs: "read replicas are consistent within 500ms." Monitor replication lag.
