# Module 1: Language (Core Java)

> **Scope:** Multithreading, JVM Internals, Collections Framework, Functional Programming, Memory Model
> **Questions:** 20 | **Critical:** 5 | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

---

### Q1. 🔴 🌐 What is the difference between `HashMap` and `ConcurrentHashMap`? How does `ConcurrentHashMap` achieve thread safety internally?

**`ConcurrentHashMap` replaces full-map locking with fine-grained, lock-free concurrency using CAS operations and synchronized bins, allowing multiple threads to read/write concurrently without blocking each other.**

**Core Concept:**
- `HashMap` is not thread-safe. Concurrent modification causes infinite loops (Java 7 linked list cycle) or data corruption (Java 8+).
- `ConcurrentHashMap` (Java 8+) uses a combination of CAS (Compare-And-Swap), `synchronized` on individual bins (buckets), and volatile reads.

**Internal Mechanics (Java 8+):**
1. **No Segment Locking** — unlike Java 7, there are no `Segment` objects.
2. **Node array is volatile** — reads are lock-free via `Unsafe.getObjectVolatile()`.
3. **Insertion** — if the bin is empty, CAS is used to place the node. If non-empty, `synchronized` on the head node of that bin.
4. **Reads never block** — `get()` uses volatile reads, no locking.
5. **Tree bins** — when a bin exceeds 8 nodes (and table ≥ 64), it converts to a red-black tree, same as `HashMap`.

**Key Differences:**

| Feature | HashMap | ConcurrentHashMap |
|---------|---------|-------------------|
| Thread safety | None | Yes (lock-free reads, bin-level locks for writes) |
| Null keys/values | Allows 1 null key, null values | **Neither allowed** |
| Iterator | Fail-fast (`ConcurrentModificationException`) | Weakly consistent (no exception) |
| Performance (single-thread) | Faster | Slight overhead |
| `size()` accuracy | Exact | Approximate (uses `baseCount` + `CounterCell[]`) |

```java
// Production pattern: computing a value atomically
ConcurrentHashMap<String, LongAdder> metrics = new ConcurrentHashMap<>();

// Atomic compute — no external synchronization needed
metrics.computeIfAbsent("api.latency", k -> new LongAdder()).increment();

// WRONG: This is NOT atomic even with ConcurrentHashMap
// if (!map.containsKey(key)) map.put(key, value); // Race condition!
```

**⚠️ Pitfalls:**
- `size()` returns an *estimate* under contention — use `mappingCount()` for long-valued counts.
- Compound operations (`check-then-act`) are NOT atomic; use `compute()`, `merge()`, or `putIfAbsent()`.
- `Collections.synchronizedMap(new HashMap<>())` is a poor alternative — it synchronizes on the entire map.

---

### Q2. 🔴 🏢 Explain the Java Memory Model (JMM). What are `volatile`, `happens-before`, and how do they prevent visibility issues?

**The Java Memory Model defines the rules under which one thread's write to a variable becomes visible to another thread, with `volatile` and `happens-before` being the core mechanisms ensuring cross-thread visibility without full synchronization.**

**Why JMM Exists:**
- Each thread may cache variables in CPU registers or L1/L2 cache.
- Without JMM guarantees, Thread B may *never* see Thread A's write to a shared variable.

**Happens-Before Rules (Key Ones):**
1. **Program order** — within a thread, each action happens-before the next.
2. **Monitor lock** — `unlock()` happens-before every subsequent `lock()` on the same monitor.
3. **Volatile** — a write to a `volatile` variable happens-before every subsequent read of that variable.
4. **Thread start** — `thread.start()` happens-before any action in the started thread.
5. **Thread join** — all actions in a thread happen-before `join()` returns.

**`volatile` Semantics:**
- **Visibility** — writes are flushed to main memory; reads fetch from main memory.
- **Ordering** — prevents instruction reordering across the volatile access.
- **NOT atomic for compounds** — `volatile int count; count++` is still a race (read-increment-write).

```java
// Classic double-checked locking — volatile is REQUIRED here
public class Singleton {
    private static volatile Singleton INSTANCE; // volatile prevents partial construction visibility

    public static Singleton getInstance() {
        if (INSTANCE == null) {                    // First check (no lock)
            synchronized (Singleton.class) {
                if (INSTANCE == null) {             // Second check (with lock)
                    INSTANCE = new Singleton();     // Without volatile, another thread could
                }                                   // see a partially constructed object
            }
        }
        return INSTANCE;
    }
}
```

**⚠️ Pitfalls:**
- `volatile` does NOT replace `synchronized` for compound actions (read-modify-write).
- Before Java 5, `volatile` did NOT guarantee happens-before — legacy code may have subtle bugs.
- `volatile long/double` is atomic on 64-bit JVMs, but the spec says it's not guaranteed — always use `volatile` for 64-bit fields shared across threads.

