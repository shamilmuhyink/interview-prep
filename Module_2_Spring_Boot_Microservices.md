# Module 2: Spring Boot & Microservices

> **Scope:** IoC/DI, AOP, Security, Service Discovery, Distributed Tracing, Resilience, REST API Design
> **Questions:** 20 | **Critical:** 5 | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

---

### Q1. 🔴 🌐 Explain the Spring IoC container and Dependency Injection. What are the different scopes and injection types?

**The Spring IoC container manages the full lifecycle of application objects (beans) by inverting the control of dependency creation — instead of objects creating their dependencies, the container injects them, enabling loose coupling, testability, and modular design.**

**Injection Types (Ordered by Preference):**

| Type | Mechanism | Recommended? |
|------|-----------|:---:|
| **Constructor** | Dependencies passed via constructor args | ✅ Preferred |
| **Setter** | Injected via setter methods | Optional dependencies only |
| **Field** | `@Autowired` directly on field | ❌ Avoid (untestable, hides dependencies) |

**Bean Scopes:**

| Scope | Lifecycle | Use Case |
|-------|-----------|----------|
| `singleton` (default) | One instance per ApplicationContext | Stateless services |
| `prototype` | New instance per injection point | Stateful/non-shared beans |
| `request` | One per HTTP request | Request-scoped data |
| `session` | One per HTTP session | User session data |
| `application` | One per ServletContext | App-wide config |

```java
// ✅ Constructor injection — immutable, testable, fail-fast
@Service
public class OrderService {
    private final OrderRepository orderRepo;
    private final PaymentGateway paymentGateway;
    private final EventPublisher eventPublisher;

    // Single constructor — @Autowired is optional since Spring 4.3
    public OrderService(OrderRepository orderRepo, 
                        PaymentGateway paymentGateway,
                        EventPublisher eventPublisher) {
        this.orderRepo = orderRepo;
        this.paymentGateway = paymentGateway;
        this.eventPublisher = eventPublisher;
    }

    // Easy to unit test:
    // new OrderService(mockRepo, mockGateway, mockPublisher)
}

// Prototype bean injected into singleton (common trap)
@Service
public class ReportService {
    private final ObjectProvider<ReportGenerator> reportGeneratorProvider;

    public ReportService(ObjectProvider<ReportGenerator> reportGeneratorProvider) {
        this.reportGeneratorProvider = reportGeneratorProvider;
    }

    public Report generate() {
        // ObjectProvider creates a new prototype instance each time
        ReportGenerator generator = reportGeneratorProvider.getObject();
        return generator.build();
    }
}
```

**⚠️ Pitfalls:**
- Injecting a `prototype` bean into a `singleton` bean directly gives you the SAME instance every time. Use `ObjectProvider`, `@Lookup`, or `Provider<T>`.
- Circular dependencies on constructor injection cause `BeanCurrentlyInCreationException`. Redesign the dependency graph.
- Field injection makes beans untestable without Spring context or reflection hacks.

---

### Q2. 🔴 🌐 How does Spring Boot auto-configuration work? Explain `@SpringBootApplication`, `spring.factories`, and conditional annotations.

**Spring Boot auto-configuration automatically configures beans based on classpath contents, existing beans, and property values — using `@Conditional` annotations to make smart, non-invasive decisions, eliminating 90%+ of XML/Java configuration.**

**`@SpringBootApplication` is a composed annotation:**
```java
@SpringBootConfiguration   // marks this as a configuration source
@EnableAutoConfiguration   // triggers auto-configuration
@ComponentScan             // scans current package + sub-packages
public class Application { }
```

**Auto-Configuration Flow:**
1. Spring Boot reads `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Boot 3.x) or `spring.factories` (Boot 2.x).
2. Each listed class is a `@Configuration` guarded by `@Conditional` annotations.
3. Conditions are evaluated — only matching configs are applied.

**Key Conditional Annotations:**

| Annotation | Triggers When |
|-----------|---------------|
| `@ConditionalOnClass` | Class exists on classpath |
| `@ConditionalOnMissingBean` | No bean of that type exists (user hasn't defined one) |
| `@ConditionalOnProperty` | Property is set to a specific value |
| `@ConditionalOnWebApplication` | Running as a web app |

```java
// Custom auto-configuration for a notification service
@AutoConfiguration
@ConditionalOnClass(SlackClient.class)           // Only if Slack SDK is on classpath
@ConditionalOnProperty(prefix = "notifications.slack", name = "enabled", havingValue = "true")
@EnableConfigurationProperties(SlackProperties.class)
public class SlackAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean                     // User can override with their own bean
    public NotificationService slackNotificationService(SlackProperties props) {
        return new SlackNotificationService(props.getWebhookUrl(), props.getChannel());
    }
}

