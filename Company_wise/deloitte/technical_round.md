# Deloitte Technical Round - Frequently Asked Questions

> **Skill Set:** Java, Angular, AWS, PostgreSQL, Spring Boot Microservices, CI/CD (GitLab)
> **Sources:** AmbitionBox, Glassdoor, GeeksforGeeks, InterviewQuery — re-ranked by frequency for the above skill set.

---

## 🔥 JAVA CORE (Most Frequently Asked)

---

### 1. Explain the four pillars of Object-Oriented Programming (OOP) with real-world examples.
**Answer:**
- **Encapsulation:** Wrapping data and methods into a single unit (class). Example: A bank account where balance is hidden and accessed only through deposit/withdraw methods.
- **Abstraction:** Hiding internal implementation details and showing only functionality. Example: Driving a car — you use steering and pedals without knowing engine internals.
- **Inheritance:** One class acquires the properties and behaviors of another. Example: `Vehicle` → `Car`, `Bike`.
- **Polymorphism:** A single function/object takes multiple forms. Achieved via method overloading (compile-time) and overriding (run-time).

---

### 2. What are Java 8 Streams? Write code using filter, map, and collect.
**Answer:**
Java Streams provide a functional, declarative approach to process collections.
```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6);

// Filter even numbers, square them, collect to list
List<Integer> result = nums.stream()
    .filter(n -> n % 2 == 0)       // [2, 4, 6]
    .map(n -> n * n)               // [4, 16, 36]
    .collect(Collectors.toList());

// Find the first non-repeating character in a string
String input = "aabbcde";
Character firstNonRepeating = input.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(Function.identity(), LinkedHashMap::new, Collectors.counting()))
    .entrySet().stream()
    .filter(e -> e.getValue() == 1)
    .map(Map.Entry::getKey)
    .findFirst().orElse(null); // 'c'

// Collect duplicates from a list
List<Integer> duplicates = nums.stream()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
    .entrySet().stream()
    .filter(e -> e.getValue() > 1)
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());
```

---

### 3. How does a HashMap work internally in Java? Difference between HashMap, TreeMap, and ConcurrentHashMap?
**Answer:**
`HashMap` uses an array of `Node` (key-value pairs). On `put(key, value)`, it computes the hash code → determines bucket index → stores or chains via linked list (or Red-Black tree after 8 collisions in Java 8+).

| Feature           | HashMap                     | TreeMap                      | ConcurrentHashMap             |
| ----------------- | --------------------------- | ---------------------------- | ----------------------------- |
| Order             | No guaranteed order         | Sorted by keys               | No guaranteed order           |
| Null keys         | Allows 1 null key           | No null keys                 | No null keys or values        |
| Performance       | O(1) average                | O(log n)                     | O(1) average                  |
| Thread Safety     | Not thread-safe             | Not thread-safe              | Thread-safe (bucket-level locking) |
| Iterator          | Fail-fast                   | Fail-fast                    | Fail-safe (weakly consistent) |

---

### 4. What is the difference between `String`, `StringBuilder`, and `StringBuffer`? Why is String immutable?
**Answer:**
| Feature       | String           | StringBuilder    | StringBuffer     |
| ------------- | ---------------- | ---------------- | ---------------- |
| Mutability    | Immutable        | Mutable          | Mutable          |
| Thread Safety | Yes (immutable)  | No               | Yes (synchronized)|
| Performance   | Slow (creates new objects) | Fastest | Slower than SB   |

**Why is String immutable?**
- **String Pool:** Allows caching/sharing in the String Pool, saving memory.
- **Security:** Strings used in class loading, URLs, DB connections — immutability prevents malicious modification.
- **Hashcode caching:** Hash code computed once and cached, making `HashMap` lookups faster.

---

### 5. What is the difference between an Abstract Class and an Interface?
**Answer:**
- **Abstract Class:** Both abstract and concrete methods. Can have instance variables (state). Single inheritance only. Use for closely related classes.
- **Interface:** Prior to Java 8 — only abstract methods. Now supports `default` and `static` methods. No state. Multiple interfaces allowed. Use as a contract for unrelated classes.

---