---

### Q3. 🔴 🏢 What are `CompletableFuture` patterns you use in production? How does it compare to raw threads and `ExecutorService`?

**`CompletableFuture` is Java's primary abstraction for composable, non-blocking asynchronous pipelines — it replaces callback hell and manual thread coordination with fluent, declarative chaining.**

**Why Over Raw Threads / ExecutorService:**
- Raw threads: No composition, no error handling chains, no combining results.
- `Future.get()`: Blocking. Defeats the purpose of async.
- `CompletableFuture`: Non-blocking composition with `thenApply`, `thenCompose`, `thenCombine`.

**Core Patterns:**

```java
// 1. Fan-out: Call 3 services in parallel, combine results
CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> userService.fetch(id), executor);
CompletableFuture<List<Order>> ordersFuture = CompletableFuture.supplyAsync(() -> orderService.fetch(id), executor);
CompletableFuture<CreditScore> creditFuture = CompletableFuture.supplyAsync(() -> creditService.fetch(id), executor);

CompletableFuture<UserProfile> profile = userFuture.thenCombine(ordersFuture, (user, orders) -> 
        new UserProfile(user, orders))
    .thenCombine(creditFuture, (partial, credit) -> 
        partial.withCreditScore(credit));

// 2. Timeout (Java 9+)
CompletableFuture<String> result = callExternalApi()
    .orTimeout(3, TimeUnit.SECONDS)           // throws TimeoutException
    .exceptionally(ex -> "fallback-value");    // graceful degradation

// 3. allOf — wait for all, fail-fast on first error
CompletableFuture<Void> all = CompletableFuture.allOf(f1, f2, f3);
all.thenRun(() -> log.info("All completed"));

// 4. Exception handling chain
CompletableFuture.supplyAsync(() -> riskyOperation())
    .thenApply(this::transform)
    .exceptionally(ex -> {
        log.error("Failed", ex);
        return defaultValue;
    })
    .thenAccept(this::publish);
```

**Key Differences:**

| Method | Behavior |
|--------|----------|
| `thenApply` | Transform result (like `map`) — same stage if complete, ForkJoinPool if not |
| `thenCompose` | Flatten nested `CompletableFuture` (like `flatMap`) |
| `thenApplyAsync` | Always runs on specified executor |
| `handle` | Receives both result and exception |
| `whenComplete` | Side-effect only, doesn't transform result |

**⚠️ Pitfalls:**
- Default executor is `ForkJoinPool.commonPool()` — shared across the JVM. **Always provide a custom executor** for I/O-bound tasks.
- `exceptionally()` only catches exceptions from previous stages — order matters.
- `thenApply` vs `thenApplyAsync`: the non-async variant can execute on the completing thread — this can cause thread hijacking in I/O workloads.
- Java 21+ Virtual Threads largely reduce the need for `CompletableFuture` in I/O-bound scenarios.

---

### Q4. 🔴 🌐 Explain Java Garbage Collection. What collectors are available, and how do you tune GC for low-latency applications?

**Java GC automatically reclaims heap memory by tracing live object graphs; for low-latency applications, G1GC (default since Java 9) or ZGC/Shenandoah (sub-millisecond pauses) are the production choices, tuned via heap sizing, pause-time goals, and region configuration.**

**Generational Hypothesis:**
- Most objects die young → separate heap into Young Gen (Eden + Survivors) and Old Gen.
- Minor GC (Young Gen) is fast; Major/Full GC (Old Gen) is expensive.

**Collectors Comparison:**

| Collector | Pause | Throughput | Use Case | JVM Flag |
|-----------|-------|-----------|----------|----------|
| **G1GC** | 10–200ms | High | General purpose (default) | `-XX:+UseG1GC` |
| **ZGC** | <1ms | Good | Ultra-low-latency | `-XX:+UseZGC` |
| **Shenandoah** | <10ms | Good | Low-latency (RedHat) | `-XX:+UseShenandoahGC` |
| **Parallel GC** | 100ms+ | Highest | Batch processing | `-XX:+UseParallelGC` |

**G1GC Tuning (Most Common in Interviews):**
```bash
java -XX:+UseG1GC \
     -Xms4g -Xmx4g \                          # Fixed heap size (avoid resizing pauses)
     -XX:MaxGCPauseMillis=100 \                # Target pause time (default 200ms)
     -XX:G1HeapRegionSize=8m \                 # Region size (1–32MB, power of 2)
     -XX:InitiatingHeapOccupancyPercent=35 \   # Start concurrent mark earlier
     -XX:+ParallelRefProcEnabled \             # Parallel reference processing
     -Xlog:gc*:file=gc.log:time,uptime,level   # GC logging
```