// Register in: META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
// com.mylib.SlackAutoConfiguration
```

**⚠️ Pitfalls:**
- Auto-configuration classes run AFTER user-defined `@Configuration` — use `@ConditionalOnMissingBean` to let users override.
- `@ComponentScan` only scans the package of the main class and below — beans in sibling packages are invisible.
- Use `--debug` flag or `spring.boot.autoconfigure.exclude` to troubleshoot unwanted auto-configs.

---

### Q3. 🔴 🏢 How do you implement inter-service communication in microservices? Compare synchronous vs. asynchronous patterns.

**Inter-service communication should default to asynchronous messaging (Kafka, RabbitMQ) for decoupled, resilient systems, with synchronous REST/gRPC reserved for real-time query needs — the choice depends on coupling tolerance, latency requirements, and failure semantics.**

**Communication Patterns:**

| Pattern | Protocol | Coupling | Latency | Failure Handling |
|---------|----------|---------|---------|------------------|
| REST (sync) | HTTP/JSON | High | Low | Circuit breaker, retries |
| gRPC (sync) | HTTP/2 + Protobuf | High | Very low | Built-in deadlines |
| Messaging (async) | Kafka/RabbitMQ | Low | Higher | Dead-letter queues, retries |
| Event-driven (async) | Kafka/SNS | Very low | Eventual | Event sourcing, idempotency |

```java
// 1. Synchronous: Spring WebClient with resilience (Spring Boot 3.x)
@Service
public class InventoryClient {
    private final WebClient webClient;

    public InventoryClient(WebClient.Builder builder) {
        this.webClient = builder.baseUrl("http://inventory-service").build();
    }

    @CircuitBreaker(name = "inventory", fallbackMethod = "fallbackStock")
    @Retry(name = "inventory")
    public Mono<StockLevel> getStock(String sku) {
        return webClient.get()
            .uri("/api/stock/{sku}", sku)
            .retrieve()
            .bodyToMono(StockLevel.class)
            .timeout(Duration.ofSeconds(3));
    }

    private Mono<StockLevel> fallbackStock(String sku, Throwable t) {
        return Mono.just(StockLevel.unknown(sku)); // Graceful degradation
    }
}

// 2. Asynchronous: Kafka event-driven
@Service
public class OrderEventPublisher {
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    public void publishOrderCreated(Order order) {
        var event = new OrderCreatedEvent(order.getId(), order.getItems(), Instant.now());
        kafkaTemplate.send("order-events", order.getId().toString(), event)
            .whenComplete((result, ex) -> {
                if (ex != null) log.error("Failed to publish event for order {}", order.getId(), ex);
            });
    }
}

@Service
public class InventoryEventConsumer {
    @KafkaListener(topics = "order-events", groupId = "inventory-service")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Idempotent processing — check if already processed
        if (eventStore.isProcessed(event.eventId())) return;
        inventoryService.reserveStock(event.items());
        eventStore.markProcessed(event.eventId());
    }
}
```

**When to Choose What:**
- **Sync** — user-facing APIs needing immediate response (auth, real-time queries).
- **Async** — cross-domain workflows (order → inventory → shipping → notification).
- **Hybrid** — synchronous gateway → async internal communication (CQRS pattern).

**⚠️ Pitfalls:**
- Synchronous chains create **distributed monoliths** — Service A → B → C means C's failure cascades to A.
- **Idempotency is mandatory** for async — messages can be delivered more than once.
- **Distributed transactions** are hard — prefer the Saga pattern over 2PC.

---

### Q4. 🔴 🌐 How does Spring Security work in Spring Boot 3? Explain the Security Filter Chain, JWT authentication, and method security.

**Spring Security 3.x uses a `SecurityFilterChain` bean-based configuration (replacing `WebSecurityConfigurerAdapter`) where HTTP requests pass through an ordered chain of filters that handle authentication, authorization, CSRF, and session management.**

**Filter Chain Flow:**
```
HTTP Request → DisableEncodeUrlFilter → SecurityContextHolderFilter → CsrfFilter 
→ AuthenticationFilter (JWT/Basic/Form) → AuthorizationFilter → DispatcherServlet
```

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // Enables @PreAuthorize, @Secured
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())  // Disable for stateless APIs
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**", "/actuator/health").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.DELETE).hasAuthority("SCOPE_delete")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint())
                .accessDeniedHandler(new BearerTokenAccessDeniedHandler())
            )
            .build();
    }

    // JWT decoder with public key validation
    @Bean
    public JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withPublicKey(rsaPublicKey).build();
    }
}

// Custom JWT token generation
@Service
public class TokenService {
    private final JwtEncoder jwtEncoder;

    public String generateToken(Authentication auth) {
        Instant now = Instant.now();
        String scope = auth.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.joining(" "));

        JwtClaimsSet claims = JwtClaimsSet.builder()
            .issuer("self")
            .issuedAt(now)
            .expiresAt(now.plus(1, ChronoUnit.HOURS))
            .subject(auth.getName())
            .claim("scope", scope)
            .build();

        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }
}

// Method-level security
@Service
public class DocumentService {
    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public Document getDocument(Long userId, Long docId) {
        return documentRepository.findById(docId).orElseThrow();
    }
}
```

