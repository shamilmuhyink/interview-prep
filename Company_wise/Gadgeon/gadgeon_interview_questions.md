# Gadgeon - Senior Java Full Stack Developer Interview Questions

> **Focus:** Core Java, Spring Boot Microservices, IoT Integration, Performance, and DSA.
> **Format:** Ordered by frequency.

---

### Q1. 🔴 [🏢] Explain the internal working of `HashMap` and how it differs from `ConcurrentHashMap`.
**Answer:**
- **HashMap:** Uses an array of nodes based on hashing. On collision, entries are chained via a Linked List, which converts to a Red-Black tree after 8 collisions in Java 8+. It is **not** thread-safe.
- **ConcurrentHashMap:** Built for concurrent access without locking the entire map. In Java 8+, it uses CAS operations and synchronizes only at the bucket head level.

| Feature               | HashMap                     | ConcurrentHashMap               |
|-----------------------|-----------------------------|---------------------------------|
| Thread-safe           | ❌ No                        | ✅ Yes                           |
| Null Keys/Values      | 1 Null Key, Multiple Values | ❌ No Nulls allowed              |
| Locking               | None                        | Bucket-level (Node head)        |

### Q2. 🔴 [🏢] What is the difference between `map()` and `flatMap()` in Java Streams?
**Answer:**
- **`map()`:** One-to-one transformation. Converts each element into exactly one new element.
- **`flatMap()`:** One-to-many transformation. Maps an element to a stream of elements, then flattens all nested streams into a single stream.

💻 **Production-quality code snippet:**
```java
List<List<String>> nestedList = Arrays.asList(
    Arrays.asList("A", "B"), Arrays.asList("C", "D")
);
// flatMap un-nests the collections
List<String> flat = nestedList.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList()); // ["A", "B", "C", "D"]
```

### Q3. 🔴 [🏢] Explain Spring Boot Auto-configuration internals.
**Answer:**
- **Trigger:** Initiated by the `@EnableAutoConfiguration` annotation inside `@SpringBootApplication`.
- **Mechanism:** Spring Boot checks the `META-INF/spring.factories` (or `org.springframework.boot.autoconfigure.AutoConfiguration.imports` in Boot 3.0+) to load auto-configuration classes.
- **Conditions:** Uses `@Conditional` annotations (e.g., `@ConditionalOnClass`, `@ConditionalOnMissingBean`) to intelligently register default beans only if custom configurations are absent.

### Q4. 🔴 [🏢] What is the Spring Bean Lifecycle?
**Answer:**
- **Instantiation:** The container creates the bean object.
- **Populate Properties:** Dependency Injection occurs.
- **Initialization:** Aware interfaces are invoked, followed by custom init methods (like `@PostConstruct`).
- **Destruction:** On application shutdown, cleanup methods (like `@PreDestroy`) execute.

### Q5. 🔴 [🏢] What is a Circuit Breaker and how does Resilience4j implement it?
**Answer:**
- **Purpose:** Protects microservices from cascading failures.
- **States:**
  - **CLOSED:** Normal operation.
  - **OPEN:** Calls fail fast without hitting the downstream service.
  - **HALF-OPEN:** Allows a limited number of test requests to check if the downstream service has recovered.

### Q6. 🟠 [🏢] How do you handle IoT device data ingestion at scale in Java? (MQTT Integration)
**Answer:**
- **Protocol:** IoT devices typically use lightweight protocols like MQTT rather than HTTP.
- **Integration:** Use Spring Integration MQTT or Eclipse Paho to subscribe to broker topics.
- **Processing:** Ingested payloads are validated, transformed, and often published to Kafka for durable, high-throughput asynchronous processing by downstream microservices.

### Q7. 🟠 [🏢] Explain the differences between Monolithic and Microservices architectures.
**Answer:**
- **Monolithic:** Bundles all features into a single deployable unit. Easy to develop initially but hard to scale and maintain.
- **Microservices:** Decomposes features into independent services. Allows polyglot persistence and independent scaling, but introduces distributed system complexity.

| Feature         | Monolithic | Microservices |
|-----------------|------------|---------------|
| Deployment      | Single unit | Independent deployable units |
| Scaling         | Scale entire app | Scale specific services |
| Data            | Shared database | Database per service |

### Q8. 🟠 [🏢] Two Sum Problem.
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(N)**
- **Approach:** Use a `HashMap` to store numbers and their indices. During iteration, check if `target - current` exists in the map to find the pair in O(1) time.