**ZGC for Sub-Millisecond (Java 21+):**
```bash
java -XX:+UseZGC -XX:+ZGenerational \   # Generational ZGC (Java 21+)
     -Xms8g -Xmx8g                       # ZGC handles multi-TB heaps
```

**Key Metrics to Monitor:**
- **GC Pause Time** — P99 latency spikes correlate with GC pauses.
- **Allocation Rate** — high allocation rate = frequent young GC.
- **Promotion Rate** — objects surviving to old gen = eventual full GC.
- **Heap After GC** — if growing, you have a memory leak.

**⚠️ Pitfalls:**
- Never set `-Xms` ≠ `-Xmx` in production — heap resizing causes pauses.
- `System.gc()` is a *suggestion*, not a command. Disable with `-XX:+DisableExplicitGC`.
- Large object allocation directly into Old Gen bypasses young GC — watch for `byte[]` buffers.
- G1's `MaxGCPauseMillis` is a *goal*, not a guarantee.

---

### Q5. 🔴 🏢 What is the difference between `synchronized`, `ReentrantLock`, and `StampedLock`? When do you use each?

**`synchronized` is the simplest locking mechanism built into the JVM; `ReentrantLock` adds flexibility (tryLock, fairness, conditions); `StampedLock` provides optimistic read locking for read-heavy workloads where maximum throughput matters.**

**Comparison:**

| Feature | synchronized | ReentrantLock | StampedLock |
|---------|-------------|---------------|-------------|
| Reentrant | Yes | Yes | **No** |
| Try lock | No | `tryLock(timeout)` | `tryOptimisticRead()` |
| Fair ordering | No (biased) | Optional (`new ReentrantLock(true)`) | No |
| Conditions | `wait()/notify()` | Multiple `Condition` objects | No |
| Read/Write | No | Use `ReentrantReadWriteLock` | Built-in (read/write/optimistic) |
| Performance | JVM-optimized (biased locking, lock elision) | Slightly more overhead | Best for read-heavy |

```java
// 1. synchronized — simple critical section
public synchronized void transfer(Account from, Account to, BigDecimal amount) {
    from.debit(amount);
    to.credit(amount);
}

// 2. ReentrantLock — timeout + interruptible
private final ReentrantLock lock = new ReentrantLock();

public boolean transferWithTimeout(Account from, Account to, BigDecimal amount) 
        throws InterruptedException {
    if (lock.tryLock(2, TimeUnit.SECONDS)) {
        try {
            from.debit(amount);
            to.credit(amount);
            return true;
        } finally {
            lock.unlock(); // ALWAYS in finally
        }
    }
    return false; // Could not acquire lock
}

// 3. StampedLock — optimistic read (no blocking for readers)
private final StampedLock sl = new StampedLock();
private double x, y;

public double distanceFromOrigin() {
    long stamp = sl.tryOptimisticRead();      // Non-blocking
    double currentX = x, currentY = y;        // Read shared state
    if (!sl.validate(stamp)) {                 // Check if a write happened
        stamp = sl.readLock();                 // Fallback to pessimistic read
        try {
            currentX = x;
            currentY = y;
        } finally {
            sl.unlockRead(stamp);
        }
    }
    return Math.sqrt(currentX * currentX + currentY * currentY);
}
```

**When to Use:**
- **`synchronized`** — simple mutual exclusion, no timeout/fairness needed. JVM optimizes heavily (biased locking, lock coarsening).
- **`ReentrantLock`** — need `tryLock()`, multiple conditions, or fairness.
- **`StampedLock`** — read-heavy workloads (95%+ reads) where optimistic locking eliminates reader contention.

**⚠️ Pitfalls:**
- `StampedLock` is NOT reentrant — calling `writeLock()` while holding a read lock causes deadlock.
- `ReentrantLock` MUST be unlocked in `finally` — missing this causes permanent lock holding.
- Biased locking was deprecated in Java 15 and removed in Java 18 — `synchronized` performance characteristics have changed.
- For simple atomic operations, `AtomicInteger`/`LongAdder` are always better than any lock.

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

---

### Q6. 🌐 What is the difference between `Stream` API's `map()` and `flatMap()`? Provide a real-world use case.

**`map()` transforms each element one-to-one; `flatMap()` transforms each element into a stream and flattens all resulting streams into a single stream — it's the monad `bind` operation that eliminates nested structures.**

```java
// map: Order -> List<LineItem> (produces Stream<List<LineItem>>)
List<List<LineItem>> nested = orders.stream()
    .map(Order::getLineItems)
    .toList(); // [[item1, item2], [item3]]

// flatMap: Order -> Stream<LineItem> (produces Stream<LineItem>)
List<LineItem> flat = orders.stream()
    .flatMap(order -> order.getLineItems().stream())
    .filter(item -> item.getPrice().compareTo(BigDecimal.TEN) > 0)
    .toList(); // [item1, item2, item3]

// Real-world: Flatten Optional chains
Optional<String> city = getUser()
    .flatMap(User::getAddress)       // Optional<Address>, not Optional<Optional<Address>>
    .flatMap(Address::getCity);
```

