# Synechron Interview Questions (Senior Java Full Stack Developer)

> **Source:** Aggregated from Glassdoor, AmbitionBox, GeeksforGeeks, PrepInsta, and Reddit.
> **Focus:** Deep core Java, Concurrency, Microservices Resiliency, Spring Boot internals, and Problem Solving.
> **Format:** Questions are ordered by frequency based on recent interview experiences.

---

### Q1. How do you find the first non-repeating character in a String using Java 8 Streams?
**Answer:**
```java
String input = "synechron";
Character firstNonRepeating = input.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(Function.identity(), LinkedHashMap::new, Collectors.counting()))
    .entrySet().stream()
    .filter(e -> e.getValue() == 1L)
    .map(Map.Entry::getKey)
    .findFirst()
    .orElse(null);
System.out.println(firstNonRepeating);
```

### Q2. How do you identify, diagnose, and resolve a memory leak in a Java application?
**Answer:**
1. **Identify:** Monitor the application using tools like Prometheus/Grafana or JConsole. A classic sign is the Old Generation heap space continuously growing after Full GCs, eventually leading to `OutOfMemoryError: Java heap space`.
2. **Diagnose:** Capture a Heap Dump (using `jmap -dump:live,format=b,file=heap.bin <pid>` or `-XX:+HeapDumpOnOutOfMemoryError`).
3. **Analyze:** Open the dump in Eclipse MAT (Memory Analyzer Tool) or VisualVM. Look at the "Dominator Tree" or "Leak Suspects" report to find which objects are holding onto the most memory and tracing their GC Roots.
4. **Resolve:** Common causes include unclosed resources (DB connections, streams), static collections growing indefinitely, or `ThreadLocal` variables not being cleaned up in thread pools.

