# Synechron Interview Questions (Senior Java Full Stack Developer)

> **Source:** Aggregated from Glassdoor, AmbitionBox, GeeksforGeeks, PrepInsta, and Reddit.
> **Focus:** Deep core Java, Concurrency, Microservices Resiliency, Spring Boot internals, and Problem Solving.
> **Format:** Questions are ordered by frequency based on recent interview experiences.

---

### Q1. 🔴 [🌐] Explain the internal working of `HashMap` and how it differs from `ConcurrentHashMap`.
**Answer:**
- **HashMap:** Uses an array of nodes. On collision, entries are chained via a Linked List, which converts to a Red-Black tree after 8 collisions (Java 8+). It is **not** thread-safe.
- **ConcurrentHashMap:** Built for concurrent access. In Java 8+, it uses CAS operations and synchronizes only at the bucket head level (lock striping), allowing high concurrency.

| Feature | HashMap | ConcurrentHashMap |
|---------|---------|-------------------|
| Thread-safe | ❌ No | ✅ Yes |
| Null Keys/Values | 1 Null Key, Multiple Values | ❌ No Nulls allowed |
| Locking | None | Bucket-level (Node head) |

### Q2. 🔴 [🌐] How do you find the first non-repeating character in a String using Java 8 Streams?
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(N)**
- **Approach:** Map characters to their frequencies using `Collectors.groupingBy` with a `LinkedHashMap` (to preserve insertion order). Then filter for frequency == 1 and get the first element.

💻 **Production-quality code snippet:**
```java
public Character firstNonRepeating(String input) {
    return input.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(Function.identity(), LinkedHashMap::new, Collectors.counting()))
        .entrySet().stream()
        .filter(e -> e.getValue() == 1L)
        .map(Map.Entry::getKey)
        .findFirst()
        .orElse(null);
}
```

### Q3. 🔴 [🌐] How do you identify, diagnose, and resolve a memory leak in a Java application?
**Answer:**
- **Identify:** Monitor using APM tools (Prometheus/Grafana) or JConsole. A classic sign is Old Generation heap space continuously growing after Full GCs, leading to `OutOfMemoryError`.
- **Diagnose:** Capture a Heap Dump using `jmap -dump:live,format=b,file=heap.bin <pid>` or the `-XX:+HeapDumpOnOutOfMemoryError` JVM flag.
- **Analyze:** Open the dump in Eclipse MAT or VisualVM. Use the "Dominator Tree" to find objects holding the most memory and trace their GC Roots.
- **Resolve:** Common culprits include unclosed resources (DB connections, streams), static collections growing indefinitely, or `ThreadLocal` variables not cleaned up in thread pools.

### Q4. 🔴 [🌐] Explain Garbage Collection in Java (G1, ZGC) and JVM Tuning.
**Answer:**
- **Goal:** Automatically reclaims memory by identifying objects unreachable from GC roots.
- **G1 (Garbage First):** Default GC since Java 9. Partitions the heap into regions and prioritizes sweeping regions with the most garbage. Good for high throughput and predictable pause times.
- **ZGC:** Scalable, low-latency garbage collector. Pause times do not exceed a few milliseconds, even with terabyte-sized heaps.
- **Tuning:** Use `-Xms` and `-Xmx` to set heap size. Analyze GC logs to ensure time spent in GC is < 5% of total application time.

### Q5. 🔴 [🌐] What is a Circuit Breaker and how does Resilience4j implement it?
**Answer:**
- **Purpose:** Protects microservices from cascading failures when downstream services are slow or down.
- **Mechanism:** Monitors failure rates and timeouts.
- **States:**
  - **CLOSED:** Normal operation; requests flow freely.
  - **OPEN:** Calls fail fast without hitting the downstream service.
  - **HALF-OPEN:** Allows a limited number of test requests to check if the downstream service has recovered.

### Q6. 🔴 [🌐] How can the Singleton pattern be broken, and how do you prevent it?
**Answer:**
- **Reflection:** Can access private constructors.
  - *Prevention:* Throw an exception in the constructor if instance != null, or use an `enum`.
- **Serialization:** Deserializing creates a new instance.
  - *Prevention:* Implement the `readResolve()` method to return the existing instance.
- **Cloning:** Calling `clone()` creates a copy.
  - *Prevention:* Override `clone()` and throw `CloneNotSupportedException`.

### Q7. 🔴 [🌐] Explain Database Indexing (B-Tree) and Covering Indexes.
**Answer:**
- **Concept:** An index is a data structure (usually B-Tree) that keeps a column logically sorted, allowing **O(log N)** lookups instead of **O(N)** full table scans.
- **Covering Index:** An index that contains all the columns needed for a specific query, meaning the database engine doesn't even need to look up the actual table rows.