**⚠️ Pitfall:** `flatMap` with very large inner streams can cause OOM — consider `Stream.concat()` lazily or process in batches.

---

### Q7. 🌐 Explain the difference between `==` and `equals()` in Java, including the String pool behavior.

**`==` compares object references (memory addresses); `equals()` compares logical equality as defined by the class — for `String`, the interning pool makes `==` deceptively work for literals but fail for `new String()`.**

```java
String a = "hello";               // String pool
String b = "hello";               // Same pool reference
String c = new String("hello");   // New heap object

a == b;                            // true  (same pool reference)
a == c;                            // false (different objects)
a.equals(c);                      // true  (same content)
c.intern() == a;                   // true  (intern returns pool reference)
```

**Contract for `equals()`:**
1. **Reflexive** — `x.equals(x)` is true.
2. **Symmetric** — `x.equals(y)` ↔ `y.equals(x)`.
3. **Transitive** — `x.equals(y)` && `y.equals(z)` → `x.equals(z)`.
4. **Consistent** — multiple calls return the same result.
5. **Null-safe** — `x.equals(null)` returns false.

**⚠️ Pitfall:** If you override `equals()`, you MUST override `hashCode()`. Violating this breaks `HashMap`, `HashSet`, and any hash-based collection.

---

### Q8. 🏢 What are Virtual Threads (Project Loom, Java 21)? How do they change concurrency programming?

**Virtual Threads are lightweight, JVM-managed threads that decouple the Java thread from the OS thread — enabling millions of concurrent threads at near-zero cost, effectively replacing reactive/async programming for I/O-bound workloads.**

**Platform Thread vs. Virtual Thread:**

| Aspect | Platform Thread | Virtual Thread |
|--------|----------------|----------------|
| Backed by | OS thread (1:1) | JVM-managed (M:N) |
| Memory | ~1MB stack each | ~few KB (grows on demand) |
| Creation cost | Expensive (kernel call) | Cheap (~1µs) |
| Max concurrent | ~10K (OS limit) | Millions |
| Blocking I/O | Wastes OS thread | JVM unmounts, carrier freed |

```java
// Creating virtual threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 100_000).forEach(i ->
        executor.submit(() -> {
            // Each task gets its own virtual thread
            var result = httpClient.send(request, BodyHandlers.ofString());
            processResult(result);
        })
    );
} // Executor auto-closes and awaits all tasks

// Spring Boot 3.2+ integration
// application.yml:
// spring.threads.virtual.enabled: true
// This makes all @Async, MVC request handling, and @Scheduled use virtual threads.
```

**⚠️ Pitfalls:**
- **Do NOT pool virtual threads** — create one per task, let them be GC'd.
- **`synchronized` pins the virtual thread to a carrier** — use `ReentrantLock` instead.
- **ThreadLocal abuse** — virtual threads make `ThreadLocal` expensive at scale. Use `ScopedValue` (preview).
- **CPU-bound tasks don't benefit** — virtual threads help only when threads block on I/O.

---

### Q9. 🌐 Explain the `ClassLoader` hierarchy and how class loading works in the JVM.

**The JVM uses a delegating ClassLoader hierarchy — Bootstrap → Platform → Application — where each loader delegates to its parent first, ensuring core classes are loaded once and preventing classpath conflicts.**

**Hierarchy (Java 9+):**
1. **Bootstrap ClassLoader** — loads `java.base` module (formerly `rt.jar`). Implemented in native code.
2. **Platform ClassLoader** — loads platform modules (`java.sql`, `java.xml`). Replaced `ExtClassLoader`.
3. **Application ClassLoader** — loads classes from `--class-path` and `--module-path`.

**Delegation Model:**
```
loadClass("com.app.MyService")
  → AppClassLoader.loadClass() 
    → delegates to PlatformClassLoader.loadClass()
      → delegates to BootstrapClassLoader
        → not found, returns to Platform
      → not found, returns to App
    → AppClassLoader searches classpath → Found!
```

**Custom ClassLoader Use Cases:**
- Hot-reloading (servlet containers like Tomcat)
- Plugin systems (load/unload JARs at runtime)
- Bytecode instrumentation (agents, APM tools)

```java
// Custom classloader for plugin isolation
public class PluginClassLoader extends URLClassLoader {
    public PluginClassLoader(URL[] urls) {
        super(urls, ClassLoader.getPlatformClassLoader()); // Skip app classloader
    }
    
    @Override
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        // Break delegation: try self first for plugin packages
        if (name.startsWith("com.plugin.")) {
            Class<?> c = findLoadedClass(name);
            if (c == null) c = findClass(name);
            if (resolve) resolveClass(c);
            return c;
        }
        return super.loadClass(name, resolve);
    }
}
```