### 6. What is the difference between Method Overloading and Method Overriding?
**Answer:**
| Feature       | Overloading (Compile-time)     | Overriding (Run-time)            |
| ------------- | ------------------------------ | -------------------------------- |
| Scope         | Within the same class          | Between parent and child class   |
| Parameters    | Must differ                    | Must be the same                 |
| Return type   | Can differ                     | Same or covariant                |
| Binding       | Static (at compile time)       | Dynamic (at run time)            |

---

### 7. Explain Exception Handling — Checked vs Unchecked. Difference between `final`, `finally`, `finalize`.
**Answer:**
- **Checked Exceptions:** Checked at compile-time. Must handle with try-catch or `throws`. Examples: `IOException`, `SQLException`.
- **Unchecked Exceptions:** Runtime errors. Programming bugs. Examples: `NullPointerException`, `ArrayIndexOutOfBoundsException`.

```
Throwable
├── Error (OutOfMemoryError, StackOverflowError)
└── Exception
    ├── Checked (IOException, SQLException)
    └── RuntimeException / Unchecked (NullPointerException)
```

| Keyword       | Purpose                                                                 |
| ------------- | ----------------------------------------------------------------------- |
| `final`       | Variable = constant, Method = can't override, Class = can't inherit     |
| `finally`     | Block that always executes after try-catch — resource cleanup           |
| `finalize()`  | GC calls before destroying object. **Deprecated since Java 9** — use `try-with-resources` |

---

### 8. Explain the Java Memory Model — Stack vs Heap.
**Answer:**
- **Stack:** Method call frames, local variables, references. LIFO. Each thread has its own stack. Auto-freed. Fast.
- **Heap:** All objects created with `new`. Shared across threads. Managed by Garbage Collector. Slower.

```
Thread 1 Stack           Heap
┌──────────────┐     ┌──────────────────┐
│ int x = 5    │     │ Employee obj     │
│ ref → obj ───┼────▶│  name = "John"   │
└──────────────┘     │  age = 30        │
                     └──────────────────┘
```

---

### 9. How does Multithreading work in Java? `Runnable` vs `Thread`?
**Answer:**
- **Extending `Thread`:** Override `run()`. Cannot extend another class.
- **Implementing `Runnable` (preferred):** Separates task from runner. Allows extending another class.

```java
// Using Runnable + Lambda (Java 8+)
Runnable task = () -> System.out.println("Thread: " + Thread.currentThread().getName());
new Thread(task).start();

// Using ExecutorService (production-grade)
ExecutorService executor = Executors.newFixedThreadPool(5);
executor.submit(task);
executor.shutdown();
```
**Key Concepts:** `synchronized`, `volatile`, `wait()/notify()`, `ExecutorService`, `CompletableFuture`.

---

### 10. What are Functional Interfaces and Lambda Expressions in Java 8?
**Answer:**
A **Functional Interface** has exactly one abstract method. Can be used as lambda target.
```java
@FunctionalInterface
interface Calculator {
    int operate(int a, int b);
}

Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;
System.out.println(add.operate(5, 3));      // 8
```
**Built-in:** `Predicate<T>`, `Function<T,R>`, `Consumer<T>`, `Supplier<T>`, `BiFunction<T,U,R>`.

---

### 11. What are Access Modifiers in Java?
**Answer:**
| Modifier    | Same Class | Same Package | Subclass (other pkg) | Everywhere |
| ----------- | ---------- | ------------ | -------------------- | ---------- |
| `private`   | ✅         | ❌           | ❌                   | ❌         |
| `default`   | ✅         | ✅           | ❌                   | ❌         |
| `protected` | ✅         | ✅           | ✅                   | ❌         |
| `public`    | ✅         | ✅           | ✅                   | ✅         |

---

### 12. What is the difference between an Array and an ArrayList?
**Answer:**
| Feature       | Array                        | ArrayList                      |
| ------------- | ---------------------------- | ------------------------------ |
| Size          | Fixed at creation            | Dynamic (grows/shrinks)        |
| Type          | Primitives & objects         | Only objects (autoboxing)      |
| Performance   | Faster (direct access)       | Slightly slower                |
| API           | No built-in methods          | Rich API (`add`, `remove`, `contains`) |

---

## 🔥 SPRING BOOT + MICROSERVICES

---

### 13. What is Dependency Injection (DI) and Inversion of Control (IoC)? Why is Constructor Injection preferred?
**Answer:**
- **IoC:** The framework (Spring container) manages object creation, not the programmer.
- **DI:** Dependencies are injected at runtime by the container.

