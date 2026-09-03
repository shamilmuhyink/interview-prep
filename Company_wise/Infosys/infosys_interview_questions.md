# Infosys Senior Java Full Stack Developer - Interview Questions (Top 50)

> **Source:** Aggregated from PrepInsta, AmbitionBox, GeeksforGeeks, and Reddit.
> **Focus:** Core Java, Spring Boot, Microservices, System Architecture, Database, and Angular/React.
> **Format:** Ordered by frequency.

---

### Q1. 🔴 🏬 Explain the internal working of `HashMap` in Java.
**Answer:** The `HashMap` operates on the principle of hashing, using an array of nodes (buckets) to store key-value pairs. When an element is added, the key's `hashCode()` determines its bucket index; if a collision occurs, the entries are linked together. In Java 8 and above, to optimize performance, a bucket seamlessly transitions from a standard **LinkedList** to a **Red-Black Tree** once it exceeds the `TREEIFY_THRESHOLD` of 8 elements, which drastically improves worst-case search time from **O(N)** to **O(log N)**.

### Q2. 🔴 🏬 What is the difference between `HashMap` and `ConcurrentHashMap`?
**Answer:** While `HashMap` is inherently **not thread-safe** and can result in infinite loops during concurrent resizing, `ConcurrentHashMap` is designed specifically for multithreading. Starting in Java 8, `ConcurrentHashMap` abandons segment-level locking in favor of **Compare-And-Swap (CAS)** operations and synchronizes only at the specific node (bucket head) being updated, allowing multiple threads to read and write concurrently without locking the entire map.

### Q3. 🔴 🏬 Explain the four pillars of OOPs with examples.
**Answer:** The four fundamental concepts of Object-Oriented Programming are **Encapsulation**, **Abstraction**, **Inheritance**, and **Polymorphism**. 
- **Encapsulation** is the bundling of data and methods (e.g., using `private` fields and getters/setters). 
- **Abstraction** hides complex implementation details, exposing only necessary features (e.g., using `interfaces` or `abstract classes`). 
- **Inheritance** allows a class to inherit properties from a parent class to promote code reusability (e.g., `Dog extends Animal`). 
- **Polymorphism** permits objects to be treated as instances of their parent class, achieved through method overloading (compile-time) and method overriding (run-time).