**⚠️ Pitfalls:**
- ClassLoader leaks cause `OutOfMemoryError: Metaspace` — common in app servers during hot redeploys.
- Breaking parent delegation can cause `ClassCastException` — same class loaded by two loaders are incompatible types.
- Java 9 modules (`module-info.java`) add a layer of access control on top of classloading.

---

### Q10. 🏢 What is the `ForkJoinPool` and how does work-stealing work?

**`ForkJoinPool` is a specialized `ExecutorService` designed for recursive divide-and-conquer parallelism, where idle threads *steal* tasks from busy threads' queues to maximize CPU utilization.**

**How Work-Stealing Works:**
1. Each worker thread has its own **double-ended queue (deque)**.
2. A thread pushes/pops tasks from the **head** of its own deque (LIFO — processes the smallest/newest subtask).
3. An idle thread steals from the **tail** of another thread's deque (FIFO — steals the largest/oldest subtask).
4. This minimizes contention: owner and thief operate on opposite ends.

```java
// RecursiveTask example: parallel merge sort
public class ParallelMergeSort extends RecursiveTask<int[]> {
    private static final int THRESHOLD = 1024;
    private final int[] array;

    public ParallelMergeSort(int[] array) { this.array = array; }

    @Override
    protected int[] compute() {
        if (array.length <= THRESHOLD) {
            Arrays.sort(array);   // Sequential below threshold
            return array;
        }
        int mid = array.length / 2;
        ParallelMergeSort left = new ParallelMergeSort(Arrays.copyOfRange(array, 0, mid));
        ParallelMergeSort right = new ParallelMergeSort(Arrays.copyOfRange(array, mid, array.length));

        left.fork();              // Submit left to the pool
        int[] rightResult = right.compute();  // Compute right in current thread
        int[] leftResult = left.join();       // Wait for left

        return merge(leftResult, rightResult);
    }
}

// Usage
ForkJoinPool pool = new ForkJoinPool(Runtime.getRuntime().availableProcessors());
int[] sorted = pool.invoke(new ParallelMergeSort(data));
```

**⚠️ Pitfalls:**
- **`commonPool()` is shared** — `parallelStream()` uses it. A slow task blocks ALL parallel streams JVM-wide.
- **Fork-then-compute-then-join** — NOT fork-then-join-then-compute. The order matters for work-stealing efficiency.
- **Don't use for I/O tasks** — work-stealing assumes CPU-bound work. I/O blocking wastes carrier threads.

---

### Q11. 🌐 How does `HashMap` work internally? What changes were made in Java 8?

**`HashMap` uses an array of buckets where each key's `hashCode()` determines the bucket index; Java 8 introduced treeification — converting buckets from linked lists to red-black trees when collisions exceed a threshold — reducing worst-case lookup from O(n) to O(log n).**

**Internal Structure:**
1. `Node<K,V>[] table` — array of buckets (default initial capacity: 16).
2. Hash computation: `hash = key.hashCode() ^ (key.hashCode() >>> 16)` — spreads high bits to reduce collisions.
3. Bucket index: `index = hash & (capacity - 1)` — bitwise AND (capacity is always power of 2).

**Java 8 Changes:**
- **Treeify threshold** — when a bucket has ≥ 8 nodes AND table size ≥ 64, convert to `TreeNode` (red-black tree).
- **Untreeify threshold** — when tree shrinks to ≤ 6 nodes, convert back to linked list.
- **Resize** — doubles capacity; nodes either stay in the same bucket or move to `oldIndex + oldCapacity`.

```java
// Proper key class for HashMap
public record CacheKey(String tenant, String resource) {
    // Records auto-generate equals() and hashCode() — correct for HashMap keys
}

// Load factor tuning
Map<String, User> cache = new HashMap<>(256, 0.5f); 
// Lower load factor = fewer collisions, more memory
// Default: 0.75 — good trade-off for most cases
```

**⚠️ Pitfalls:**
- Mutable keys in `HashMap` → lost entries. Keys should be effectively immutable.
- Poor `hashCode()` (e.g., constant) → all entries in one bucket → O(n) or O(log n) for every operation.
- `HashMap` is NOT thread-safe — concurrent `put()` can cause infinite loop (Java 7) or data loss (Java 8+).

---

### Q12. 🏢 What are the different types of references in Java (Strong, Weak, Soft, Phantom)? When do you use each?