### Q3. How do you implement a custom LRU (Least Recently Used) Cache?
**Answer:**
The most efficient way to implement an LRU cache in Java is by using a `HashMap` combined with a `Doubly Linked List`. The HashMap provides $O(1)$ access time, and the Doubly Linked List allows $O(1)$ additions and removals to keep track of the most/least recently used items.
Alternatively, Java provides a built-in way using `LinkedHashMap` by overriding the `removeEldestEntry` method:
```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        // initialCapacity, loadFactor, accessOrder (true for LRU)
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

### Q4. How can the Singleton pattern be broken, and how do you prevent it?
**Answer:**
1. **Reflection:** Can access the private constructor and create a new instance.
   *Prevention:* Throw an exception in the constructor if the instance already exists, or use an `Enum` for Singleton.
2. **Serialization:** Deserializing an object creates a new instance.
   *Prevention:* Implement the `readResolve()` method to return the existing instance.
3. **Cloning:** Calling `clone()` creates a copy.
   *Prevention:* Override `clone()` and throw `CloneNotSupportedException`.
4. **Multiple ClassLoaders:** Different classloaders can load the same class twice.
   *Prevention:* Ensure the Singleton is loaded by a common parent classloader.

### Q5. What is `BeanCurrentlyInCreationException` and how do you resolve circular dependencies in Spring Boot?
**Answer:**
This exception occurs when two or more beans depend on each other, creating a circular reference (e.g., Bean A requires Bean B, but Bean B requires Bean A in their constructors). Spring cannot decide which one to instantiate first.
**Resolution:**
1. **Refactor Design:** The best solution is to redesign the classes. Extract the common functionality into a third bean (Bean C) that both A and B can inject.
2. **Use `@Lazy`:** Annotate one of the constructor parameters with `@Lazy`. Spring will inject a proxy instead of the actual bean and delay the initialization until it's first used.
3. **Setter/Field Injection:** Instead of constructor injection, use setter injection, allowing Spring to instantiate the beans first and wire them later (though constructor injection is always preferred for immutability).

### Q6. How do you implement resiliency in Microservices? Explain Circuit Breakers and Timeouts.
**Answer:**
When Service A calls Service B, and Service B is down or extremely slow, Service A's threads will block waiting for a response, eventually bringing down Service A as well (Cascading Failure).
- **Timeouts:** Ensure every external HTTP call (using RestTemplate, WebClient, or Feign) has a strict read and connection timeout configured.
- **Circuit Breaker:** We use a library like **Resilience4j**. If the failure rate of Service B exceeds a threshold (e.g., 50% over 10 calls), the circuit "opens". Service A immediately returns a fallback response or throws an exception without even trying to call Service B, allowing Service B time to recover. After a wait duration, it enters a "half-open" state to test if Service B is back online.

### Q7. Explain the purpose of `default` and `static` methods in Java 8 Interfaces.
**Answer:**
- **`default` methods:** Allow adding new methods to interfaces without breaking the existing classes that implement the interface (backward compatibility). For example, `forEach` was added to the `Iterable` interface.
- **`static` methods:** Belong to the interface class itself, not the implementing object. They are used for helper/utility methods specific to the interface (e.g., `Stream.of()`).

### Q8. How do you detect and resolve deadlocks in Java?
**Answer:**
A deadlock occurs when two or more threads are blocked forever, waiting for each other to release locks.
- **Detection:** If the application hangs, take a Thread Dump (using `jstack <pid>`). At the bottom of the dump, the JVM will explicitly print "Found one Java-level deadlock" along with the threads involved and the monitors (locks) they are holding and waiting for.
- **Resolution/Prevention:**
  1. Always acquire locks in the exact same predefined order across all threads.
  2. Use `tryLock(timeout)` from the `java.util.concurrent.locks.Lock` interface instead of intrinsic `synchronized` blocks. If a thread can't acquire all needed locks within the timeout, it backs off, releases its current locks, and tries again.
  3. Keep synchronized blocks as short as possible.

### Q9. How do you implement environment-specific configurations in Spring Boot?
**Answer:**
We use Spring Profiles. By defining properties files like `application-dev.yml`, `application-qa.yml`, and `application-prod.yml`, we can separate configurations (DB URLs, credentials). We activate a profile using `-Dspring.profiles.active=prod` as a JVM argument or by setting the `SPRING_PROFILES_ACTIVE` environment variable in our deployment pipeline.

### Q10. Explain the architecture of Kafka and how it ensures high availability.
**Answer:**
Kafka uses a distributed architecture consisting of Brokers, Topics, Partitions, and ZooKeeper (or KRaft). Data is split into **Partitions** across multiple brokers for scalability. High availability is achieved through **Replication**. Each partition has a designated *Leader* and one or more *Followers*. Producers write to the Leader, which replicates the data to the Followers. If a Leader broker goes down, Kafka automatically elects a synchronized Follower as the new Leader, ensuring no data loss.

### Q11. What is a Docker Multi-stage build and why is it useful for Java applications?
**Answer:**
A multi-stage build uses multiple `FROM` statements in a single Dockerfile.
1. The first stage (the "builder") uses a heavy base image with the JDK and Maven/Gradle to compile the code and build the JAR file.
2. The second stage uses a much smaller, lightweight JRE-only base image (like Alpine). It copies *only* the compiled JAR from the first stage.
**Benefit:** It drastically reduces the final image size, improving deployment speed and reducing the security attack surface since build tools are not included in the production image.

### Q12. What is the difference between `volatile` and `synchronized` in Java?
**Answer:**
- `volatile`: Ensures visibility. It tells the JVM that the variable's value will be modified by different threads, so it should never be cached thread-locally and always read from main memory. It does not provide atomicity or mutual exclusion.
- `synchronized`: Ensures both visibility and mutual exclusion. Only one thread can execute a synchronized block/method at a time, making compound actions (like `count++`) safe.

### Q13. How do you solve the N+1 Query Problem in Hibernate/JPA?
**Answer:**
The N+1 problem occurs when an entity with a one-to-many relationship is loaded, and then the related entities are accessed in a loop, causing 1 query for the parent and N queries for the children.
**Solutions:**
1. Use `JOIN FETCH` in JPQL/HQL to load the parent and children in a single query.
2. Use `@EntityGraph` in Spring Data JPA to define which associations to fetch eagerly.
3. Use Hibernate-specific `@BatchSize` annotation to fetch children in batches (e.g., IN clauses) rather than individually.

### Q14. Explain `@Transactional` propagation levels in Spring Boot.
**Answer:**
Propagation determines how a transaction behaves when a transactional method calls another transactional method.
- `REQUIRED` (Default): Joins the existing transaction if one exists; otherwise, creates a new one.
- `REQUIRES_NEW`: Always suspends the current transaction and creates a new, independent one.
- `NESTED`: Executes within a nested transaction (using savepoints) if one exists.
- `MANDATORY`: Requires an existing transaction; throws an exception if none exists.
- `SUPPORTS`: Executes within a transaction if one exists, but does not create one if it doesn't.

### Q15. Why do we need an API Gateway in a Microservices architecture?
**Answer:**
An API Gateway acts as the single entry point for all clients into the microservices ecosystem. It provides:
1. **Routing & Load Balancing**: Directs requests to the appropriate backend service.
2. **Security**: Handles centralized authentication/authorization (e.g., verifying JWTs), preventing each microservice from implementing it.
3. **Cross-Cutting Concerns**: Handles rate limiting, CORS, logging, and SSL termination.
4. **Aggregation**: Can aggregate responses from multiple microservices into a single response to reduce client round-trips.

### Q16. In Angular, what is the difference between an Observable and a Promise?
**Answer:**
- **Promises**: Emit a single value, execute immediately (eager), and cannot be cancelled.
- **Observables**: Can emit multiple values over time (stream), are lazy (only execute when subscribed to), and can be cancelled via `unsubscribe()`. They also support powerful operators from RxJS like `map`, `filter`, and `switchMap`.

### Q17. Explain Angular Component Lifecycle Hooks.
**Answer:**
- `ngOnChanges()`: Called when an input binding (`@Input()`) value changes.
- `ngOnInit()`: Called once after the first `ngOnChanges`. Ideal for component initialization and fetching data.
- `ngDoCheck()`: Called during every change detection run.
- `ngAfterViewInit()`: Called after the component's views and child views have been initialized.
- `ngOnDestroy()`: Called just before the component is destroyed. Ideal for cleaning up subscriptions to avoid memory leaks.

### Q18. Given an array of integers and a target sum, return the indices of the two numbers that add up to the target (Two Sum).
**Answer:**
Use a HashMap to store the numbers and their indices as you iterate.
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

### Q19. How do you implement asynchronous programming in Java 8+?
**Answer:**
Using `CompletableFuture`. It allows you to run tasks asynchronously and chain them without blocking.
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // perform long task
    return "Result";
}).thenApply(result -> {
    // transform result
    return result.toUpperCase();
});
```

### Q20. What is the difference between `wait()` and `sleep()`?
**Answer:**
- `wait()`: A method of the `Object` class. It releases the lock/monitor it holds and must be called from a synchronized context. Used for inter-thread communication.
- `sleep()`: A static method of the `Thread` class. It pauses thread execution for a specific time but **does not** release any locks it holds.