### Q8. 🔴 [🏢] Explain the Saga Pattern for distributed transactions.
**Answer:**
- **Problem:** Two-Phase Commit (2PC) does not scale well in microservices (blocks resources, tight coupling).
- **Solution:** A Saga is a sequence of local transactions. Each local transaction updates the database and publishes a message/event to trigger the next local transaction in the saga.
- **Rollback:** If a local transaction fails, the saga executes **compensating transactions** to undo the changes made by the preceding local transactions.

### Q9. 🔴 [🌐] What is the N+1 Query Problem in Hibernate/JPA and how do you solve it?
**Answer:**
- **Problem:** When loading an entity with a one-to-many relationship, accessing the children in a loop triggers 1 query for the parent and N queries for the children.
- **Solutions:**
  1. Use `JOIN FETCH` in JPQL/HQL to load parent and children in a single query.
  2. Use `@EntityGraph` in Spring Data JPA to define associations to fetch eagerly.
  3. Use Hibernate's `@BatchSize` annotation to fetch children in batches (e.g., `IN` clauses).

### Q10. 🟠 [🌐] How do you implement a custom LRU (Least Recently Used) Cache?
**Answer:**
- **Complexity:** ⚡ Time: **O(1)** | Space: **O(N)**
- **Approach:** Use a `HashMap` combined with a `Doubly Linked List`. The HashMap provides O(1) access, while the DLL manages the most/least recently used items via O(1) removals and additions at the ends.

💻 **Production-quality code snippet (using LinkedHashMap):**
```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // true for access-order
        this.capacity = capacity;
    }
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

### Q11. 🟠 [🌐] Explain `@Transactional` propagation levels in Spring Boot.
**Answer:**
- **Propagation:** Determines how a transaction behaves when called from another transactional method.
- `REQUIRED` (Default): Joins the existing transaction if one exists; otherwise creates a new one.
- `REQUIRES_NEW`: Always suspends the current transaction and creates a new, independent one.
- `NESTED`: Executes within a nested transaction (using DB savepoints) if one exists.
- `MANDATORY`: Requires an existing transaction; throws an exception if none exists.

### Q12. 🟠 [🌐] What is `BeanCurrentlyInCreationException` and how do you resolve circular dependencies?
**Answer:**
- **Problem:** Bean A requires Bean B, but Bean B requires Bean A in their constructors. Spring cannot decide which to instantiate first.
- **Resolutions:**
  1. **Refactor Design (Best):** Extract the common functionality into a third Bean C.
  2. **`@Lazy` Injection:** Annotate one constructor parameter with `@Lazy`. Spring injects a proxy and delays initialization until first use.
  3. **Setter Injection:** Allows Spring to instantiate both beans first, then wire them. (Note: Constructor injection is preferred).

### Q13. 🟠 [🌐] How do you detect and resolve deadlocks in Java?
**Answer:**
- **Detection:** Take a Thread Dump (`jstack <pid>`). The JVM will explicitly state "Found one Java-level deadlock" along with the involved threads and locks.
- **Resolution/Prevention:**
  1. Always acquire locks in a strict, predefined global order.
  2. Use `Lock.tryLock(timeout)` from `java.util.concurrent.locks` instead of `synchronized` blocks. If a thread fails to acquire the lock within the timeout, it backs off and releases held locks.
  3. Keep synchronized critical sections as small as possible.

### Q14. 🟠 [🌐] What is the difference between `volatile` and `synchronized`?
**Answer:**
| Feature | `volatile` | `synchronized` |
|---------|------------|----------------|
| Lock | No locking | Acquires lock (Mutex) |
| Atomicity | ❌ No (except read/write) | ✅ Yes |
| Visibility | ✅ Yes (Reads from main memory) | ✅ Yes |
| Use Case | Status flags | Critical sections |

### Q15. 🟠 [🌐] Explain the architecture of Kafka and how it ensures high availability.
**Answer:**
- **Architecture:** Consists of Brokers, Topics, Partitions, and ZooKeeper/KRaft. Data is split into **Partitions** across multiple brokers for scalability.
- **High Availability (Replication):** Each partition has a designated *Leader* and one or more *Followers*. Producers write to the Leader, which replicates to Followers. If the Leader goes down, Kafka automatically elects an in-sync Follower as the new Leader, ensuring zero data loss.

### Q16. 🟠 [🌐] What are Virtual Threads in Java 21? How do they differ from Platform Threads?
**Answer:**
- **Platform Threads:** 1:1 mapping with OS threads. Expensive to create, consumes ~1MB memory per thread, leading to scalability limits (e.g., maxing out at a few thousand threads).
- **Virtual Threads:** Lightweight, user-mode threads managed by the JVM (M:N mapping). Millions can be created concurrently.
- **Use Case:** Excellent for heavily I/O-bound tasks (thread-per-request model) as they unblock OS threads during I/O wait times automatically without complex reactive frameworks.

### Q17. 🟡 [🌐] Spring Boot Actuator: What is it and how do you secure it?
**Answer:**
- **Purpose:** Exposes operational information about the running application (health, metrics, info, env, thread dumps).
- **Security:** Actuator endpoints contain sensitive data. They should be secured using Spring Security, restricted to admin roles, or completely hidden from public network interfaces (binding them to a different internal management port).

### Q18. 🟡 [🌐] Two Sum Problem.
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(N)**
- **Approach:** Use a `HashMap` to store numbers and indices. Check if `target - current` exists in the map to find the pair in O(1) time.

💻 **Code Snippet (Optimized):**
```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) return new int[] { map.get(complement), i };
        map.put(nums[i], i);
    }
    throw new IllegalArgumentException("No solution");
}
```

### Q19. 🟡 [🌐] Explain the purpose of `default` and `static` methods in Java 8 Interfaces.
**Answer:**
- **`default` methods:** Allow adding new methods to interfaces without breaking existing implementing classes (backward compatibility). e.g., `forEach` in `Iterable`.
- **`static` methods:** Belong to the interface itself. Used for helper/utility methods (e.g., `Stream.of()`).

### Q20. 🟡 [🌐] Why do we need an API Gateway in a Microservices architecture?
**Answer:**
- **Routing:** Directs client requests to the correct backend microservice.
- **Cross-Cutting Concerns:** Centralizes JWT validation, rate limiting, and CORS handling.
- **Aggregation:** Can combine responses from multiple microservices to reduce client round-trips.

### Q21. 🟡 [🌐] What is a Docker Multi-stage build and why is it useful?
**Answer:**
- **Concept:** Uses multiple `FROM` statements in a single Dockerfile. The first stage uses a heavy JDK image to compile the code. The second stage uses a tiny JRE-only image and copies just the compiled JAR.
- **Benefit:** Drastically reduces final image size and minimizes the security attack surface by excluding build tools from production.

### Q22. 🟢 [🌐] How do you implement asynchronous programming in Java 8+?
**Answer:**
- **`CompletableFuture`:** Allows running tasks asynchronously and chaining them without blocking. Best for I/O-intensive tasks because it can use a custom thread pool.
```java
CompletableFuture.supplyAsync(() -> fetchFromDb())
                 .thenApply(String::toUpperCase)
                 .thenAccept(System.out::println);