**Java provides four reference types with increasing reclaimability — Strong (default, never GC'd while reachable), Soft (GC'd under memory pressure), Weak (GC'd at next GC cycle), and Phantom (post-finalization, for cleanup callbacks).**

| Reference | GC Behavior | Use Case |
|-----------|------------|----------|
| **Strong** | Never collected while reachable | Normal variables |
| **Soft** (`SoftReference`) | Collected only under memory pressure | Memory-sensitive caches |
| **Weak** (`WeakReference`) | Collected at next GC regardless | Canonical maps, listener registries |
| **Phantom** (`PhantomReference`) | Enqueued after finalization, before memory reclaimed | Resource cleanup (replaces `finalize()`) |

```java
// WeakHashMap for auto-evicting metadata cache
Map<Connection, SessionMetadata> sessions = new WeakHashMap<>();
// When Connection is GC'd, the entry is automatically removed

// SoftReference for memory-sensitive image cache
Map<String, SoftReference<BufferedImage>> imageCache = new ConcurrentHashMap<>();

public BufferedImage getImage(String path) {
    SoftReference<BufferedImage> ref = imageCache.get(path);
    BufferedImage img = (ref != null) ? ref.get() : null;
    if (img == null) {
        img = loadFromDisk(path);
        imageCache.put(path, new SoftReference<>(img));
    }
    return img;
}

// Phantom reference for native resource cleanup (Java 9+ Cleaner)
Cleaner cleaner = Cleaner.create();
cleaner.register(myObject, () -> freeNativeMemory(nativePtr));
```

**⚠️ Pitfalls:**
- `SoftReference` retention is JVM-dependent — some JVMs hold them too long, causing OOM.
- `WeakHashMap` is NOT thread-safe — use `Collections.synchronizedMap()` or guard externally.
- **Never use `finalize()`** — it's deprecated and removed in Java 18. Use `Cleaner` or try-with-resources.

---

## 🟢 GOOD TO KNOW (Questions 13–20)

---

### Q13. 🌐 What is the difference between `Callable` and `Runnable`?

**`Runnable.run()` returns `void` and cannot throw checked exceptions; `Callable.call()` returns a typed result and can throw checked exceptions — making `Callable` the correct choice for tasks that produce values or may fail.**

```java
// Runnable — fire-and-forget
Runnable task = () -> log.info("Processing...");

// Callable — returns result, throwable
Callable<Report> task = () -> {
    if (!hasAccess()) throw new SecurityException("Denied");
    return generateReport();
};

Future<Report> future = executor.submit(task);
Report report = future.get(5, TimeUnit.SECONDS); // Blocking with timeout
```

**⚠️ Pitfall:** `future.get()` wraps checked exceptions in `ExecutionException` — always unwrap: `ex.getCause()`.

---

### Q14. 🏢 What is the `happens-before` relationship with `final` fields?

**A `final` field's value, assigned in the constructor, is guaranteed to be visible to all threads once the constructor completes — without any synchronization — provided the `this` reference does not escape during construction.**

```java
public class ImmutableConfig {
    private final Map<String, String> properties;

    public ImmutableConfig(Map<String, String> source) {
        this.properties = Map.copyOf(source); // Defensive copy + unmodifiable
        // JMM guarantees: any thread reading 'properties' after construction
        // sees the fully initialized map — no volatile/synchronized needed.
    }

    // DO NOT do this — 'this' escapes before construction finishes
    // registry.register(this); // WRONG: other thread may see partially constructed object
}
```

**⚠️ Pitfall:** If `this` escapes the constructor (e.g., passing it to another thread, registering a listener), the `final` field guarantee is void.

---

### Q15. 🌐 Explain `ThreadLocal` and its pitfalls in web applications.

**`ThreadLocal` provides per-thread isolated storage where each thread has its own independent copy of the variable — essential for per-request context (MDC, transactions), but dangerous in thread pools where threads are reused.**

```java
// Common pattern: request context propagation
public class RequestContext {
    private static final ThreadLocal<String> CORRELATION_ID = new ThreadLocal<>();

    public static void set(String id) { CORRELATION_ID.set(id); }
    public static String get() { return CORRELATION_ID.get(); }
    public static void clear() { CORRELATION_ID.remove(); } // CRITICAL!
}

// Spring interceptor — ensures cleanup
@Component
public class CorrelationInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
        RequestContext.set(req.getHeader("X-Correlation-ID"));
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest req, HttpServletResponse res, 
                                 Object handler, Exception ex) {
        RequestContext.clear(); // ALWAYS clean up — thread is returned to pool
    }
}
```

**⚠️ Pitfalls:**
- **Memory leak** — forgetting `remove()` in thread pools leaks data AND prevents GC of the value.
- **Virtual Threads** — `ThreadLocal` is expensive with millions of virtual threads. Use `ScopedValue` (preview in Java 21+).
- **`InheritableThreadLocal`** — copies parent value to child thread, but NOT to executor-managed threads without custom handling.

---

### Q16. 🏢 What are sealed classes and when should you use them?

**Sealed classes (Java 17) restrict which classes can extend them — enabling exhaustive pattern matching and modeling closed domain hierarchies where the set of subtypes is fixed and known at compile time.**

```java
// Domain modeling: a payment can ONLY be one of these types
public sealed interface Payment permits CreditCardPayment, BankTransfer, DigitalWallet {
    BigDecimal amount();
}

public record CreditCardPayment(BigDecimal amount, String cardLastFour) implements Payment {}
public record BankTransfer(BigDecimal amount, String iban) implements Payment {}
public record DigitalWallet(BigDecimal amount, WalletType type) implements Payment {}

// Exhaustive pattern matching (Java 21+)
public String processPayment(Payment payment) {
    return switch (payment) {
        case CreditCardPayment cc -> chargeCard(cc.cardLastFour(), cc.amount());
        case BankTransfer bt      -> initTransfer(bt.iban(), bt.amount());
        case DigitalWallet dw     -> chargeWallet(dw.type(), dw.amount());
        // No default needed — compiler knows all cases are covered
    };
}
```

**⚠️ Pitfall:** Sealed types only restrict direct subclasses. `non-sealed` subclasses can be extended freely — use this intentionally, not accidentally.

---

### Q17. 🏬 What is the difference between `fail-fast` and `fail-safe` iterators?

**Fail-fast iterators (e.g., `ArrayList`, `HashMap`) throw `ConcurrentModificationException` when the collection is structurally modified during iteration; fail-safe iterators (e.g., `CopyOnWriteArrayList`, `ConcurrentHashMap`) work on a snapshot or are weakly consistent, never throwing exceptions.**

```java
// Fail-fast — throws ConcurrentModificationException
List<String> list = new ArrayList<>(List.of("a", "b", "c"));
for (String s : list) {
    if (s.equals("b")) list.remove(s); // BOOM! CME
}
// Correct: use iterator.remove() or list.removeIf(s -> s.equals("b"))

// Fail-safe — works on snapshot
List<String> cowList = new CopyOnWriteArrayList<>(List.of("a", "b", "c"));
for (String s : cowList) {
    if (s.equals("b")) cowList.remove(s); // No exception — iterates over snapshot
}
```

**⚠️ Pitfall:** `CopyOnWriteArrayList` copies the entire array on every write — suitable only for read-heavy/write-rare scenarios (e.g., listener lists).

---

### Q18. 🏬 How does `record` type differ from a regular class? What are its limitations?

**Records (Java 16) are transparent carriers of immutable data — the compiler generates `equals()`, `hashCode()`, `toString()`, accessors, and a canonical constructor — making them ideal for DTOs, value objects, and API responses.**

```java
// Compact — replaces 50+ lines of boilerplate
public record OrderSummary(
    UUID orderId,
    String customerName,
    BigDecimal total,
    Instant createdAt
) {
    // Compact constructor for validation
    public OrderSummary {
        Objects.requireNonNull(orderId, "orderId must not be null");
        if (total.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Total cannot be negative");
        }
        customerName = customerName.trim(); // Can reassign parameters
    }
}
```

**Limitations:**
- Cannot extend another class (implicitly extends `Record`).
- All fields are `final` — no setters, truly immutable.
- Cannot declare instance fields beyond components.
- Can implement interfaces, though.

**⚠️ Pitfall:** Records with mutable component types (e.g., `List`, `Date`) are NOT truly immutable — use `List.copyOf()` in the compact constructor.

---

### Q19. 🏬 Explain the `try-with-resources` mechanism and how to create custom `AutoCloseable` resources.

**`try-with-resources` guarantees deterministic cleanup of resources implementing `AutoCloseable` — the compiler generates `finally` blocks that call `close()` in reverse declaration order, even if exceptions are thrown.**

```java
// Multiple resources — closed in reverse order (conn last)
try (
    var conn = dataSource.getConnection();
    var stmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
    var rs = executeQuery(stmt, userId)
) {
    return mapResultSet(rs);
}  // rs.close() → stmt.close() → conn.close()

// Custom AutoCloseable
public class DistributedLock implements AutoCloseable {
    private final String lockKey;
    private final RedisCommands<String, String> redis;

    public DistributedLock(RedisCommands<String, String> redis, String key, Duration ttl) {
        this.redis = redis;
        this.lockKey = key;
        String result = redis.set(key, "locked", SetArgs.Builder.nx().ex(ttl));
        if (!"OK".equals(result)) throw new LockAcquisitionException("Failed to acquire: " + key);
    }

    @Override
    public void close() {
        redis.del(lockKey);
    }
}

// Usage
try (var lock = new DistributedLock(redis, "order:" + orderId, Duration.ofSeconds(30))) {
    processOrder(orderId);
} // Lock automatically released
```

**⚠️ Pitfall:** If `close()` throws, the primary exception is preserved and `close()` exceptions are added as **suppressed exceptions** — check `ex.getSuppressed()`.

---

### Q20. 🌐 What is the difference between `Comparable` and `Comparator`? When do you use each?

**`Comparable` defines a type's *natural ordering* within the class itself (single sort logic); `Comparator` defines *external, reusable ordering strategies* that can be composed and swapped at runtime — use `Comparable` for canonical ordering, `Comparator` for everything else.**

```java
// Comparable — natural order (one per class)
public record Employee(String name, int salary) implements Comparable<Employee> {
    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary); // Natural: by salary
    }
}

// Comparator — flexible, composable (Java 8+)
Comparator<Employee> byNameThenSalary = Comparator
    .comparing(Employee::name, String.CASE_INSENSITIVE_ORDER)
    .thenComparingInt(Employee::salary)
    .reversed();

List<Employee> sorted = employees.stream()
    .sorted(byNameThenSalary)
    .toList();

// Null-safe comparator
Comparator<Employee> nullSafe = Comparator.nullsLast(
    Comparator.comparing(Employee::name)
);
```

**⚠️ Pitfall:** Never use `a.salary - b.salary` in comparators — integer overflow produces wrong results. Always use `Integer.compare()` or `Comparator.comparingInt()`.

---

### Q32. 🧩 What is a marker interface in spring boot (Java)?

**A marker interface is an interface that has no methods or fields. It simply "marks" a class to indicate that it possesses a certain property or behavior to the JVM or framework.**

- In core Java, examples are `Serializable`, `Cloneable`, and `RandomAccess`.
- While Spring doesn't heavily rely on marker interfaces directly, an older example in the Spring ecosystem would be the `Aware` super-interface (though its sub-interfaces have methods). Generally, Spring Boot prefers annotations (like `@Component`, `@Service`) over marker interfaces to tag classes and provide metadata. If the interviewer asked specifically about "spring boot", they might be testing your fundamental Java knowledge or referring to how annotations have largely replaced the need for marker interfaces in modern frameworks.

---

### Q33. 🛠️ Functional interface in spring boot (Java)

**A functional interface is a Java 8 concept—an interface that contains exactly one abstract method. They can have multiple default or static methods. They are used extensively in lambda expressions and method references.**

- In the context of Spring Boot/Java, examples include `Runnable`, `Callable`, `Comparator`, `Predicate`, `Function`, `Consumer`, and `Supplier`.
- Spring Framework specific examples: `ApplicationRunner` and `CommandLineRunner` (both have exactly one `run` method and are often used as functional interfaces to execute code at startup), `RowMapper` in Spring JDBC.

---

### Q34. 🔄 Difference between Runnable and Callable

**Both are functional interfaces used to define a task that can be executed concurrently by a thread, but they have key differences:**

- **Return Type**: `Runnable`'s `run()` method returns `void` (no result). `Callable`'s `call()` method returns a generic result (`V`).
- **Exception Handling**: `Runnable`'s `run()` method cannot throw checked exceptions (they must be caught inside). `Callable`'s `call()` method can throw a checked `Exception`.
- **Usage**: `Runnable` can be used with `Thread` or `ExecutorService`. `Callable` can only be executed by `ExecutorService` (which returns a `Future` object to retrieve the result later).

---

### Q35. 🗺️ Use of flatMap()

**`flatMap()` is a method in Java 8 Streams used for both transformation (mapping) and flattening.**

- While `map()` transforms each element into a single new value, `flatMap()` transforms each element into a stream of zero or more values, and then flattens all those generated streams into a single, continuous stream.
- **Use case**: If you have a `List<List<String>>` and you want a single `List<String>` containing all the inner strings, you would use `flatMap(List::stream)`. It's used to deal with nested structures and one-to-many relationships in data processing.

---

### Q36. 🟢 🏢 How do you identify, diagnose, and resolve a memory leak or deadlock in a Java application?
- **Memory Leak**: Use tools like JConsole or Grafana. A classic sign is Old Generation growing continuously. Capture a heap dump (`jmap`) and analyze in Eclipse MAT to trace GC Roots.
- **Deadlock**: If the app hangs, take a Thread Dump (`jstack`). The JVM will explicitly print "Found one Java-level deadlock". Prevent by locking in a consistent order or using `tryLock(timeout)`.

---

### Q37. 🟢 🏢 Explain the purpose of `default` and `static` methods in Java 8 Interfaces.
- **`default`**: Allow adding new methods to interfaces without breaking existing implementing classes (backward compatibility).
- **`static`**: Belong to the interface itself, used for utility methods specific to the interface (e.g., `Stream.of()`).