**⚠️ Pitfalls:**
- **Filter order matters** — custom filters must be placed at the correct position: `.addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)`.
- **CSRF must NOT be disabled for browser-based apps** — only disable for stateless REST APIs.
- **`@PreAuthorize` requires `@EnableMethodSecurity`** — without it, annotations are silently ignored.
- **Don't store secrets in JWTs** — they're Base64-encoded, not encrypted.

---

### Q5. 🔴 🏢 What is the Circuit Breaker pattern? How do you implement resilience in microservices with Resilience4j?

**The Circuit Breaker pattern prevents cascading failures by short-circuiting calls to a failing downstream service — transitioning through CLOSED (normal), OPEN (rejecting calls), and HALF_OPEN (testing recovery) states based on failure rate thresholds.**

**State Machine:**
```
CLOSED ─(failure rate > threshold)──► OPEN ──(wait duration elapsed)──► HALF_OPEN
  ▲                                                                        │
  └─────────────────────(success rate > threshold)─────────────────────────┘
  OPEN ◄───────────────(failure rate still high)───────────────────────────┘
```

```java
// application.yml
resilience4j:
  circuitbreaker:
    instances:
      payment-service:
        sliding-window-size: 10
        failure-rate-threshold: 50          # Open if 50%+ calls fail
        wait-duration-in-open-state: 10s    # Wait before trying HALF_OPEN
        permitted-number-of-calls-in-half-open-state: 3
        slow-call-duration-threshold: 2s    # Calls > 2s count as slow
        slow-call-rate-threshold: 80        # Open if 80%+ calls are slow
  retry:
    instances:
      payment-service:
        max-attempts: 3
        wait-duration: 500ms
        exponential-backoff-multiplier: 2   # 500ms, 1s, 2s
        retry-exceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException
  timelimiter:
    instances:
      payment-service:
        timeout-duration: 3s

// Service with combined resilience annotations
@Service
public class PaymentService {
    private final WebClient webClient;

    @CircuitBreaker(name = "payment-service", fallbackMethod = "paymentFallback")
    @Retry(name = "payment-service")
    @TimeLimiter(name = "payment-service")
    public CompletableFuture<PaymentResult> processPayment(PaymentRequest request) {
        return webClient.post()
            .uri("/api/payments")
            .bodyValue(request)
            .retrieve()
            .bodyToMono(PaymentResult.class)
            .toFuture();
    }

    private CompletableFuture<PaymentResult> paymentFallback(PaymentRequest request, Throwable t) {
        log.warn("Payment service unavailable, queueing for retry: {}", t.getMessage());
        paymentQueue.enqueue(request);  // Queue for async retry
        return CompletableFuture.completedFuture(PaymentResult.queued(request.orderId()));
    }
}

// Monitoring circuit breaker state
@EventListener
public void onCircuitBreakerEvent(CircuitBreakerOnStateTransitionEvent event) {
    log.warn("Circuit breaker '{}' transitioned: {} → {}", 
        event.getCircuitBreakerName(),
        event.getStateTransition().getFromState(),
        event.getStateTransition().getToState());
    alertService.notify("Circuit breaker state change: " + event.getCircuitBreakerName());
}
```

**Annotation Ordering (innermost → outermost):**
`Retry → CircuitBreaker → RateLimiter → TimeLimiter → Bulkhead`

**⚠️ Pitfalls:**
- **Don't retry on non-transient errors** (400 Bad Request, 404 Not Found) — only retry on `IOException`, timeouts, and 5xx.
- **Bulkhead + Circuit Breaker** — use together. Circuit breaker prevents cascading failure; bulkhead limits concurrency to prevent thread exhaustion.
- **Fallback methods must have the same signature** plus a `Throwable` parameter, or Spring silently ignores them.

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

---

### Q6. 🌐 What is Spring AOP? How does it work internally, and what are common use cases?