**Why Constructor Injection is preferred:**
- Dependencies are `final` and immutable — set once, never changed.
- Makes the class testable — easy to pass mocks in unit tests.
- Fails fast at startup if a dependency is missing (not at runtime like field injection).

```java
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;

    // Constructor Injection — no @Autowired needed if single constructor
    public OrderService(PaymentService paymentService, InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
}
```

---

### 14. Explain `@Component` vs `@Service` vs `@Repository` vs `@Controller`. How many ways to define a Bean?
**Answer:**
All are `@Component` specializations, auto-detected by `@ComponentScan`.
| Annotation     | Layer            | Special Behavior                                    |
| -------------- | ---------------- | --------------------------------------------------- |
| `@Component`   | Generic          | Base annotation for any Spring-managed bean          |
| `@Service`     | Business Logic   | Semantic marker for service layer                    |
| `@Repository`  | Data Access      | Translates persistence exceptions → `DataAccessException` |
| `@Controller`  | Web / MVC        | Handles HTTP requests, returns views                 |
| `@RestController` | REST API      | `@Controller` + `@ResponseBody` (returns JSON)      |

**3 ways to define a Bean:**
1. **Stereotype annotations** (`@Component`, `@Service`, etc.) + `@ComponentScan`
2. **`@Bean` methods** inside `@Configuration` classes
3. **Auto-configuration** via `@EnableAutoConfiguration` (Spring Boot starters)

---

### 15. What are Microservices? How are they different from Monolithic architecture?
**Answer:**
| Feature           | Monolithic                     | Microservices                     |
| ----------------- | ------------------------------ | --------------------------------- |
| Deployment        | Single deployable unit         | Independent deployable services   |
| Scaling           | Scale entire app               | Scale individual services         |
| Tech stack        | Single stack                   | Polyglot (different per service)  |
| Failure impact    | One bug can crash everything   | Fault isolation per service       |
| Complexity        | Simple to develop initially    | Distributed system complexity     |
| Data              | Shared database                | Database per service              |

**When to use Microservices:** Large teams, independent scaling requirements, different release cycles per feature.

---

### 16. What Microservices Design Patterns have you used?
**Answer:**
- **API Gateway (Spring Cloud Gateway):** Single entry point for all clients. Handles routing, rate limiting, authentication.
- **Service Discovery (Eureka / Consul):** Services register themselves; consumers discover them dynamically.
- **Circuit Breaker (Resilience4j):** Prevents cascading failures when a downstream service is down. States: CLOSED → OPEN → HALF_OPEN.
- **SAGA Pattern:** Manages distributed transactions across multiple services using a sequence of local transactions with compensating actions on failure.
- **Config Server (Spring Cloud Config):** Centralized externalized configuration for all microservices.
- **Event-Driven / Async Communication (Kafka/RabbitMQ):** Decouples services. Producer publishes events, consumers process independently.

```java
// Circuit Breaker with Resilience4j
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
public PaymentResponse processPayment(PaymentRequest request) {
    return paymentClient.charge(request);
}

public PaymentResponse paymentFallback(PaymentRequest request, Throwable t) {
    return new PaymentResponse("PENDING", "Payment service unavailable. Queued for retry.");
}
```

---

### 17. How do you secure a Spring Boot REST API? Explain JWT + OAuth2 flow.
**Answer:**
Use **Spring Security** with **JWT (JSON Web Token)** for stateless authentication.

**Flow:**
1. Client sends credentials (username/password) to `/auth/login`.
2. Server validates, generates a JWT token (with claims, expiry) and returns it.
3. Client stores the token and sends it in the `Authorization: Bearer <token>` header with every subsequent request.
4. A `JwtAuthenticationFilter` intercepts requests, validates the token, and sets the `SecurityContext`.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf(csrf -> csrf.disable())
            .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**").permitAll()
                .anyRequest().authenticated())
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }
}
```

---

### 18. Explain Hibernate ORM. Write code for a One-to-Many relationship.
**Answer:**
Hibernate maps Java objects to database tables, eliminating manual JDBC code.
```java
@Entity
public class Department {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Employee> employees = new ArrayList<>();
}