```

### Q23. 🟢 [🌐] What is the difference between `wait()` and `sleep()`?
**Answer:**
| Feature | `wait()` | `sleep()` |
|---------|----------|-----------|
| Class | `Object` | `Thread` |
| Lock Release | ✅ Yes | ❌ No |
| Wake up | via `notify()`/`notifyAll()` | Automatically after timeout |
| Context | Must be in `synchronized` block | Anywhere |

### Q24. 🟢 [🌐] In Angular, what is the difference between an Observable and a Promise?
**Answer:**
- **Promises:** Eager (executes immediately), emits a single value, cannot be cancelled.
- **Observables:** Lazy (executes only when subscribed), emits multiple values over time (stream), can be cancelled (`unsubscribe()`), and supports RxJS operators (`map`, `filter`, `switchMap`).

### Q25. 🟢 [🌐] Explain how JWT and Role-Based Access Control (RBAC) work together in Spring Security.
**Answer:**
- **Authentication (JWT):** The client logs in, and the server validates credentials. It generates a JWT containing user details and their specific roles (e.g., `["ROLE_USER", "ROLE_ADMIN"]`) in the payload claims, signing it with a secret key.
- **Authorization (RBAC):** On subsequent requests, a custom Spring Security filter intercepts the token. It extracts the roles from the claims and populates the `SecurityContext`.
- **Enforcement:** Endpoints are protected using annotations like `@PreAuthorize("hasRole('ADMIN')")`, which checks the `SecurityContext` before allowing execution.

### Q26. 🟢 [🌐] What is the difference between Level 1 (L1) and Level 2 (L2) cache in Hibernate?
**Answer:**
| Feature | L1 Cache (First-Level) | L2 Cache (Second-Level) |
|---------|------------------------|-------------------------|
| Scope | Session (EntityManager) | SessionFactory (Global) |
| Default | Enabled by default | Disabled by default (Needs Ehcache/Redis) |
| Sharing | Not shared across threads | Shared across all sessions |

### Q27. 🟢 [🌐] Explain the Entity Lifecycle states in JPA/Hibernate.
**Answer:**
- **Transient:** Object instantiated using `new` but not associated with a Session or database row.
- **Persistent:** Object is associated with a Session. Any changes made to it will be automatically flushed to the database.
- **Detached:** The Session is closed or cleared, but the object still has an ID. Changes are no longer tracked. (Can be merged back).
- **Removed:** Marked for deletion. Will be physically deleted from the database upon flush/commit.

### Q28. 🟢 [🌐] How do you handle authentication tokens across all outgoing requests in Angular?
**Answer:**
- **HTTP Interceptors:** Use an `HttpInterceptor`. It intercepts every outgoing HTTP request globally.
- **Implementation:** You clone the incoming request, attach the JWT to the `Authorization: Bearer <token>` header, and pass the cloned request to the next handler (`next.handle(request)`).

### Q29. 🟢 [🌐] What are Route Guards in Angular and how are they used for RBAC?
**Answer:**
- **Purpose:** Prevents unauthorized users from navigating to protected routes.
- **`CanActivate`:** A guard interface that returns `true` if navigation is allowed, or `false` (often redirecting to a login page) if not.
- **RBAC Check:** The guard injects an AuthService, decodes the JWT (or checks local state), verifies if the user has the required roles for the target route, and blocks/allows the navigation accordingly.

### Q30. 🟢 [🌐] When would you use Docker Compose versus Kubernetes?
**Answer:**
- **Docker Compose:** Ideal for local development or single-host deployments. It uses a `docker-compose.yml` to spin up your application, database, and cache containers simultaneously on a single machine.
- **Kubernetes (K8s):** Necessary for production orchestration across a cluster of multiple machines (nodes). Handles self-healing, automated rollouts/rollbacks, and horizontal auto-scaling, which Compose cannot do.

### Q31. 🟢 [🌐] Explain the different types of Kubernetes Services.
**Answer:**
| Service Type | Use Case | Accessibility |
|--------------|----------|---------------|
| **ClusterIP** | Backend-to-backend communication | Internal only |
| **NodePort** | Exposes a specific port on every Node | External (primitive) |
| **LoadBalancer** | Exposes the app to the internet | External (Provisions a Cloud LB) |

### Q32. 🟢 [🌐] How would you architect a basic 3-tier web application on AWS?
**Answer:**
- **Presentation Tier:** Angular front-end hosted on **Amazon S3** (as static website hosting) distributed via **CloudFront** (CDN) for low latency.
- **Logic Tier:** Spring Boot REST APIs packaged in Docker containers running on **Amazon ECS** or **EKS** (Kubernetes), sitting behind an **API Gateway** or Application Load Balancer (ALB).
- **Data Tier:** Relational data stored in **Amazon RDS** (PostgreSQL/MySQL) deployed in private subnets with a standby replica for high availability.

### Q33. 🟢 [🌐] How do you scale a relational database like PostgreSQL for heavy read traffic?
**Answer:**
- **Read Replicas:** Create asynchronous read replicas. Route all `SELECT` queries to the replicas while keeping the primary instance dedicated to `INSERT/UPDATE/DELETE` operations.
- **Connection Pooling:** Use PgBouncer or HikariCP to limit the number of active connections preventing DB exhaustion.
- **Materialized Views:** For complex, frequently requested aggregations, use materialized views to pre-compute the results.

### Q34. 🟢 [🌐] Describe a typical CI/CD pipeline using GitHub Actions.
**Answer:**
- **Continuous Integration (CI):** Triggered on PR creation. Checks out code -> Runs Maven/Gradle build -> Executes Unit & Integration Tests -> Runs SonarQube for code quality.
- **Continuous Deployment (CD):** Triggered on merge to `main`. Builds the Docker image -> Pushes to a registry (DockerHub/ECR) -> Triggers an update to Kubernetes (e.g., updating the deployment manifest) to roll out the new image.

### Q35. 🟢 [🌐] What makes a REST API "Idempotent" and which HTTP methods guarantee this?
**Answer:**
- **Idempotency:** Making multiple identical requests produces the exact same result on the server as making a single request. Safely handles network retries.
- **Idempotent Methods:** `GET`, `PUT`, `DELETE`.
- **Non-Idempotent Methods:** `POST` (creates a new resource every time).