**Spring AOP provides modularization of cross-cutting concerns (logging, security, transactions) by weaving advice (code) around join points (method executions) using proxy-based interception — without modifying the target business logic.**

**How It Works Internally:**
- **JDK Dynamic Proxy** — used when the target implements an interface. Creates a proxy implementing the same interface.
- **CGLIB Proxy** — used when the target is a class (no interface). Creates a subclass at runtime.
- Spring Boot 3 defaults to CGLIB proxies (`spring.aop.proxy-target-class=true`).

```java
// Custom annotation + aspect for method-level auditing
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Audited {
    String action();
}

@Aspect
@Component
@Order(1) // Lower value = higher priority
public class AuditAspect {
    private final AuditService auditService;

    @Around("@annotation(audited)")
    public Object audit(ProceedingJoinPoint joinPoint, Audited audited) throws Throwable {
        String user = SecurityContextHolder.getContext().getAuthentication().getName();
        long start = System.nanoTime();
        try {
            Object result = joinPoint.proceed();
            auditService.log(user, audited.action(), "SUCCESS", durationMs(start));
            return result;
        } catch (Exception ex) {
            auditService.log(user, audited.action(), "FAILED: " + ex.getMessage(), durationMs(start));
            throw ex;
        }
    }
}

// Usage
@Service
public class AccountService {
    @Audited(action = "TRANSFER_FUNDS")
    @Transactional
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        // Business logic — AOP handles auditing transparently
    }
}
```

**⚠️ Pitfalls:**
- **Self-invocation bypasses the proxy** — calling `this.method()` within the same class skips AOP. Use `@Lazy` self-injection or `AopContext.currentProxy()`.
- `@Transactional` IS an AOP aspect — same self-invocation trap applies.
- `@Around` advice MUST call `joinPoint.proceed()` or the target method never executes.

---

### Q7. 🏢 How does service discovery work in microservices? Compare client-side (Eureka) vs. server-side (Kubernetes) discovery.

**Service discovery enables microservices to locate each other dynamically without hardcoded URLs — client-side discovery (Eureka) gives the client a registry to query, while server-side discovery (Kubernetes DNS/Services) delegates resolution to the infrastructure.**

| Feature | Client-Side (Eureka) | Server-Side (K8s) |
|---------|---------------------|-------------------|
| Registry location | Dedicated Eureka server | Built into K8s DNS |
| Load balancing | Client-side (Spring Cloud LoadBalancer) | kube-proxy / Istio |
| Health checks | Eureka heartbeats (30s) | K8s liveness/readiness probes |
| Platform dependency | Cloud-agnostic | Kubernetes required |
| Complexity | More code, self-managed | Infrastructure-managed |

```java
// Eureka client registration (Spring Cloud)
// application.yml
spring:
  application:
    name: order-service
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10

// Using WebClient with service discovery
@Bean
@LoadBalanced  // Enables service-name resolution
public WebClient.Builder webClientBuilder() {
    return WebClient.builder();
}

// Call by service name — Eureka resolves to actual host:port
webClient.get().uri("http://inventory-service/api/stock/{sku}", sku)

// Kubernetes alternative — no Eureka needed
// Service name resolves via K8s DNS:
// http://inventory-service.default.svc.cluster.local:8080/api/stock/{sku}
```

**Production Recommendation:**
- **On Kubernetes** → use K8s Services + Istio/Linkerd for service mesh. No Eureka needed.
- **On VMs/bare metal** → Eureka or HashiCorp Consul.
- **Multi-environment** → Spring Cloud Kubernetes project bridges both worlds.

**⚠️ Pitfalls:**
- Eureka's registry is eventually consistent — a crashed service remains registered for up to 90 seconds.
- K8s DNS-based discovery doesn't do client-side load balancing — all traffic goes through `ClusterIP`.

---

### Q8. 🏢 Explain distributed tracing with OpenTelemetry and how Spring Boot 3 integrates with it.

**Distributed tracing tracks a single request as it flows through multiple microservices by propagating a unique trace ID — Spring Boot 3 uses Micrometer Observation API with OpenTelemetry (or Brave/Zipkin) as the backend, replacing the deprecated Spring Cloud Sleuth.**

**Key Concepts:**
- **Trace** — end-to-end journey of a request across services.
- **Span** — a single unit of work within a trace (e.g., one service call, one DB query).
- **Context Propagation** — trace ID is passed via HTTP headers (`traceparent`, `b3`).

```xml
<!-- pom.xml dependencies (Spring Boot 3.2+) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-exporter-otlp</artifactId>
</dependency>
```