@Entity
public class Employee {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
}
```
- **`mappedBy`** = inverse side (no FK column here). **`@JoinColumn`** = owning side (FK lives here).
- **`FetchType.LAZY`** = load on demand (default for collections, preferred for performance).
- **`CascadeType.ALL`** = persist/merge/remove propagates to children.

---

### 19. How do you handle Global Exception Handling in Spring Boot REST APIs?
**Answer:**
Use `@RestControllerAdvice` with `@ExceptionHandler`.
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(new ErrorResponse(404, ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        String msg = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return ResponseEntity.badRequest().body(new ErrorResponse(400, msg));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        return ResponseEntity.status(500).body(new ErrorResponse(500, "Internal Server Error"));
    }
}
```

---

### 20. What is Spring Boot Actuator? What endpoints does it provide?
**Answer:**
Actuator provides production-ready monitoring and management features.
| Endpoint               | Purpose                                 |
| ---------------------- | --------------------------------------- |
| `/actuator/health`     | Health status (UP/DOWN)                 |
| `/actuator/info`       | Application info                        |
| `/actuator/metrics`    | JVM memory, CPU, HTTP request metrics   |
| `/actuator/env`        | Environment properties                  |
| `/actuator/loggers`    | View and change log levels at runtime   |
| `/actuator/httptrace`  | Recent HTTP request traces              |

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, loggers
  endpoint:
    health:
      show-details: always
```

---

### 21. What is Connection Pooling? How does HikariCP work in Spring Boot?
**Answer:**
Connection pooling reuses pre-created DB connections instead of opening/closing for every request.
- **Without pooling:** Each request → TCP handshake + auth → query → close. Expensive.
- **With pooling (HikariCP — Spring Boot default):** Pool created at startup. Requests borrow, use, and return connections.

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
```

---

### 22. Explain `@Transactional` in Spring. What are isolation levels and propagation types?
**Answer:**
`@Transactional` manages database transactions declaratively. If any exception occurs, the transaction is rolled back.

**Propagation Types:**
| Type            | Behavior                                             |
| --------------- | ---------------------------------------------------- |
| `REQUIRED`      | Join existing TX or create new (default)             |
| `REQUIRES_NEW`  | Always create a new TX, suspend existing             |
| `MANDATORY`     | Must run within an existing TX, else throws exception|
| `NOT_SUPPORTED` | Suspend existing TX, run without TX                  |

**Isolation Levels:**
| Level              | Prevents                            |
| ------------------ | ----------------------------------- |
| `READ_UNCOMMITTED` | Nothing                             |
| `READ_COMMITTED`   | Dirty reads                         |
| `REPEATABLE_READ`  | Dirty reads + non-repeatable reads  |
| `SERIALIZABLE`     | All anomalies (slowest)             |

```java
@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.READ_COMMITTED)
public void transferFunds(Long fromId, Long toId, BigDecimal amount) {
    accountRepository.debit(fromId, amount);
    accountRepository.credit(toId, amount);
}
```

---

## 🔥 ANGULAR

---

### 23. What are Angular Lifecycle Hooks? Explain the most important ones.
**Answer:**
| Hook                | When it fires                                      |
| ------------------- | -------------------------------------------------- |
| `ngOnChanges`       | When an `@Input` property value changes             |
| `ngOnInit`          | Once after first `ngOnChanges` — initialization     |
| `ngDoCheck`         | Every change detection run                          |
| `ngAfterViewInit`   | After component's view is initialized               |
| `ngOnDestroy`       | Before component is destroyed — cleanup             |

```typescript
export class UserComponent implements OnInit, OnDestroy {
  private subscription: Subscription;

  ngOnInit() {
    this.subscription = this.userService.getUsers().subscribe(data => this.users = data);
  }
  ngOnDestroy() {
    this.subscription.unsubscribe(); // Prevent memory leaks
  }
}
```

---

### 24. What is Angular Data Binding? Explain the types.
**Answer:**
| Type                  | Syntax                   | Direction            |
| --------------------- | ------------------------ | -------------------- |
| Interpolation         | `{{ expression }}`       | Component → View     |
| Property Binding      | `[property]="expr"`      | Component → View     |
| Event Binding         | `(event)="handler()"`    | View → Component     |
| Two-way Binding       | `[(ngModel)]="property"` | Both directions      |