### Q4. 🔴 🏬 How does Java 8 Streams API work? Write a snippet to filter and map.
**Answer:** The Streams API introduces a functional, declarative approach to processing collections, enabling developers to chain operations like filtering and mapping without mutating the underlying data. It supports lazy evaluation and can easily execute operations in parallel.
```java
List<String> result = names.stream()
    .filter(name -> name.startsWith("A"))
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### Q5. 🔴 🏬 What is the difference between Monolithic and Microservices architecture?
**Answer:** A **Monolithic** architecture bundles the entire application's functionality into a single, tightly coupled deployable unit, which can make scaling and deploying updates cumbersome. In contrast, a **Microservices** architecture decomposes the application into small, independent, and loosely coupled services that communicate over a network, allowing discrete teams to develop, deploy, and scale specific features independently.

### Q6. 🟠 🏬 What is the difference between `String`, `StringBuilder`, and `StringBuffer`?
**Answer:** In Java, a **`String`** is completely immutable, meaning every modification creates a new object in the String pool. When mutability is required, **`StringBuffer`** offers a thread-safe option since its methods are synchronized, whereas **`StringBuilder`** provides a significantly faster, **non-thread-safe** alternative that is universally preferred for string manipulation within a single thread.

### Q7. 🟠 🏬 Explain Garbage Collection in Java (G1 vs ZGC).
**Answer:** Garbage Collection is the JVM's automated process of reclaiming heap memory by destroying objects that are no longer reachable. The **G1 (Garbage-First)** collector partitions the heap into equal-sized regions and prioritizes sweeping regions that contain the most garbage to meet predictable pause-time goals. The **ZGC (Z Garbage Collector)** is a highly scalable, low-latency modern collector that performs expensive work concurrently, ensuring pause times remain in the sub-millisecond range regardless of heap size.

### Q8. 🟠 🏬 What is the difference between `Runnable` and `Callable`?
**Answer:** Both interfaces represent asynchronous tasks meant to be executed by concurrent threads. The key difference is that a **`Runnable`** executes a `run()` method which cannot return a result or throw a checked exception. Conversely, a **`Callable`** executes a `call()` method that gracefully returns a generic result and is fully permitted to throw checked exceptions.

### Q9. 🟠 🏬 What is the difference between Checked and Unchecked Exceptions?
**Answer:** **Checked exceptions** (like `IOException` or `SQLException`) are verified at compile-time, forcing the developer to either catch them or declare them in the method signature using `throws`. **Unchecked exceptions** (like `NullPointerException` or `IllegalArgumentException`), which inherit from `RuntimeException`, are not enforced by the compiler and generally represent programming errors that should be fixed rather than caught.

### Q10. 🟠 🏬 Explain Spring DI (Dependency Injection) and IoC.
**Answer:** **Inversion of Control (IoC)** is an architectural principle where the framework (the Spring Container) takes control of managing the application's flow and object instantiation. **Dependency Injection (DI)** is the specific design pattern used to implement IoC; instead of objects manually creating their dependencies, the Spring container dynamically injects the required dependencies via constructors, setters, or fields at runtime.

### Q11. 🟠 🏬 How does Spring Boot Auto-configuration work?
**Answer:** Spring Boot's Auto-configuration intelligently attempts to automatically configure your application based on the `.jar` dependencies present on the classpath. Triggered by the `@EnableAutoConfiguration` annotation, it utilizes `@Conditional` annotations (like `@ConditionalOnClass` or `@ConditionalOnMissingBean`) to register default beans only when you haven't explicitly defined your own custom configurations.

### Q12. 🟠 🏬 Explain the JWT/OAuth2 authentication flow.
**Answer:** In a standard JWT flow, a user authenticates with their credentials, and the server generates a cryptographically signed **JSON Web Token (JWT)**. The client securely stores this token and attaches it to the `Authorization: Bearer <token>` header of all subsequent API requests. The server intercepts these requests, verifies the JWT's signature and expiration without needing a database lookup, and grants access if valid.

### Q13. 🟠 🏬 What is a Circuit Breaker? (Resilience4j)
**Answer:** The **Circuit Breaker** pattern protects microservices from cascading failures. If a downstream service repeatedly fails or times out, the circuit "opens," immediately failing fast and returning a graceful fallback response instead of blocking threads. After a predefined timeout, it transitions to a "half-open" state to test if the failing service has recovered before fully closing the circuit again.

### Q14. 🟠 🏬 What is an API Gateway?
**Answer:** An **API Gateway** acts as the singular, centralized entry point for all client requests entering a microservices architecture. It abstracts the complexity of the backend by handling crucial cross-cutting concerns such as **routing requests** to the correct internal microservices, enforcing **security and authentication**, managing **rate limiting**, and providing centralized logging.

### Q15. 🟠 🏬 How do you handle Global Exceptions in Spring Boot?
**Answer:** Global exception handling in Spring Boot is gracefully managed by annotating a centralized class with **`@ControllerAdvice`** (or `@RestControllerAdvice`). Within this class, methods annotated with **`@ExceptionHandler`** are defined to intercept specific exception types thrown anywhere in the application, allowing you to return standardized, consistent JSON error responses across all APIs.

### Q16. 🟠 🏬 What are ACID properties?
**Answer:** The ACID properties ensure reliable processing of database transactions:
- **Atomicity:** The transaction is "all or nothing"; if one part fails, the entire transaction rolls back.
- **Consistency:** The database strictly transitions from one valid state to another, enforcing all constraints.
- **Isolation:** Concurrent transactions operate entirely independently and do not interfere with each other.
- **Durability:** Once a transaction is successfully committed, its changes are permanently persisted, even in the event of a system crash.

### Q17. 🟠 🏬 Explain SQL Joins.
**Answer:** SQL Joins are used to combine rows from two or more tables based on a related column. 
- An **INNER JOIN** returns only the records that have matching values in both tables.
- A **LEFT JOIN** returns all records from the left table, along with any matched records from the right table.
- A **RIGHT JOIN** returns all records from the right table and matched records from the left.
- A **FULL OUTER JOIN** returns all records when there is a match in either the left or right table.

### Q18. 🟠 🏬 What is the N+1 Query Problem in Hibernate?
**Answer:** The N+1 Query Problem is a severe performance anti-pattern that occurs when an ORM fetches a list of parent entities (1 query) and subsequently executes an additional query for each parent to fetch its lazily-loaded child entities (N queries). This is optimally resolved by instructing Hibernate to load the relationships in a single optimized query using the **`JOIN FETCH`** keyword in JPQL or by applying Spring Data JPA's **`@EntityGraph`**.

### Q19. 🟠 🏬 Explain Database Indexing (B-Tree).
**Answer:** Database indexing is a powerful optimization technique that creates specialized data structures to drastically accelerate data retrieval speeds. Most relational databases default to using **B-Tree (Balanced Tree)** indexes, which keep data logically sorted and allow searches, sequential access, insertions, and deletions to operate in highly efficient **O(log N)** time rather than scanning the entire table.

### Q20. 🟠 🏬 Docker (Image vs Container).
**Answer:** A Docker **Image** is a read-only, immutable template that contains the application code, runtime environment, libraries, and necessary configuration files. A Docker **Container** is a live, running instance of that image; it represents an isolated, executable environment that can be started, stopped, and scaled independently.

### Q21. 🟠 🏬 CI/CD Pipeline Concepts.
**Answer:** **Continuous Integration (CI)** is the automated practice where developers frequently merge code changes into a central repository, triggering automated builds and unit tests to catch bugs early. **Continuous Deployment/Delivery (CD)** extends this by automatically packaging the validated code and securely deploying it to staging or production environments, ensuring a rapid and reliable release lifecycle.

### Q22. 🟠 🏬 Two Sum Problem.
**Answer:** The "Two Sum" algorithmic challenge requires finding the indices of two distinct numbers in an array that add up to a specific target. The optimal solution achieves an **O(N)** time complexity by utilizing a **HashMap** to store each number and its original index during iteration, instantly checking if the required complement (`target - current_number`) already exists in the map.

### Q23. 🟠 🏬 Palindrome Check (String).
**Answer:** To check if a string is a palindrome (reading the same forwards and backwards), the most efficient approach utilizes a **two-pointer technique**. By placing one pointer at the beginning of the string and another at the end, you incrementally compare the characters while moving the pointers inward; if any mismatch is found, it is not a palindrome.

### Q24. 🟠 🏬 Character/Word Frequency Counting.
**Answer:** The standard approach for frequency counting involves iterating over the input and using a **HashMap** to map each character or word to its occurrence count. In modern Java, this can be elegantly achieved in a single line by streaming the elements and utilizing the `Collectors.groupingBy()` collector paired with `Collectors.counting()`.

### Q25. 🟡 🏬 What is the difference between `map()` and `flatMap()` in Streams?
**Answer:** In the Java Streams API, **`map()`** is strictly used for a one-to-one transformation, converting each element into exactly one new element. Conversely, **`flatMap()`** is utilized for a one-to-many transformation; it takes an element, maps it to a stream of multiple elements, and seamlessly flattens all the resulting nested streams back into a single continuous stream.

### Q26. 🟡 🏬 `volatile` vs `synchronized`.
**Answer:** The **`volatile`** keyword guarantees memory visibility across threads by forcing the JVM to always read and write the variable directly to main memory rather than a thread-local cache, though it does not prevent race conditions. The **`synchronized`** keyword provides both visibility and strict mutual exclusion, ensuring that only one thread can execute a block of code at a time, making compound operations completely thread-safe.

### Q27. 🟡 🏬 `ArrayList` vs `LinkedList`.
**Answer:** An **`ArrayList`** is backed by a resizable dynamic array, making it exceptionally fast for random access `O(1)` but slow for insertions and deletions in the middle `O(N)` since elements must be shifted. A **`LinkedList`** is backed by a doubly-linked list, offering extremely fast `O(1)` insertions and deletions when the node reference is known, but suffering from slow `O(N)` sequential traversal for access.

### Q28. 🟡 🏬 Difference between `Comparable` and `Comparator`.
**Answer:** The **`Comparable`** interface is implemented directly by the class itself (e.g., `implements Comparable<T>`) to define a single, default "natural" sorting order using the `compareTo()` method. In contrast, the **`Comparator`** interface is external to the class, allowing you to define multiple, flexible custom sorting sequences (like sorting by age, then by name) using the `compare()` method without modifying the original source code.

### Q29. 🟡 🏬 Explain the Spring Bean Lifecycle.
**Answer:** The Spring Bean lifecycle is fully managed by the IoC container. It begins with instantiation, followed by dependency injection (populating properties). Next, aware interfaces like `BeanNameAware` are invoked, followed by custom initialization methods annotated with **`@PostConstruct`**. Once fully initialized, the bean is ready for use, and upon application shutdown, destruction methods annotated with **`@PreDestroy`** are executed to clean up resources.

### Q30. 🟡 🏬 What is Service Discovery (Eureka)?
**Answer:** Service Discovery is a crucial microservices pattern that allows services to find and communicate with each other dynamically. Using a registry like **Netflix Eureka**, microservices register their active IP addresses and ports upon startup; other services can then query the registry to resolve the physical locations of their dependencies, completely eliminating the need for fragile, hardcoded URLs.

### Q31. 🟡 🏬 Explain the Saga Pattern for distributed transactions.
**Answer:** The **Saga Pattern** effectively manages data consistency across microservices without relying on distributed locks (two-phase commit). It breaks a global transaction into a sequence of isolated local transactions; if any step in the sequence fails, the system executes predefined **compensating transactions** in reverse order to seamlessly undo the work completed by the preceding steps.

### Q32. 🟡 🏬 What is `@RestController` vs `@Controller`?
**Answer:** While **`@Controller`** is traditionally used in Spring MVC to return HTML views, **`@RestController`** is a specialized convenience annotation for building RESTful APIs. It implicitly combines `@Controller` and `@ResponseBody`, ensuring that the return values of all methods are automatically serialized directly into JSON or XML format within the HTTP response body.

### Q33. 🟡 🏬 What is Spring Boot Actuator?
**Answer:** **Spring Boot Actuator** is a powerful sub-project that automatically exposes a suite of built-in HTTP endpoints (such as `/actuator/health`, `/metrics`, and `/env`) designed specifically to monitor, audit, and interact with a production application. It provides deep visibility into the application's internal state, memory usage, and component health without writing custom monitoring code.

### Q34. 🟡 🏬 `@Component` vs `@Service` vs `@Repository`?
**Answer:** All three are Spring stereotype annotations that register classes as beans, but they carry distinct semantic meanings. **`@Component`** is a generic stereotype for any Spring-managed component; **`@Service`** specifically denotes the business logic layer; and **`@Repository`** explicitly indicates the data access layer, automatically activating Spring's translation of underlying database exceptions into unchecked Spring `DataAccessException`s.

### Q35. 🟡 🏬 Explain `@Transactional` propagation.
**Answer:** Transaction propagation dictates how an executing method behaves if a transaction context already exists. The default, **`REQUIRED`**, will seamlessly join an active transaction or create a new one if none exists. Conversely, **`REQUIRES_NEW`** will forcefully suspend the current transaction, spin up a completely independent new transaction, and resume the original only after the new one finishes.

### Q36. 🟡 🏬 Normalization (1NF to 3NF).
**Answer:** Normalization is the systematic process of structuring database tables to eliminate redundant data. 
- **1NF (First Normal Form):** Ensures all columns contain atomic, indivisible values.
- **2NF:** Achieves 1NF and guarantees that all non-key attributes are fully dependent on the entire primary key (no partial dependencies).
- **3NF:** Achieves 2NF and ensures that no non-key attribute relies on another non-key attribute (no transitive dependencies).

### Q37. 🟡 🏬 Explain Angular Lifecycle Hooks.
**Answer:** Angular components possess a defined lifecycle managed by specific hook methods. Crucial hooks include **`ngOnChanges()`** which fires whenever an `@Input()` binding is updated, **`ngOnInit()`** which executes once for component initialization (ideal for API calls), and **`ngOnDestroy()`** which runs immediately before the component is destroyed, making it the required location to unsubscribe from observables to prevent memory leaks.

### Q38. 🟡 🏬 Angular Routing and Lazy Loading.
**Answer:** Angular's **Router** enables seamless navigation between different views in a Single Page Application (SPA). **Lazy Loading** is a critical optimization technique where specific feature modules are loaded asynchronously on-demand only when a user navigates to their route, drastically reducing the initial JavaScript bundle size and accelerating the application's startup time.

### Q39. 🟡 🏬 Observable vs Promise (RxJS).
**Answer:** While both handle asynchronous operations, a **Promise** executes eagerly and emits exactly one value before completing. An **Observable** (powered by RxJS) is completely lazy (executes only when subscribed to), can emit a continuous stream of multiple values over time, supports cancellation via `unsubscribe()`, and offers powerful data transformation operators like `map` and `switchMap`.

### Q40. 🟡 🏬 Kubernetes Basics (Pod, Service).
**Answer:** In Kubernetes, a **Pod** is the smallest and most fundamental deployable unit, representing a single instance of a running process that typically encapsulates one or more highly coupled Docker containers. Because Pods are ephemeral and their IPs change frequently, a Kubernetes **Service** is used to group Pods logically and provide a stable, persistent IP address and DNS name for reliable networking.

### Q41. 🟡 🏬 Agile Scrum Methodology.
**Answer:** **Scrum** is an iterative Agile framework designed to deliver working software incrementally. Work is organized into fixed-length cycles called **Sprints** (usually 2 weeks). It relies on distinct roles (Scrum Master, Product Owner, Development Team) and relies heavily on structured ceremonies, including Daily Standups, Sprint Planning, and Sprint Retrospectives to drive continuous improvement.

### Q42. 🟡 🏬 Reverse a Linked List.
**Answer:** Reversing a linked list involves iterating through the nodes and systematically redirecting each node's `next` pointer to point to its preceding node. This is optimally achieved in **O(N)** time and **O(1)** auxiliary space by maintaining three separate pointers during traversal: `prev`, `current`, and `next`.

### Q43. 🟡 🏬 Detect Cycle in Linked List (Floyd's).
**Answer:** Floyd’s Cycle-Finding Algorithm elegantly detects loops using two pointers moving at different speeds. A **slow pointer** advances one node at a time while a **fast pointer** advances two; if a cycle exists, the fast pointer will inevitably loop around and intersect with the slow pointer.

### Q44. 🟢 🏬 Explain CompletableFuture and asynchronous programming.
**Answer:** Introduced in Java 8, **`CompletableFuture`** is a powerful enhancement over the classic `Future` interface, enabling truly non-blocking asynchronous programming. It empowers developers to seamlessly chain asynchronous tasks together using functional callbacks like `thenApply()` or `thenAccept()`, and robustly combine the results of multiple independent futures without manually managing thread execution.

### Q45. 🟢 🏬 What is the difference between `wait()` and `sleep()`?
**Answer:** The **`wait()`** method belongs to the `Object` class and is used for inter-thread communication; it forcefully relinquishes the acquired lock and pauses the thread until notified. Conversely, **`sleep()`** is a static utility of the `Thread` class that pauses execution for a specified duration but **strictly retains all locks** it holds during the slumber.

### Q46. 🟢 🏬 Difference between `throw` and `throws`.
**Answer:** The **`throw`** keyword is utilized explicitly inside a method block to manually instantiate and throw a specific exception object (e.g., `throw new Exception()`). The **`throws`** keyword is strictly appended to the method's signature declaration, serving as a warning to the caller that the method is capable of throwing one or more checked exceptions that must be handled.

### Q47. 🟢 🏬 What is Spring WebFlux?
**Answer:** **Spring WebFlux** is the fully reactive, non-blocking alternative to traditional Spring MVC, built on the foundations of Project Reactor. Designed explicitly for high-concurrency systems, it utilizes event-loop architecture to efficiently process a massive number of concurrent requests with a minimal thread pool, making it ideal for streaming data and microservices with heavy I/O operations.

### Q48. 🟢 🏬 Kafka Architecture (Topics, Partitions).
**Answer:** Apache Kafka is an enterprise-grade distributed event streaming platform. Data streams are categorized into **Topics**, which are internally sharded into multiple **Partitions** across various broker servers. This partitioning enables massively parallel processing for consumers, while Kafka's strict replication of these partitions guarantees high availability and zero data loss during broker outages.

### Q49. 🟢 🏬 SQL vs NoSQL.
**Answer:** **SQL** databases are strictly relational, utilizing predefined schemas and relying on the rigid ACID properties, making them ideal for complex, structured queries and transactional integrity. **NoSQL** databases are highly flexible, non-relational datastores (document, key-value) that embrace BASE principles (Eventual Consistency) and are fundamentally optimized for massive horizontal scaling and unstructured data.

### Q50. 🟢 🏬 What are Angular Interceptors?
**Answer:** Angular **Interceptors** are robust global middleware services used to systematically intercept, inspect, and transform every outgoing HTTP request or incoming HTTP response. They are universally employed to cleanly attach authorization headers (like JWT tokens) to requests, implement global error handling, or inject loading spinners without polluting individual component logic.