```yaml
# application.yml
management:
  tracing:
    sampling:
      probability: 1.0  # 100% in dev, reduce in prod (0.1 = 10%)
  otlp:
    tracing:
      endpoint: http://otel-collector:4318/v1/traces

logging:
  pattern:
    level: "%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]"
```

```java
// Custom spans for important business operations
@Service
public class PaymentService {
    private final ObservationRegistry observationRegistry;

    public PaymentResult processPayment(PaymentRequest request) {
        return Observation.createNotStarted("payment.process", observationRegistry)
            .lowCardinalityKeyValue("payment.method", request.method().name())
            .observe(() -> {
                // This block is automatically wrapped in a span
                validateRequest(request);
                return chargePaymentGateway(request);
            });
    }
}
```

**⚠️ Pitfalls:**
- Sampling at 100% in production generates massive data. Start with 10% and increase for debugging.
- `@Async` methods lose trace context — use `TaskDecorator` to propagate context to async threads.
- Kafka consumers need manual context extraction from message headers.

---

### Q9. 🏢 How do you handle distributed transactions across microservices? Explain the Saga pattern.

**Distributed transactions across microservices should use the Saga pattern — a sequence of local transactions where each service commits independently, with compensating transactions to undo changes if any step fails — avoiding the performance and availability problems of two-phase commit (2PC).**

**Saga Types:**

| Type | Coordination | Pros | Cons |
|------|-------------|------|------|
| **Choreography** | Events between services | Decoupled, simple | Hard to trace, no central view |
| **Orchestration** | Central orchestrator | Clear flow, easy debugging | Single point of coordination |

```java
// Orchestration Saga — Order creation workflow
@Service
public class OrderSagaOrchestrator {

    public OrderResult createOrder(OrderRequest request) {
        String sagaId = UUID.randomUUID().toString();
        try {
            // Step 1: Reserve inventory
            InventoryReservation reservation = inventoryService.reserve(sagaId, request.items());

            // Step 2: Process payment
            PaymentConfirmation payment;
            try {
                payment = paymentService.charge(sagaId, request.paymentInfo(), request.total());
            } catch (PaymentException e) {
                // Compensate Step 1
                inventoryService.cancelReservation(sagaId);
                throw e;
            }

            // Step 3: Create order
            Order order;
            try {
                order = orderRepository.save(new Order(request, reservation, payment));
            } catch (Exception e) {
                // Compensate Steps 1 & 2
                paymentService.refund(sagaId);
                inventoryService.cancelReservation(sagaId);
                throw e;
            }

            // Step 4: Publish event (async — no compensation needed)
            eventPublisher.publish(new OrderCreatedEvent(order));
            return OrderResult.success(order);

        } catch (Exception e) {
            log.error("Saga {} failed: {}", sagaId, e.getMessage());
            return OrderResult.failed(sagaId, e.getMessage());
        }
    }
}
```

**Key Principles:**
1. **Each step must be idempotent** — retries must be safe.
2. **Each step must have a compensating action** — the "undo" for partial completion.
3. **Use a saga log/event store** — track which steps completed for recovery.
4. **Semantic locks** — mark resources as "pending" until saga completes.

**⚠️ Pitfalls:**
- Compensations can also fail — implement compensation retries with exponential backoff.
- Eventual consistency means the UI must handle "pending" states.
- **Do NOT use 2PC (XA transactions)** across microservices — it creates tight coupling and blocks resources.

---

### Q10. 🌐 What is the difference between `@RestController` and `@Controller`? How do you design RESTful APIs in Spring Boot?

**`@RestController` combines `@Controller` + `@ResponseBody`, automatically serializing return values to JSON/XML; `@Controller` returns view names for server-side rendering — use `@RestController` for APIs and `@Controller` for MVC web pages.**