```html
<h1>{{ title }}</h1>
<img [src]="imageUrl">
<button (click)="onClick()">Click Me</button>
<input [(ngModel)]="username">  <!-- Requires FormsModule -->
```

---

### 25. Explain routing in Angular. What are lazy-loaded modules?
**Answer:**
Angular's `RouterModule` maps URL paths to components.
```typescript
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users', component: UserListComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: 'admin', loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule) }, // Lazy loading
  { path: '**', component: NotFoundComponent }
];
```
**Lazy Loading:** Modules are loaded on demand (when the user navigates to that route) instead of at application startup. This reduces the initial bundle size and improves load time.

---

### 26. What are Angular Services and how does Dependency Injection work in Angular?
**Answer:**
Services are `@Injectable()` classes that encapsulate reusable business logic.
```typescript
@Injectable({ providedIn: 'root' }) // Singleton — available app-wide
export class UserService {
  constructor(private http: HttpClient) {}
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
}
```
Injected into components via constructor:
```typescript
@Component({ selector: 'app-users', templateUrl: './users.component.html' })
export class UsersComponent {
  constructor(private userService: UserService) {} // Injected automatically
}
```

---

### 27. What are Angular Pipes? How do you create a custom pipe?
**Answer:**
Pipes transform data in templates. Built-in: `date`, `uppercase`, `currency`, `async`.
```typescript
@Pipe({ name: 'truncate' })
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number = 50): string {
    return value.length > limit ? value.substring(0, limit) + '...' : value;
  }
}
```
```html
<p>{{ longText | truncate:100 }}</p>
<p>{{ today | date:'dd/MM/yyyy' }}</p>
```

---

### 28. What is the difference between `Observable` and `Promise` in Angular?
**Answer:**
| Feature       | Promise                      | Observable (RxJS)              |
| ------------- | ---------------------------- | ------------------------------ |
| Values        | Single value                 | Multiple values over time      |
| Lazy/Eager    | Eager (executes immediately) | Lazy (executes on subscribe)   |
| Cancellable   | No                           | Yes (unsubscribe)              |
| Operators     | `.then()`, `.catch()`        | `map`, `filter`, `switchMap`, `debounceTime`, etc. |
| Use case      | One-time async operation     | Event streams, HTTP calls, WebSocket |

```typescript
// Observable
this.http.get<User[]>('/api/users').pipe(
  map(users => users.filter(u => u.active)),
  catchError(err => of([]))
).subscribe(data => this.users = data);
```

---

## 🔥 POSTGRESQL

---

### 29. Write a SQL query to find the Nth highest salary. Explain window functions.
**Answer:**
```sql
-- Nth highest using DENSE_RANK (PostgreSQL)
WITH RankedSalaries AS (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rnk
    FROM employees
)
SELECT DISTINCT salary FROM RankedSalaries WHERE rnk = 3; -- 3rd highest

-- Using LIMIT/OFFSET (PostgreSQL specific)
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 2;
```
**Window Functions** perform calculations across rows related to the current row without collapsing them:
- `ROW_NUMBER()` — unique sequential number
- `RANK()` — rank with gaps
- `DENSE_RANK()` — rank without gaps
- `LAG()` / `LEAD()` — access previous/next row

---

### 30. Explain Joins in SQL. Write a query using different join types.
**Answer:**
| Join Type      | Returns                                                  |
| -------------- | -------------------------------------------------------- |
| INNER JOIN     | Only matching rows from both tables                      |
| LEFT JOIN      | All rows from left + matched from right (NULL if no match)|
| RIGHT JOIN     | All rows from right + matched from left                  |
| FULL OUTER JOIN| All rows from both (NULL where no match)                 |
| CROSS JOIN     | Cartesian product                                        |
| SELF JOIN      | Table joined with itself                                 |