💻 **Code Snippet (Optimized):**
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
    throw new IllegalArgumentException("No solution");
}
```

### Q9. 🟠 [🏢] Reverse a Linked List.
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(1)**
- **Approach:** Iterate through the list using three pointers (`prev`, `current`, `next`) to flip the `next` references.

💻 **Code Snippet (Optimized):**
```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode current = head;
    while (current != null) {
        ListNode nextTemp = current.next;
        current.next = prev;
        prev = current;
        current = nextTemp;
    }
    return prev;
}
```

### Q10. 🟠 [🏢] Docker: Image vs Container.
**Answer:**
| Concept        | Description |
|----------------|-------------|
| **Image**      | A read-only, immutable template containing the OS, runtime, app code, and libraries. |
| **Container**  | A live, runnable, isolated instance of the image. |

### Q11. 🟡 [🏢] Explain the internals of ThreadPoolExecutor.
**Answer:**
- **Core Pool Size:** The minimum number of threads kept alive.
- **Max Pool Size:** The absolute maximum number of threads allowed.
- **Work Queue:** A blocking queue that holds tasks waiting to be executed if all core threads are busy.
- **Keep-Alive Time:** If thread count exceeds core size, idle threads are terminated after this duration.

### Q12. 🟡 [🏢] CompletableFuture vs parallelStream.
**Answer:**
- **`parallelStream()`:** Best for CPU-intensive tasks. It uses the common ForkJoinPool, meaning a blocking I/O task in one stream can stall all other parallel streams in the JVM.
- **`CompletableFuture`:** Best for I/O-intensive tasks (network calls, DB queries). Allows specifying a custom `Executor` to avoid blocking the common pool and enables complex task chaining.

### Q13. 🟡 [🏢] Explain Database Indexing (B-Tree).
**Answer:**
- **Concept:** An index is a distinct data structure (usually a B-Tree) that keeps a specific column logically sorted.
- **Benefit:** Allows database engines to locate records in **O(log N)** time rather than performing a full table scan (O(N)).

### Q14. 🟡 [🏢] What is an API Gateway?
**Answer:**
- **Purpose:** The single entry point for all client requests entering a microservices ecosystem.
- **Features:** Handles cross-cutting concerns like dynamic routing, JWT validation, rate limiting, and CORS.

### Q15. 🟢 [🏢] Explain the JWT authentication flow.
**Answer:**
- **Login:** Client sends credentials. Server validates and signs a JWT (JSON Web Token), returning it to the client.
- **Storage:** Client securely stores the token (e.g., HttpOnly Cookie or LocalStorage).
- **Subsequent Requests:** Token is attached to the `Authorization: Bearer <token>` header.
- **Validation:** Server intercepts the request and verifies the token's cryptographic signature statelessly.

### Q16. 🟠 [🏢] Explain Garbage Collection in Java (G1, ZGC).
**Answer:**
- **Goal:** Automatically reclaims memory by identifying objects that are no longer reachable from GC roots.
- **G1 (Garbage First):** The default GC since Java 9. It partitions the heap into regions and prioritizes sweeping regions with the most garbage. Good for high throughput and predictable pause times.
- **ZGC:** A scalable, low-latency garbage collector. Pause times do not exceed a few milliseconds, even with terabyte-sized heaps.

### Q17. 🟠 [🏢] How do you implement a thread-safe Singleton Pattern?
**Answer:**
- **Approach:** The safest and most efficient way is using Double-Checked Locking or an `enum`.
- **Mechanism:** Double-checked locking prevents synchronization overhead on every call by only locking when the instance is null.

💻 **Code Snippet (Optimized):**
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

### Q18. 🟡 [🏢] What is the difference between `volatile` and `synchronized`?
**Answer:**
| Feature | `volatile` | `synchronized` |
|---------|------------|----------------|
| Lock | No locking | Acquires lock (Mutex) |
| Atomicity | ❌ No (except read/write) | ✅ Yes |
| Visibility | ✅ Yes (Reads from main memory) | ✅ Yes |
| Use Case | Status flags | Critical sections |

### Q19. 🟡 [🏢] Explain `@Transactional` propagation and isolation levels.
**Answer:**
- **Propagation:** Defines how transactions relate to each other.
  - `REQUIRED` (Default): Join existing or create a new one.
  - `REQUIRES_NEW`: Always create a new transaction, suspending the existing one.
- **Isolation:** Controls how changes made by one transaction become visible to others.
  - `READ_COMMITTED` (Typical default): Prevents dirty reads.
  - `SERIALIZABLE`: Highest level, prevents phantom reads but reduces concurrency.

### Q20. 🟡 [🏢] What are Angular Lifecycle Hooks?
**Answer:**
- **`ngOnChanges`:** Called when an input binding value changes.
- **`ngOnInit`:** Called once after the first `ngOnChanges`. Used for component initialization and API calls.
- **`ngOnDestroy`:** Called just before the component is destroyed. Used to unsubscribe from Observables to prevent memory leaks.