```java
@RestController
@RequestMapping("/api/v1/orders")
@Tag(name = "Orders", description = "Order management APIs")
public class OrderController {

    private final OrderService orderService;

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @Operation(summary = "Create a new order")
    public ResponseEntity<OrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request) {
        Order order = orderService.create(request);
        URI location = URI.create("/api/v1/orders/" + order.getId());
        return ResponseEntity.created(location).body(OrderResponse.from(order));
    }

    @GetMapping
    public Page<OrderSummary> listOrders(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) OrderStatus status) {
        return orderService.findAll(status, PageRequest.of(page, size));
    }

    @GetMapping("/{orderId}")
    public OrderResponse getOrder(@PathVariable UUID orderId) {
        return orderService.findById(orderId)
            .map(OrderResponse::from)
            .orElseThrow(() -> new ResourceNotFoundException("Order", orderId));
    }

    @PatchMapping("/{orderId}/status")
    public OrderResponse updateStatus(
            @PathVariable UUID orderId,
            @Valid @RequestBody UpdateStatusRequest request) {
        return OrderResponse.from(orderService.updateStatus(orderId, request.status()));
    }

    @DeleteMapping("/{orderId}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void cancelOrder(@PathVariable UUID orderId) {
        orderService.cancel(orderId);
    }
}

// Global exception handler — consistent error responses
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ProblemDetail> handleNotFound(ResourceNotFoundException ex) {
        ProblemDetail detail = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        detail.setTitle("Resource Not Found");
        detail.setProperty("resourceType", ex.getResourceType());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(detail);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ProblemDetail> handleValidation(MethodArgumentNotValidException ex) {
        ProblemDetail detail = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        detail.setTitle("Validation Failed");
        Map<String, String> errors = ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(FieldError::getField, FieldError::getDefaultMessage));
        detail.setProperty("errors", errors);
        return ResponseEntity.badRequest().body(detail);
    }
}
```

**REST API Design Principles:**
- Use **nouns** for resources (`/orders`), not verbs (`/getOrders`).
- Use `ProblemDetail` (RFC 7807) for error responses (Spring Boot 3 native).
- Version via URL path (`/api/v1/`) or `Accept` header.
- Use `PATCH` for partial updates, `PUT` for full replacement.

---

### Q11. 🏬 How do you externalize configuration in Spring Boot for different environments?

**Spring Boot supports externalized configuration through a prioritized property source hierarchy — from command-line args (highest) to `application.yml` (lowest) — with profile-specific files, environment variables, and config servers for multi-environment deployments.**

**Property Source Priority (highest → lowest):**
1. Command-line arguments (`--server.port=9090`)
2. `SPRING_APPLICATION_JSON` environment variable
3. OS environment variables (`SERVER_PORT=9090`)
4. Profile-specific files (`application-prod.yml`)
5. `application.yml` / `application.properties`
6. `@PropertySource` annotations
7. Default properties

```yaml
# application.yml — base/shared config
spring:
  application:
    name: order-service
  datasource:
    url: jdbc:postgresql://localhost:5432/orders
    hikari:
      maximum-pool-size: 10

---
# application-prod.yml — production overrides
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST}:5432/orders
    hikari:
      maximum-pool-size: 50
  
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
```

```java
// Type-safe configuration binding
@ConfigurationProperties(prefix = "app.payment")
@Validated
public record PaymentProperties(
    @NotBlank String gatewayUrl,
    @Min(1) @Max(30) int timeoutSeconds,
    RetryConfig retry
) {
    public record RetryConfig(int maxAttempts, Duration initialDelay) {}
}

// Usage — clean injection
@Service
public class PaymentService {
    private final PaymentProperties config;
    // config.gatewayUrl(), config.timeoutSeconds(), config.retry().maxAttempts()
}
```

**⚠️ Pitfalls:**
- **Never commit secrets** to `application.yml`. Use environment variables, Vault, or AWS Secrets Manager.
- `@Value("${missing.prop}")` throws at startup if the property doesn't exist — use `@Value("${prop:default}")`.
- Relaxed binding maps `app.payment.gateway-url` ↔ `APP_PAYMENT_GATEWAYURL` (env var) — know the naming rules.

---

### Q12. 🏢 What is Spring WebFlux and when should you use it over Spring MVC?

**Spring WebFlux is the reactive, non-blocking web framework built on Project Reactor — it excels when handling thousands of concurrent I/O-bound connections with minimal threads, but adds complexity that is rarely justified unless you have specific backpressure or streaming requirements.**

| Feature | Spring MVC | Spring WebFlux |
|---------|-----------|----------------|
| Programming model | Blocking, thread-per-request | Non-blocking, event-loop |
| Thread usage | 200+ threads (Tomcat pool) | ~4 event-loop threads (Netty) |
| Best for | CRUD apps, traditional REST | Streaming, high concurrency, SSE |
| Backpressure | No | Yes (Reactive Streams) |
| Debugging | Stack traces are clear | Callback-based, harder to debug |

```java
// WebFlux reactive endpoint
@RestController
@RequestMapping("/api/v1/events")
public class EventController {

    @GetMapping(produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<StockPrice>> streamPrices() {
        return stockService.priceStream()
            .map(price -> ServerSentEvent.<StockPrice>builder()
                .data(price)
                .event("price-update")
                .build());
    }
}
```