```sql
-- Find employees with their department names (and those without a department)
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;

-- Find employees earning more than their manager
SELECT e.name AS employee, e.salary AS emp_salary, m.name AS manager, m.salary AS mgr_salary
FROM employees e
INNER JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

---

### 31. What are Indexes in PostgreSQL? When should you NOT use them?
**Answer:**
An index (typically B-Tree) speeds up `SELECT` queries by creating a sorted lookup structure.

**Types in PostgreSQL:**
- **B-Tree (default):** For equality and range queries (`=`, `<`, `>`, `BETWEEN`).
- **Hash:** For equality only (`=`). Rarely used.
- **GIN (Generalized Inverted Index):** For full-text search, JSONB, arrays.
- **GiST:** For geometric/spatial data, range types.

**When NOT to use indexes:**
- Small tables (sequential scan is faster).
- Columns with low cardinality (e.g., boolean, gender).
- Heavily write-intensive tables (indexes slow down INSERT/UPDATE/DELETE).
- Columns rarely used in WHERE/JOIN/ORDER BY.

```sql
-- Create index
CREATE INDEX idx_employees_email ON employees(email);

-- Composite index
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date DESC);

-- Partial index (only active users)
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- Check if query uses index
EXPLAIN ANALYZE SELECT * FROM employees WHERE email = 'john@test.com';
```

---

### 32. Explain Normalization and Denormalization. When to use each?
**Answer:**
**Normalization** — reduces redundancy, improves data integrity:
- **1NF:** Atomic values, no repeating groups.
- **2NF:** No partial dependency on composite primary key.
- **3NF:** No transitive dependency.

**Denormalization** — adds redundancy for read performance:
- Use in read-heavy systems, reporting databases, or caching layers.
- Example: Storing `department_name` directly in the `employees` table instead of joining every time.

---

### 33. What are Triggers, Stored Procedures, and CTEs in PostgreSQL?
**Answer:**
- **Trigger:** Auto-fires on INSERT/UPDATE/DELETE. Used for auditing, validation.
- **Stored Procedure:** Precompiled SQL block. Reduces network traffic, promotes reuse.
- **CTE (Common Table Expression):** Temporary named result set using `WITH`. Improves readability.

```sql
-- CTE: Find departments with avg salary > 80000
WITH dept_avg AS (
    SELECT department_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT d.name, da.avg_salary
FROM dept_avg da
JOIN departments d ON d.id = da.department_id
WHERE da.avg_salary > 80000;

-- Audit Trigger
CREATE OR REPLACE FUNCTION audit_employee_changes()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO employee_audit(employee_id, action, changed_at)
    VALUES (NEW.id, TG_OP, NOW());
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_employee_audit
AFTER INSERT OR UPDATE ON employees
FOR EACH ROW EXECUTE FUNCTION audit_employee_changes();
```

---

### 34. How do you optimize a slow PostgreSQL query?
**Answer:**
1. **`EXPLAIN ANALYZE`** — analyze the actual execution plan.
2. **Add indexes** on columns in `WHERE`, `JOIN`, `ORDER BY`.
3. **Avoid `SELECT *`** — fetch only required columns.
4. **Fix N+1 queries** — use `JOIN FETCH` in JPA or batch fetching.
5. **Use pagination** — `LIMIT/OFFSET` or keyset pagination (`WHERE id > last_id`).
6. **VACUUM / ANALYZE** — update statistics for the query planner.
7. **Use connection pooling** (PgBouncer or HikariCP).
8. **Partition large tables** — range or list partitioning.

```sql
-- Before: Full table scan
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 123;

-- After: Index scan
CREATE INDEX idx_orders_customer ON orders(customer_id);
EXPLAIN ANALYZE SELECT order_id, total FROM orders WHERE customer_id = 123;
```

---

## 🔥 AWS (Amazon Web Services)

---

### 35. What AWS services have you used? Explain S3, EC2, and RDS.
**Answer:**
| Service | Purpose                                           | Key Features                              |
| ------- | ------------------------------------------------- | ----------------------------------------- |
| **S3**  | Object storage (files, images, backups)            | Buckets, versioning, lifecycle policies, presigned URLs |
| **EC2** | Virtual servers (compute)                          | Instance types, Auto Scaling Groups, Security Groups |
| **RDS** | Managed relational database (PostgreSQL, MySQL)    | Automated backups, read replicas, Multi-AZ |
| **Lambda** | Serverless compute (event-driven functions)     | Pay per invocation, max 15 min runtime    |
| **SQS** | Managed message queue                              | Standard (at-least-once) vs FIFO (exactly-once) |
| **CloudWatch** | Monitoring, logging, alarms                  | Metrics, dashboards, log groups           |
| **ECR/ECS** | Container registry / Container orchestration   | Docker deployments, Fargate (serverless)  |

---

### 36. How do you deploy a Spring Boot application on AWS?
**Answer:**
**Common deployment strategies:**

1. **EC2 (Traditional):**
   - Build JAR → `mvn clean package`
   - SCP to EC2 → `java -jar app.jar`
   - Use `systemd` to run as a service.

2. **ECS + Fargate (Containerized — Preferred):**
   - Dockerize the app → push image to ECR.
   - Create ECS Task Definition → ECS Service with ALB.
   - Auto-scaling based on CPU/memory.

3. **Elastic Beanstalk (PaaS):**
   - Upload JAR directly → Beanstalk handles infra.

```dockerfile
# Dockerfile for Spring Boot
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/myapp.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### 37. What is S3? How do you upload/download files from a Spring Boot application?
**Answer:**
S3 (Simple Storage Service) is an object storage service for any amount of data.
```java
@Service
public class S3Service {
    private final S3Client s3Client;

    public String uploadFile(MultipartFile file) {
        String key = UUID.randomUUID() + "_" + file.getOriginalFilename();
        s3Client.putObject(
            PutObjectRequest.builder().bucket("my-bucket").key(key).build(),
            RequestBody.fromInputStream(file.getInputStream(), file.getSize())
        );
        return key;
    }

    public byte[] downloadFile(String key) {
        ResponseInputStream<GetObjectResponse> response = s3Client.getObject(
            GetObjectRequest.builder().bucket("my-bucket").key(key).build()
        );
        return response.readAllBytes();
    }
}
```

---

### 38. What is the difference between SQS and SNS? When do you use Kafka vs SQS?
**Answer:**
| Feature   | SQS (Queue)                  | SNS (Notification)           |
| --------- | ---------------------------- | ---------------------------- |
| Model     | Point-to-point (1 consumer)  | Pub/Sub (many subscribers)   |
| Delivery  | Pull-based                   | Push-based                   |
| Use case  | Task queues, decoupling      | Fan-out notifications        |

| Feature    | SQS                          | Kafka                          |
| ---------- | ---------------------------- | ------------------------------ |
| Managed    | Fully managed (AWS)          | Self-managed or MSK            |
| Replay     | No (message deleted after read) | Yes (offset-based replay)   |
| Throughput | Moderate                     | Very high                      |
| Use case   | Simple async tasks           | Event sourcing, streaming      |

---

## 🔥 CI/CD — GitLab

---

### 39. Explain your CI/CD pipeline. How does GitLab CI/CD work?
**Answer:**
GitLab CI/CD uses a `.gitlab-ci.yml` file at the project root to define the pipeline.

**Pipeline stages (typical):**
```yaml
stages:
  - build
  - test
  - sonar
  - docker
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"

build:
  stage: build
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn clean package -DskipTests
  artifacts:
    paths:
      - target/*.jar

test:
  stage: test
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn test
  artifacts:
    reports:
      junit: target/surefire-reports/*.xml

sonar_scan:
  stage: sonar
  script:
    - mvn sonar:sonar -Dsonar.host.url=$SONAR_HOST -Dsonar.token=$SONAR_TOKEN

docker_build:
  stage: docker
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

deploy_staging:
  stage: deploy
  script:
    - aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment
  environment:
    name: staging
  only:
    - develop

deploy_production:
  stage: deploy
  script:
    - aws ecs update-service --cluster prod-cluster --service my-service --force-new-deployment
  environment:
    name: production
  only:
    - main
  when: manual  # Requires manual approval
```

---

### 40. What is Docker? Difference between an Image and a Container?
**Answer:**
| Concept     | Image                                    | Container                                |
| ----------- | ---------------------------------------- | ---------------------------------------- |
| What        | Blueprint / template (read-only)         | Running instance of an image             |
| Analogy     | Class                                    | Object                                   |
| State       | Immutable                                | Mutable (has writable layer)             |
| Created by  | `docker build`                           | `docker run`                             |

```bash
# Build image
docker build -t myapp:latest .

# Run container
docker run -d -p 8080:8080 --name myapp-container myapp:latest

# List running containers
docker ps

# View logs
docker logs myapp-container
```

---

### 41. What is Kubernetes (K8s)? How does it relate to your deployment?
**Answer:**
Kubernetes orchestrates containerized applications — automating deployment, scaling, and management.

**Key concepts:**
- **Pod:** Smallest deployable unit. Contains one or more containers.
- **Deployment:** Manages desired state (replicas, image version). Supports rolling updates.
- **Service:** Stable network endpoint to access Pods (ClusterIP, NodePort, LoadBalancer).
- **ConfigMap / Secret:** Externalized configuration and sensitive data.
- **HPA (Horizontal Pod Autoscaler):** Scales pods based on CPU/memory.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    spec:
      containers:
        - name: myapp
          image: myregistry/myapp:latest
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod"
```

---

### 42. What are GitLab CI/CD variables, artifacts, and caching?
**Answer:**
- **Variables:** Environment variables available in pipeline jobs. Can be defined at project/group level or in `.gitlab-ci.yml`. Masked variables hide secrets in logs.
- **Artifacts:** Files produced by a job and passed to subsequent stages (e.g., JAR files, test reports).
- **Cache:** Reusable dependencies across pipeline runs (e.g., Maven `.m2` folder) to speed up builds.

```yaml
variables:
  DEPLOY_ENV: "staging"

build:
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - .m2/repository    # Cache Maven dependencies
  artifacts:
    paths:
      - target/*.jar      # Pass JAR to next stage
    expire_in: 1 hour
```

---

## 🔥 DSA / CODING (Frequently Asked in Online Assessment)

---

### 43. Write a program to find two numbers in an array that add up to a target (Two Sum).
**Answer:**
```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[] { map.get(complement), i };
        }
        map.put(nums[i], i);
    }
    throw new IllegalArgumentException("No two sum solution");
}
```
**Time:** O(n) | **Space:** O(n)

---

### 44. Write a program to find the middle element of a LinkedList and detect a loop.
**Answer:**
Both use **Floyd's Tortoise and Hare** (fast/slow pointer).
```java
// Find middle
public ListNode findMiddle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}

// Detect loop
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

---

### 45. Write a program to group anagrams from a list of words.
**Answer:**
```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
// Input:  ["eat","tea","tan","ate","nat","bat"]
// Output: [["eat","tea","ate"],["tan","nat"],["bat"]]
```
**Time:** O(N × K log K)

---

### 46. Sort an array without using built-in sorting.
**Answer:**
```java
public void bubbleSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}
```

---

## 🔥 DESIGN PATTERNS & MISCELLANEOUS

---

### 47. What is a Singleton Design Pattern?
**Answer:**
Ensures a class has only one instance and provides a global access point.
```java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
In Spring, all beans are Singletons by default.

---

### 48. Explain REST API best practices. Difference between GET, POST, PUT, PATCH, DELETE.
**Answer:**
| Method   | Purpose         | Idempotent | Request Body |
| -------- | --------------- | ---------- | ------------ |
| `GET`    | Retrieve data   | Yes        | No           |
| `POST`   | Create resource | No         | Yes          |
| `PUT`    | Full update     | Yes        | Yes          |
| `PATCH`  | Partial update  | Yes        | Yes          |
| `DELETE` | Delete resource | Yes        | No           |

**Best Practices:**
- Use nouns for URIs: `/api/users`, not `/api/getUsers`.
- Use proper HTTP status codes: `200`, `201`, `204`, `400`, `404`, `500`.
- Version APIs: `/api/v1/users`.
- Use pagination for lists: `?page=1&size=20`.
- Handle errors consistently with a standard error response body.

---

### 49. What is the difference between `@RestController` and `@Controller`?
**Answer:**
- **`@Controller`:** Returns a **view** (HTML page via template engine like Thymeleaf). Methods return view names.
- **`@RestController`:** `@Controller` + `@ResponseBody`. Returns **data** (JSON/XML) directly. Every method's return value is serialized to the response body.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }
}
```

---

### 50. How do you handle database migrations in a Spring Boot + PostgreSQL project?
**Answer:**
Use **Flyway** or **Liquibase** for version-controlled database migrations.

**Flyway (preferred for SQL-based migrations):**
- Place SQL files in `src/main/resources/db/migration/`.
- Naming convention: `V1__create_users_table.sql`, `V2__add_email_column.sql`.
- Flyway auto-runs on startup, applies pending migrations in order.

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- V2__add_department_id.sql
ALTER TABLE users ADD COLUMN department_id BIGINT REFERENCES departments(id);
```

```yaml
# application.yml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
```