**When to Choose WebFlux:**
- ✅ Real-time streaming (SSE, WebSocket dashboards)
- ✅ API gateways proxying many downstream calls
- ✅ High fan-out with many concurrent external HTTP calls
- ❌ CRUD-heavy apps with blocking JDBC — no benefit
- ❌ Small team unfamiliar with reactive programming

**⚠️ Pitfall:** With Java 21 Virtual Threads, Spring MVC handles massive concurrency without reactive complexity. **Virtual Threads are the recommended default** for most applications over WebFlux now.

---

## 🟢 GOOD TO KNOW (Questions 13–20)

---

### Q13. 🏢 How do you implement API Gateway pattern in microservices?

**An API Gateway is a single entry point for all client requests that handles cross-cutting concerns (routing, auth, rate limiting) and protocol translation — Spring Cloud Gateway (reactive) is the standard choice in the Spring ecosystem.**

```yaml
# Spring Cloud Gateway config
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service  # Load-balanced via service discovery
          predicates:
            - Path=/api/v1/orders/**
          filters:
            - StripPrefix=0
            - name: CircuitBreaker
              args:
                name: orderCB
                fallbackUri: forward:/fallback/orders
            - name: RequestRateLimiter
              args:
                redis-rate-limiter:
                  replenishRate: 100
                  burstCapacity: 200
```

**⚠️ Pitfall:** API Gateway can become a single point of failure — always deploy multiple instances behind a load balancer.

---

### Q14. 🌐 How does Spring Boot Actuator help in production monitoring?

**Spring Boot Actuator exposes production-ready operational endpoints for health checks, metrics, environment inspection, and runtime management — integrating with Prometheus, Grafana, and Kubernetes probes out of the box.**

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus,env
  endpoint:
    health:
      show-details: when-authorized
      group:
        readiness:
          include: db,redis,kafka
        liveness:
          include: livenessState
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
```

```java
// Custom health indicator
@Component
public class PaymentGatewayHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        boolean reachable = paymentGateway.ping();
        return reachable 
            ? Health.up().withDetail("latency", paymentGateway.latencyMs() + "ms").build()
            : Health.down().withDetail("error", "Gateway unreachable").build();
    }
}
```

**⚠️ Pitfall:** Never expose all actuator endpoints in production. Restrict with `include` and secure with Spring Security.

---

### Q15. 🌐 What is the `@Transactional` annotation and what are its propagation behaviors?

**`@Transactional` demarcates transactional boundaries declaratively — Spring wraps the method in a proxy that begins/commits/rolls back transactions, with propagation controlling how transactions interact when methods call other `@Transactional` methods.**

| Propagation | Behavior |
|------------|----------|
| `REQUIRED` (default) | Join existing txn or create new one |
| `REQUIRES_NEW` | Always create new txn, suspend existing |
| `NESTED` | Create a savepoint within existing txn |
| `MANDATORY` | Must run within existing txn or throw exception |
| `NOT_SUPPORTED` | Suspend existing txn, run non-transactionally |

```java
@Service
public class OrderService {
    @Transactional  // REQUIRED by default
    public void createOrder(OrderRequest request) {
        Order order = orderRepository.save(new Order(request));
        paymentService.processPayment(order);  // Joins same transaction
        notificationService.sendConfirmation(order);  // Also same transaction
    }
}

@Service
public class NotificationService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendConfirmation(Order order) {
        // Runs in a NEW transaction — if this fails, 
        // the order creation still commits
        notificationRepo.save(new Notification(order));
    }
}
```

**⚠️ Pitfalls:**
- `@Transactional` on private methods does nothing — proxies can't intercept private calls.
- Self-invocation (`this.method()`) bypasses the proxy — the `@Transactional` annotation is ignored.
- Default rollback is on **unchecked exceptions only** — checked exceptions commit. Use `rollbackFor = Exception.class` to change.

---

### Q16. 🏬 How do you implement API versioning in Spring Boot?

**API versioning strategies include URI path (`/v1/`), query parameter, custom header, and content negotiation — URI path versioning is the most widely adopted for its simplicity and discoverability, though header-based is cleaner for complex APIs.**

```java
// Strategy 1: URI versioning (most common)
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 { }

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 { }

// Strategy 2: Custom header versioning
@RestController
@RequestMapping("/api/users")
public class UserController {
    @GetMapping(headers = "X-API-Version=1")
    public UserV1Response getUserV1(@PathVariable Long id) { }

    @GetMapping(headers = "X-API-Version=2")
    public UserV2Response getUserV2(@PathVariable Long id) { }
}
```

**⚠️ Pitfall:** Don't version aggressively — it multiplies maintenance. Use backward-compatible changes (additive fields, optional params) as long as possible.

---

### Q17. 🏢 What is Spring Cloud Config and how does centralized configuration work?

**Spring Cloud Config provides server-side centralized configuration management backed by Git, Vault, or databases — services fetch configuration at startup and can refresh at runtime without restarting.**

```yaml
# Config Server
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/company/config-repo
          default-label: main
          search-paths: '{application}'

# Client service
spring:
  config:
    import: configserver:http://config-server:8888
  cloud:
    config:
      fail-fast: true
      retry:
        max-attempts: 5
```

```java
// Refresh config without restart
@RefreshScope
@Component
public class FeatureFlags {
    @Value("${feature.new-checkout.enabled:false}")
    private boolean newCheckoutEnabled;
}
// POST /actuator/refresh → Spring re-creates this bean with updated values
```

**⚠️ Pitfall:** On Kubernetes, ConfigMaps + `spring-cloud-kubernetes-config` is preferred over a dedicated config server.

---

### Q18. 🌐 How do you handle request validation in Spring Boot?

**Spring Boot integrates Jakarta Bean Validation (Hibernate Validator) for declarative request validation — annotate DTOs with constraint annotations and use `@Valid` on controller parameters to trigger automatic validation with structured error responses.**

```java
public record CreateUserRequest(
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2-100 characters")
    String name,

    @NotBlank @Email(message = "Invalid email format")
    String email,

    @NotNull @Min(18) @Max(150)
    Integer age,

    @Pattern(regexp = "^\\+[1-9]\\d{1,14}$", message = "Invalid phone number")
    String phone
) {}

// Custom cross-field validator
@Constraint(validatedBy = DateRangeValidator.class)
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidDateRange {
    String message() default "End date must be after start date";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

**⚠️ Pitfall:** `@Validated` (Spring) on a class enables method-level validation (e.g., `@Min` on parameters). `@Valid` (Jakarta) on a parameter triggers bean validation. They are NOT interchangeable.

---

### Q19. 🏢 How do you implement event-driven microservices with Spring Cloud Stream?

**Spring Cloud Stream abstracts messaging middleware (Kafka, RabbitMQ) behind a binder API — services declare `Function`, `Consumer`, or `Supplier` beans that Spring auto-binds to message channels, enabling middleware-agnostic event-driven architectures.**

```java
// Producer — functional style (no annotations needed)
@Bean
public Supplier<Flux<OrderEvent>> orderEvents() {
    return () -> orderEventSink.asFlux();
}

// Consumer
@Bean
public Consumer<OrderCreatedEvent> processOrder() {
    return event -> {
        log.info("Processing order: {}", event.orderId());
        inventoryService.reserve(event.items());
    };
}

// Processor (transform and forward)
@Bean
public Function<OrderCreatedEvent, ShipmentRequest> prepareShipment() {
    return event -> new ShipmentRequest(event.orderId(), event.shippingAddress());
}
```

```yaml
spring:
  cloud:
    stream:
      bindings:
        processOrder-in-0:
          destination: order-events
          group: inventory-service  # Consumer group for load balancing
      kafka:
        binder:
          brokers: kafka:9092
```

**⚠️ Pitfall:** Always set a `group` — without it, every instance of the service processes every message (broadcast), instead of load-balanced consumption.

---

### Q20. 🏬 What are Spring Profiles and how do you use them effectively?

**Spring Profiles provide a mechanism to segregate parts of your application configuration and bean registration, making them available only in specific environments — activated via `spring.profiles.active` property, environment variable, or programmatically.**

```java
// Profile-specific beans
@Configuration
public class StorageConfig {
    @Bean
    @Profile("local")
    public StorageService localStorage() {
        return new FileSystemStorageService("/tmp/uploads");
    }

    @Bean
    @Profile("prod")
    public StorageService s3Storage(S3Client s3Client) {
        return new S3StorageService(s3Client, "prod-uploads-bucket");
    }
}
```

```yaml
# Activate profile
# Option 1: application.yml
spring.profiles.active: local

# Option 2: Environment variable (Docker/K8s)
SPRING_PROFILES_ACTIVE=prod

# Option 3: Command line
java -jar app.jar --spring.profiles.active=prod

# Profile groups (Spring Boot 2.4+)
spring:
  profiles:
    group:
      prod: "prod,metrics,ssl"
      local: "local,h2,swagger"
```

**⚠️ Pitfalls:**
- **Don't use profiles for feature flags** — profiles are for environment-specific config. Use a feature flag system (LaunchDarkly, Spring Cloud Config).
- **`@Profile("!prod")`** — negation works but is hard to reason about. Prefer explicit profiles.
- **Default profile** — if no profile is active, Spring uses `default`. Use this for development.
