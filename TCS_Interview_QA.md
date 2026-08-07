# TCS Java Fullstack Microservices Developer Interview Guide 2026
*Prioritized by descending order of frequency based on recent AmbitionBox data and TCS interview trends.*

### 1. "Tell me about yourself" & "Explain your current project / roles and responsibilities."
**Why it's asked:** To gauge your communication skills, tech stack familiarity, and exact role in the SDLC.
**How to answer:** 
- Keep it to 2-3 minutes.
- **Formula:** Present -> Past -> Project Details -> Your specific contribution.
- **Example:** "I am a Java Full Stack Developer with X years of experience. In my current project, I developed a web-based inventory management system for a retail client. I used **Java 8+, Spring Boot, and MySQL** for the backend, and implemented authentication using **Spring Security**. My core responsibility was designing and developing scalable RESTful APIs, optimizing database queries, and migrating monolithic modules into autonomous Microservices. I also handled deployments using our CI/CD pipeline (Jenkins/Docker)."

### 2. What are the differences between Spring Boot and Spring MVC?
| Feature | Spring MVC | Spring Boot |
| :--- | :--- | :--- |
| **Purpose** | A framework module specifically used for building web applications (Model-View-Controller). | An opinionated tool/framework built *on top* of Spring to quickly build stand-alone, production-grade applications. |
| **Configuration** | Requires extensive XML or Java-based configuration (DispatcherServlet, ViewResolver). | **Convention over Configuration**. Minimal to no configuration required. |
| **Server** | Requires external deployment to a server (like Tomcat or WebLogic) via a `.war` file. | Comes with **Embedded Servers** (Tomcat/Jetty). Runs directly via `java -jar`. |
| **Dependencies** | You must manually manage compatible dependency versions in your `pom.xml`. | Uses **Starter dependencies** (e.g., `spring-boot-starter-web`) which automatically resolve version conflicts. |

### 3. What are the internal annotations behind `@SpringBootApplication`?
**It is a convenience annotation that combines three crucial annotations:**
1. **`@SpringBootConfiguration`:** Marks the class as a configuration class (similar to `@Configuration`) allowing you to register extra beans.
2. **`@EnableAutoConfiguration`:** The core magic of Spring Boot. It tells Spring to automatically configure beans based on the jars present in the classpath (e.g., configuring Tomcat if `spring-boot-starter-web` is present).
3. **`@ComponentScan`:** Scans the package where the main class resides (and its sub-packages) to find and register other Spring components (`@Service`, `@Controller`, `@Repository`).

### 4. Explain the internal working of `HashMap`. What happens if `hashCode()` only returns a constant?
**Internal Working:**
- `HashMap` works on the principle of **Hashing**. It stores data in key-value pairs.
- It contains an array of `Node` (or `Entry`) objects called "buckets" (default size 16).
- **Put Operation:** When `put(key, value)` is called, it calculates the hash of the key (`hash(key.hashCode())`) to find the bucket index. If the bucket is empty, it stores the node. If not, it handles the collision by forming a LinkedList (or a Red-Black Tree in Java 8 if the list size exceeds 8).
- **Get Operation:** Calculates the hash to find the bucket, then uses `equals()` to traverse the list/tree and find the exact key.

**If `hashCode()` returns a constant (e.g., `return 1;`):**
- **Impact:** Every single key will generate the exact same hash and fall into the **same bucket** (Index 1).
- **Result:** The `HashMap` degrades completely into a single LinkedList (or Tree). 
- **Performance:** Time complexity for `get()` and `put()` drops from **O(1)** to **O(n)** (or **O(log n)** for trees), destroying the performance benefits of using a Map.

### 5. `String` vs `StringBuilder` vs `StringBuffer`
| Feature | `String` | `StringBuilder` | `StringBuffer` |
| :--- | :--- | :--- | :--- |
| **Mutability** | **Immutable** (Cannot be changed once created). | **Mutable** (Can be modified). | **Mutable** (Can be modified). |
| **Thread Safety**| Thread-safe (inherently due to immutability). | **Not Thread-safe**. | **Thread-safe** (Methods are synchronized). |
| **Performance** | Slow for string manipulations (creates new objects in String Pool). | **Fastest** (No synchronization overhead). | Slow (Synchronization overhead). |
| **Use Case** | Constants, configuration keys. | Single-threaded string concatenations. | Multi-threaded string manipulations. |

### 6. Explain OOP concepts with real-time examples.
- **Encapsulation:** Bundling data and methods that operate on the data into a single unit, hiding internal state. *Example:* A `BankAccount` class where `balance` is private, and can only be modified via public `deposit()` and `withdraw()` methods to prevent arbitrary assignments.
- **Inheritance:** A child class inherits properties and behavior from a parent class (reusability). *Example:* `ElectricCar` extends `Car`, inheriting methods like `startEngine()` but adding its own `chargeBattery()`.
- **Polymorphism:** The ability to take many forms (Compile-time via Method Overloading, Run-time via Method Overriding). *Example:* A generic `Payment` interface with a `processPayment()` method. Classes `CreditCardPayment` and `UPIPayment` implement it differently. The caller just calls `processPayment()` without knowing the exact type.
- **Abstraction:** Hiding complex implementation details and showing only the essential features. *Example:* Driving a car. You know pressing the accelerator moves the car (interface), but you don't need to know how the internal combustion engine works.

### 7. Exception Hierarchy in Java (Checked vs Unchecked)
- **`Throwable`** is the root class. It splits into two main branches: **`Error`** and **`Exception`**.
- **`Error`:** Critical JVM issues that the application cannot and should not try to catch (e.g., `OutOfMemoryError`, `StackOverflowError`).
- **`Exception`:** Exceptional conditions that an application should try to catch.
  - **Checked Exceptions:** Inherit directly from `Exception`. Checked at compile-time. Must be handled with `try-catch` or declared in `throws`. (e.g., `IOException`, `SQLException`).
  - **Unchecked Exceptions:** Inherit from `RuntimeException`. Not checked at compile-time. Usually indicate programming logic errors. (e.g., `NullPointerException`, `IndexOutOfBoundsException`).

### 8. Java 8 Features: Streams, Lambda, and Functional Interfaces.
**Core Features:**
- **Lambda Expressions:** Allow you to write more concise code by enabling functional programming. `(a, b) -> a + b;`
- **Functional Interfaces:** An interface with exactly one abstract method (e.g., `Runnable`, `Callable`, `Predicate`). Annotated with `@FunctionalInterface`.
- **Streams API:** A powerful way to process collections of objects in a declarative manner (filter, map, reduce).

**Coding Question: Write code to filter out loans with an incomplete status using Java 8.**
```java
List<Loan> allLoans = // ... list of loans
List<Loan> completeLoans = allLoans.stream()
    .filter(loan -> !"INCOMPLETE".equalsIgnoreCase(loan.getStatus()))
    .collect(Collectors.toList());
```

### 9. Garbage Collection in Java (How does it work, types of GC)
- **How it works:** GC automatically reclaims memory by deleting unreachable objects (objects with no active references) in the Heap. It operates on the **Mark and Sweep** algorithm (marks reachable objects, sweeps unreachable ones).
- **Generational Hypothesis:** Objects are split into Young Generation (Eden, S0, S1) and Old Generation. Most objects die young. Minor GC runs often on Young Gen; Major GC (Full GC) runs rarely on Old Gen.
- **Types of GC:** 
  1. Serial GC (Single thread, good for small apps).
  2. Parallel GC (Multiple threads, default in Java 8).
  3. G1 GC (Garbage First - Default in Java 9+. Splits heap into regions to minimize pause times).

### 10. Difference between `Comparable` and `Comparator`
- **`Comparable` (`java.lang`):** 
  - Modifies the class whose instances you want to sort.
  - Uses the `compareTo(Object obj)` method.
  - Defines the **default** or natural sorting order (e.g., sorting Students by Roll Number).
- **`Comparator` (`java.util`):** 
  - Does NOT modify the class being sorted. You create a separate class (or Lambda) for the logic.
  - Uses the `compare(Object obj1, Object obj2)` method.
  - Defines **custom** sorting orders (e.g., sorting Students by Name, or by Age, without altering the Student class).

### 11. Difference between `==` and `.equals()` in Java
- **`==` Operator:** Compares **memory references**. It checks if both objects point to the exact same location in memory. (For primitives, it compares the actual values).
- **`.equals()` Method:** Compares **content/value**. By default (in the `Object` class), `.equals()` behaves like `==`. However, classes like `String` and `Integer` override it to compare the actual character sequence or numeric value.

### 12. What is the difference between `@Component`, `@Service`, `@Repository`, and `@Controller`?
- **`@Component`:** The most generic Spring annotation. Marks a Java class as a bean so the component-scanning mechanism can pick it up.
- **`@Service`:** A specialization of `@Component`. It doesn't provide any new behavior over `@Component` but clarifies the class's intent (holds business logic).
- **`@Repository`:** A specialization of `@Component`. It automatically translates database-specific exceptions (like `SQLException`) into Spring’s unified `DataAccessException` hierarchy.
- **`@Controller`:** A specialization of `@Component`. Marks the class as a web controller, allowing Spring MVC to route HTTP requests to its `@RequestMapping` methods.

### 13. Explain Inversion of Control (IoC) and Dependency Injection (DI)
- **IoC (Inversion of Control):** A design principle where the framework (Spring) takes control of the program flow and object creation, rather than the developer manually instantiating objects using the `new` keyword.
- **DI (Dependency Injection):** The pattern used to implement IoC. Instead of a class creating its dependencies, the Spring IoC container injects the required dependencies into the class (via Constructor, Setter, or Field injection) when it is created.

### 14. How do you connect to multiple databases using Spring Boot?
**Steps:**
1. **Properties:** Define credentials for both databases in `application.yml`.
2. **DataSource Beans:** Create a `@Configuration` class to define multiple `DataSource` beans. Mark one as `@Primary`.
3. **EntityManagerFactory:** Configure a `LocalContainerEntityManagerFactoryBean` for each data source to specify which entity packages belong to which DB.
4. **TransactionManager:** Configure a `PlatformTransactionManager` for each database.
5. **Repositories:** Use `@EnableJpaRepositories` and point the `basePackages` to specific repository folders.

### 15. What are the SOLID Principles in Java?
- **S (Single Responsibility Principle):** A class should have only one reason to change (one responsibility).
- **O (Open/Closed Principle):** Classes should be open for extension but closed for modification.
- **L (Liskov Substitution Principle):** Subclasses should be substitutable for their base classes without breaking the application.
- **I (Interface Segregation Principle):** Many client-specific interfaces are better than one general-purpose interface (don't force a class to implement methods it doesn't need).
- **D (Dependency Inversion Principle):** Depend on abstractions (interfaces), not concrete implementations.

### 16. What is `Optional` in Java 8 and why use it?
- **Purpose:** `Optional` is a container object which may or may not contain a non-null value. It was introduced to prevent the infamous `NullPointerException`.
- **Usage:** Instead of returning `null` from a method when a value is absent, you return `Optional.empty()`. 
- **Benefits:** It forces the caller of the method to explicitly handle the case where a value might be missing using methods like `.isPresent()`, `.ifPresent()`, or `.orElse()`.

### 17. How does Spring Boot auto-configuration actually work?
- When `@EnableAutoConfiguration` is triggered, Spring Boot looks for a file named `META-INF/spring.factories` (or `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` in newer versions) inside all jars on the classpath.
- It loads the auto-configuration classes listed there and evaluates their `@Conditional` annotations (e.g., `@ConditionalOnClass`, `@ConditionalOnMissingBean`). 
- If the conditions are met (e.g., Tomcat classes are on the classpath), Spring automatically instantiates and wires those beans into the context.

### 18. What is the difference between `PUT` and `PATCH` methods in REST?
- **`PUT`:** Updates an **entire resource**. Must provide the complete payload. Missing fields are logically set to null/default. (Idempotent).
- **`PATCH`:** Used for **partial updates**. Modifies specific fields without affecting the rest of the resource.

### 19. What is the difference between a Controller and a Front Controller?
- **Controller:** Handles specific user requests and executes business logic (`UserController`).
- **Front Controller:** A centralized, single entry point that intercepts *all* incoming requests (`DispatcherServlet`). It handles routing, authentication, and logging before delegating to a specific Controller.

### 20. `HashMap` vs `Hashtable`
- **HashMap:** Not thread-safe, allows one `null` key and multiple `null` values, highly performant for single-threaded operations. Introduced in Java 1.2.
- **Hashtable:** Thread-safe (all methods are synchronized), does NOT allow any `null` keys or values, very slow due to blocking locks. Legacy class from Java 1.0.

### 21. `filter()` vs `map()` in Java 8 Streams
- **`filter()`:** Used for *conditional selection*. It takes a `Predicate` (a boolean condition). Elements that return `true` pass through; others are dropped. The resulting stream has fewer (or equal) elements, but they are of the exact same type.
- **`map()`:** Used for *transformation*. It takes a `Function`. It applies the function to every element, transforming them. The resulting stream has the exact same number of elements, but they can be of a completely different type.

### 22. 🔴 SCENARIO: Two users try to book the exact same train seat at the exact same millisecond. How do you handle this concurrency?
- **Solution 1: Optimistic Locking (Best for high-read/low-write):** Add a `@Version` column to the Seat entity in JPA. When User B tries to save, Hibernate checks the version. Since User A already incremented it, User B gets an `OptimisticLockException`, and we tell User B "Seat already booked".
- **Solution 2: Pessimistic Locking (Best for high collisions):** Use `SELECT ... FOR UPDATE` (in JPA: `LockModeType.PESSIMISTIC_WRITE`). The database locks the row physically for User A. User B's query blocks and waits until User A commits.

### 23. Explain `@Autowired` and its different modes.
- `@Autowired` tells Spring to resolve and inject a collaborating bean into our bean.
- **Modes of Injection:**
  1. **Constructor Injection:** (Recommended) Spring provides the dependency through the constructor. Perfect for mandatory dependencies and enables immutability (`final` fields).
  2. **Setter Injection:** Spring uses setter methods. Good for optional dependencies.
  3. **Field Injection:** Uses reflection to inject directly into fields. Highly discouraged as it makes the class hard to unit test without Spring and hides the class's dependencies.

### 24. What is `@Qualifier` in Spring?
- **Purpose:** Used in dependency injection to resolve ambiguity when multiple beans of the exact same type exist.
- **Example:**
```java
@Autowired
@Qualifier("cardPayment") // Explicitly tells Spring which bean to inject
private Payment paymentService;
```

### 25. Monolithic vs Microservices architecture (Pros/Cons)
- **Monolith:** Entire application is built and deployed as a single codebase/artifact. 
  - *Pros:* Simple to develop, test, and deploy initially.
  - *Cons:* Hard to scale independently, tight coupling, a bug anywhere can crash the whole app.
- **Microservices:** Application broken down into small, autonomous, loosely coupled services based on business domains.
  - *Pros:* Independent scaling, polyglot programming (use different tech for different services), fault isolation.
  - *Cons:* Extreme operational complexity (networking, distributed data, deployment).

### 26. What is Service Discovery (e.g., Eureka) and why is it needed?
- **Why it's needed:** In cloud environments (Microservices), the IP addresses of service instances change dynamically due to auto-scaling and failures. Hardcoding IPs is impossible.
- **How it works:** A central server (Eureka) acts as a dynamic phonebook. When a microservice boots up, it registers its IP with Eureka. When Service A needs to call Service B, it asks Eureka for Service B's current IP address.

### 27. 🔴 SCENARIO: Microservice A calls Microservice B. Microservice B is down or taking too long. How do you handle this to prevent cascading failure?
- Use the **Circuit Breaker Pattern** (e.g., Resilience4j).
- If Service B times out repeatedly, the circuit breaker "opens" and stops sending requests to Service B entirely (failing fast). 
- Instead of the thread hanging, Service A instantly returns a **Fallback Response** (e.g., cached data or a default error message) to the user, preventing thread exhaustion in Service A.

### 28. Difference between `Thread` and `Process`
- **Process:** A program in execution. It is heavyweight and has its own isolated memory space. Processes do not share memory.
- **Thread:** A lightweight sub-process; the smallest unit of processing. Multiple threads reside within the same process and share that process's memory space (Heap), making context switching much faster but requiring synchronization.

### 29. Explain `@RestController` vs `@Controller`
- **`@Controller`:** Used for traditional Spring MVC applications. The methods return a `String` which the `ViewResolver` resolves to an HTML template (like Thymeleaf or JSP).
- **`@RestController`:** Used for RESTful web services. It is a convenience annotation that combines `@Controller` and `@ResponseBody`. It bypasses the `ViewResolver` and automatically serializes the returned object into JSON/XML using HttpMessageConverters.

### 30. Explain Kafka Architecture (Topics, Partitions, Brokers)
- **Broker:** A single Kafka server node. A Kafka cluster is made of multiple brokers.
- **Topic:** A logical channel where Producers publish messages and Consumers read them (like a table in a database).
- **Partition:** A Topic is split into multiple Partitions distributed across Brokers. This allows Kafka to scale horizontally. Messages in a partition are strictly ordered and immutable.
- **Consumer Group:** A group of consumers reading from a topic. Each partition is consumed by only one consumer within the group, enabling load balancing.

### 31. Default vs Static methods in Java 8 Interfaces
- Before Java 8, interfaces could only have abstract methods. 
- **Default Methods:** Allow you to add new methods to an interface with a default implementation without breaking the existing classes that implement the interface (Backward compatibility).
- **Static Methods:** Similar to default methods, but they belong to the interface class itself, not the instance. They cannot be overridden by implementing classes. Used for utility methods.

### 32. Abstract Class vs Interface
| Feature | Abstract Class | Interface |
| :--- | :--- | :--- |
| **Methods** | Can have abstract and concrete methods. | Abstract methods. Java 8+ allows `default` and `static`. |
| **Variables** | Can have instance variables (state). | Implicitly `public static final` (constants). |
| **Multiple Inheritance**| Can only extend ONE abstract class. | Can implement MULTIPLE interfaces. |

### 33. Thread Lifecycle and States in Java
1. **NEW:** Thread created but `start()` not called yet.
2. **RUNNABLE:** Thread is executing or ready to execute in the JVM.
3. **BLOCKED:** Thread is waiting to acquire a monitor lock (e.g., waiting to enter a `synchronized` block).
4. **WAITING:** Thread is waiting indefinitely for another thread to perform an action (`Object.wait()`, `Thread.join()`).
5. **TIMED_WAITING:** Waiting for a specified time (`Thread.sleep(1000)`).
6. **TERMINATED:** Thread has finished execution.

### 34. 🔴 SCENARIO: You are fetching 100,000 records from a database in a Spring Boot application, and it causes an `OutOfMemoryError`. How do you fix it?
- **Do not load everything into memory at once.**
- **Solution 1 (Pagination):** Use Spring Data JPA `Pageable` to fetch records in chunks (e.g., 1000 at a time).
- **Solution 2 (Streaming):** Use Java 8 `Stream` with JPA. Annotate the repository method with `@Query` and return a `Stream<Entity>`. Wrap the call in a `@Transactional(readOnly = true)` block. The database will push records one by one without blowing up the Heap.

### 35. `volatile` keyword vs `synchronized` block
- **`volatile`:** Applied to variables. It prevents threads from caching the variable locally. It forces all reads and writes to go straight to main memory, ensuring visibility across threads. It does NOT guarantee atomicity (e.g., `count++` can still cause race conditions).
- **`synchronized`:** Applied to methods or code blocks. It ensures mutually exclusive access (only one thread can execute it at a time), guaranteeing both visibility and atomicity.

### 36. Explain `map()` vs `flatMap()` in Java Streams
- **`map()`:** Used for data transformation. It takes an element and applies a function to it, returning a single new element (1-to-1 mapping). If you map over a list of lists, you get a `Stream<List<T>>`.
- **`flatMap()`:** Used for transformation AND flattening. If you map over a list of lists, it flattens the nested structures so you get a single continuous stream of elements `Stream<T>` (1-to-many mapping).

### 37. What is the difference between `Statement` and `PreparedStatement` in JDBC?
- **`Statement`:** Used to execute normal SQL queries. Vulnerable to SQL Injection. It compiles the SQL query every time it is executed, which is slower for repeated executions.
- **`PreparedStatement`:** Used to execute parameterized SQL queries. Protects against SQL Injection by treating user input strictly as data, not executable code. Pre-compiles the query on the database, making it much faster for repeated executions.

### 38. What is the difference between `@Primary` and `@Qualifier`?
- **`@Qualifier`:** Specifically tells Spring exactly *which* bean name to inject when multiple beans of the same type exist. It is applied at the injection point (`@Autowired`).
- **`@Primary`:** Indicates that a specific bean should be given preference when multiple beans are candidates to be autowired to a single-valued dependency. It is applied at the bean definition level.

### 39. 💡 DSA: Given a string, write a program to find the first non-repeating character.
```java
public char firstNonRepeatingChar(String str) {
    Map<Character, Integer> counts = new LinkedHashMap<>();
    for (char c : str.toCharArray()) {
        counts.put(c, counts.getOrDefault(c, 0) + 1);
    }
    for (Map.Entry<Character, Integer> entry : counts.entrySet()) {
        if (entry.getValue() == 1) return entry.getKey();
    }
    return '_';
}
```

### 40. How to prevent SQL Injection in Java?
- Always use **`PreparedStatement`** instead of `Statement`. `PreparedStatement` automatically escapes special characters.
- If using an ORM like Hibernate or Spring Data JPA, avoid concatenating strings to form HQL/JPQL queries; always use named parameters (`:username`) or positional parameters (`?1`).

### 41. `wait()` vs `sleep()` in multithreading
- **`wait()`:** Called on an `Object`. Causes the current thread to wait until another thread calls `notify()` or `notifyAll()` on that object. Critically, it **releases the lock** it holds on the object.
- **`sleep()`:** Called on the `Thread` class. Causes the currently executing thread to pause for a specified duration. It **does NOT release any locks** it holds.

### 42. `ArrayList` vs `LinkedList`
- **ArrayList:** Internally uses a dynamic array. Read operations (`get(index)`) are extremely fast `O(1)`. Insert/Delete operations in the middle are slow `O(n)` because elements must be shifted.
- **LinkedList:** Internally uses a doubly-linked list. Read operations are slow `O(n)` because it must traverse from the start. Insert/Delete operations are fast `O(1)` (once the node is found) because no shifting is required.

### 43. 🔴 SCENARIO: In a microservice architecture, how do you trace a request that travels across 5 different services?
- Use **Distributed Tracing** (e.g., Spring Cloud Sleuth, Zipkin, OpenTelemetry).
- The API Gateway generates a unique `Trace ID` for the initial request.
- Every microservice propagates this same `Trace ID` (usually via HTTP headers like `X-B3-TraceId`) to the downstream services. 
- You can then go into Zipkin/Kibana and search for that `Trace ID` to see the entire lifecycle and timing of the request across all 5 services.

### 44. CI/CD, DevOps, and Hibernate Integration basics.
- **CI/CD:** Continuous Integration (automated testing on commit) and Continuous Deployment (automated deployment to production) using tools like Jenkins/Docker.
- **Hibernate:** Java ORM framework mapping `@Entity` classes directly to relational DB tables, eliminating boilerplate JDBC.

### 45. What is `Optional.orElseGet()` vs `Optional.orElse()`?
- Both provide a fallback value if the `Optional` is empty.
- **`.orElse(defaultValue)`:** The `defaultValue` is evaluated *immediately*, regardless of whether the Optional is empty or not. (Can cause performance hits if the default value is computationally expensive to generate).
- **`.orElseGet(() -> defaultValue)`:** The Supplier lambda is *only* executed if the Optional is actually empty (Lazy evaluation).

### 46. What is `@Bean` vs `@Component` in Spring?
- **`@Component`:** A class-level annotation. You use it when you own the source code and want Spring to automatically detect and register the class during classpath scanning.
- **`@Bean`:** A method-level annotation. Used inside a `@Configuration` class. You use it when you need to explicitly declare and configure a bean, especially for third-party library classes where you cannot simply add a `@Component` annotation to their source code.

### 47. Difference between `final`, `finally`, and `finalize`?
- **`final`:** Keyword used to apply restrictions (a `final` class cannot be subclassed, a `final` method cannot be overridden, a `final` variable cannot be reassigned).
- **`finally`:** Block used with `try-catch` to execute crucial code (like closing DB connections) regardless of whether an exception is thrown or not.
- **`finalize()`:** A method in the `Object` class called by the Garbage Collector just before an object is destroyed. (Deprecated in Java 9+ due to unpredictability and performance issues).

### 48. 🔴 SCENARIO: You have a `ConcurrentModificationException` while iterating over a list and removing items. How do you resolve it?
- **The Cause:** You are using a standard `for-each` loop and calling `list.remove()` which breaks the internal iterator count.
- **Solution 1:** Use `Iterator.remove()` explicitly.
- **Solution 2:** Use Java 8 `list.removeIf(condition)`.
- **Solution 3:** If multi-threaded, use a thread-safe collection like `CopyOnWriteArrayList`.

### 49. Difference between `@RequestParam` and `@PathVariable`
- **`@RequestParam`:** Used to extract query parameters from the URL (e.g., `/users?id=5`). Mostly used for filtering or pagination.
- **`@PathVariable`:** Used to extract data right from the URI path itself (e.g., `/users/5`). Used to identify specific resources in RESTful APIs.

### 50. React: What is the `useEffect` hook and its dependency array?
- `useEffect` is a React Hook that lets you perform side effects in functional components (like fetching data, manually manipulating the DOM, setting up subscriptions).
- **Dependency Array:** The second argument.
  - `[]` (Empty): Effect runs ONLY once after the initial render (like `componentDidMount`).
  - `[prop1, prop2]`: Effect runs after initial render AND whenever `prop1` or `prop2` change.
  - *No array passed*: Effect runs after *every single render* (usually a bad idea).

### 51. Singleton Design Pattern - How to make it thread-safe?
- **Singleton:** Ensures a class has only one instance and provides a global point of access to it.
- **Thread Safety (Double-Checked Locking):**
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

### 52. 💡 DSA: Two Sum Problem. Given an array of integers, return indices of the two numbers such that they add up to a target.
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
    return new int[] {};
}
```

### 53. What is Spring Boot Actuator?
- **Purpose:** Actuator provides built-in HTTP endpoints to monitor and manage a Spring Boot application in production (e.g., `/actuator/health`, `/actuator/metrics`).

### 54. Explain the Factory Design Pattern.
- **Creational Pattern:** Provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created.
- Instead of calling `new Notification()` directly, you call `NotificationFactory.createNotification("EMAIL")`. This hides the instantiation logic and makes the system easily extensible.

### 55. How do Microservices communicate with each other?
- **Synchronous (Blocking):** REST APIs (RestTemplate/WebClient), Feign Client, gRPC.
- **Asynchronous (Non-blocking):** Message Brokers (Kafka, RabbitMQ) via event-driven architecture.

### 56. 💡 DSA: Write a program to reverse a Linked List.
```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;
    while (curr != null) {
        ListNode nextTemp = curr.next;
        curr.next = prev;
        prev = curr;
        curr = nextTemp;
    }
    return prev;
}
```

### 57. SQL: Difference between `INNER JOIN` and `LEFT JOIN`
- **`INNER JOIN`:** Returns only the rows where there is a match in *both* tables based on the join condition.
- **`LEFT JOIN` (Left Outer Join):** Returns ALL rows from the left table, and the matched rows from the right table. If there is no match, the right side will contain `NULL` values.

### 58. SQL: Explain `GROUP BY` and `HAVING` clause.
- **`GROUP BY`:** Used to group rows that have the same values into summary rows, typically to perform aggregate functions on them like `COUNT()`, `MAX()`, `SUM()`.
- **`HAVING`:** Added to SQL because the `WHERE` keyword cannot be used with aggregate functions. `HAVING` filters the results *after* they have been grouped by the `GROUP BY` clause.

### 59. Explain the Saga Design Pattern. How do you handle failures?
- **Context:** Standard ACID locks don't work across multiple microservice databases.
- **Saga Pattern:** A global transaction is broken into a sequence of local transactions.
- **Handling Failures:** If a step fails, the Saga triggers **Compensating Transactions** to undo the previous steps (e.g., if Payment fails, trigger Cancel Order).

### 60. What is `@Valid` and `@Validated` in Spring Boot?
- **`@Valid`:** A standard Java (JSR-380) annotation used to validate complex objects (like a `@RequestBody` DTO). 
- **`@Validated`:** A Spring-specific variant of `@Valid`. It supports group validation (validating different fields based on the specific scenario, like create vs update).

### 61. What is the difference between `@Mock` and `@InjectMocks` in Mockito?
- **`@Mock`:** Creates a fake/dummy instance of a class (mock object) whose behavior you can stub (control what methods return).
- **`@InjectMocks`:** Creates a real instance of the class you are trying to test, and automatically injects the objects created with `@Mock` into this class.

### 62. 🔴 SCENARIO: How would you design a rate limiter for an API in Spring Boot?
- **Purpose:** To prevent abuse or DDoS attacks by limiting a user to X requests per minute.
- **Implementation:** 
  - Use an API Gateway (like Spring Cloud Gateway) which has built-in Redis Rate Limiting.
  - Or, implement **Bucket4j** locally using an interceptor or filter, backing it with Redis so the rate limits are synchronized across all your load-balanced instances.

### 63. What is a Circuit Breaker? (Resilience4j)
- **Problem:** To prevent cascading failures when a downstream service is dead.
- **Solution:** Circuit breaker monitors calls. If failure rate exceeds a threshold, it "Opens" the circuit, failing fast or returning a fallback instead of blocking threads.

### 64. Explain `reduce()` in Java 8 Streams.
- **Purpose:** It is a terminal operation used to combine all elements of a stream into a single result (e.g., sum all numbers, concatenate strings, find max).
- It takes two arguments: an `identity` (an initial starting value) and a `BinaryOperator` (a lambda detailing how to combine two elements).

### 65. What is an API Gateway? Why use it instead of direct client calls?
- **What it is:** A single entry point for external clients.
- **Responsibilities:** Routing (URL to Microservice), Security (JWT validation), Rate limiting, and CORS.

### 66. Describe the 12-Factor App methodology.
- A set of best practices for building modern, scalable cloud-native microservices.
- **Key Factors:** Codebase (one repo per service), Dependencies (explicitly declare them), Config (store config in the environment, not code), Backing services (treat databases as attached resources), Stateless processes (share nothing), Disposability (fast startup, graceful shutdown).

### 67. 💡 DSA: Valid Parentheses. Check if a string of brackets is valid.
```java
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    for (char c : s.toCharArray()) {
        if (c == '(') stack.push(')');
        else if (c == '{') stack.push('}');
        else if (c == '[') stack.push(']');
        else if (stack.isEmpty() || stack.pop() != c) return false;
    }
    return stack.isEmpty();
}
```

### 68. How does `@Transactional` work, and what are its propagation levels?
- **How it works:** Spring wraps the method in a proxy using AOP. It opens a DB connection, begins a transaction, executes logic, and automatically commits (or rolls back if a `RuntimeException` is thrown).
- **Propagation Levels (Key ones):**
  - `REQUIRED` (Default): Joins the existing transaction if one exists, otherwise creates a new one.
  - `REQUIRES_NEW`: Always suspends the existing transaction and starts a completely new, independent one.

### 69. Explain the Spring Bean Lifecycle
1. **Instantiation:** Spring container creates the object using the constructor.
2. **Populate Properties:** Dependencies are injected (DI).
3. **Aware Interfaces:** (e.g., `BeanNameAware`, `ApplicationContextAware`) are invoked.
4. **Pre-Initialization:** `BeanPostProcessor.postProcessBeforeInitialization()` runs.
5. **Initialization:** Custom init methods (`@PostConstruct`, `InitializingBean.afterPropertiesSet()`) run.
6. **Post-Initialization:** `BeanPostProcessor.postProcessAfterInitialization()` runs (where AOP proxies are created).
7. **Destruction:** When context closes, `@PreDestroy` and `DisposableBean.destroy()` are called.

### 70. What are the core components of Spring Security?
- **`SecurityContextHolder`:** The most fundamental object. It stores the details of the currently authenticated principal (user) inside a `ThreadLocal`.
- **`AuthenticationManager`:** The interface responsible for validating user credentials. Its default implementation (`ProviderManager`) delegates to `AuthenticationProvider`s.
- **`UserDetailsService`:** An interface you implement to load user-specific data from your database (e.g., loading username, password, and roles) so the `AuthenticationManager` can compare it against the user's login attempt.

### 71. REST vs SOAP web services
- **REST:** Architectural style, uses HTTP methods (GET/POST), lightweight, relies on JSON (mostly).
- **SOAP:** Protocol, heavily structured, uses XML strictly, has strict contracts (WSDL), and built-in security/transaction standards (WS-Security).

### 72. 💡 DSA: Find the missing number in an array containing numbers from 1 to N.
```java
public int missingNumber(int[] nums) {
    int n = nums.length;
    int expectedSum = n * (n + 1) / 2; // Sum formula
    int actualSum = 0;
    for (int num : nums) actualSum += num;
    return expectedSum - actualSum;
}
```

### 73. SQL: Difference between `View` and `Materialized View`?
- **View:** A virtual table based on a SELECT query. It does not store data physically. Every time you query the view, the underlying SQL is executed dynamically.
- **Materialized View:** Physically stores the query results on the disk. Querying it is incredibly fast, but the data must be refreshed periodically to stay updated with the base tables.

### 74. Explain the OAuth 2.0 flow
- **OAuth 2.0** is an authorization framework, not an authentication protocol.
- **Flow (Authorization Code Grant):**
  1. User clicks "Login with Google".
  2. Client app redirects user to Google's Authorization Server.
  3. User logs in and grants permission.
  4. Google redirects back to the client with an **Authorization Code**.
  5. The Client securely exchanges the Code for an **Access Token** via a backend call.
  6. Client uses the Access Token to fetch resources from the Resource Server.

### 75. What is the difference between `@Entity` and `@Table`?
- **`@Entity`:** A mandatory JPA annotation applied to a class to mark it as a persistent entity that maps to a database table.
- **`@Table`:** An optional annotation used to specify exact database details. If omitted, the table name defaults to the class name. With `@Table`, you can customize the `name`, `schema`, or define `uniqueConstraints`.

### 76. What is `@Async` in Spring Boot?
- An annotation that tells Spring to execute a method in a separate thread from the caller (Asynchronous execution).
- **Requirements:** You must enable it using `@EnableAsync` on a configuration class. The method should return `void` or `CompletableFuture`.

### 77. Docker Containers vs Virtual Machines
- **VMs:** Heavyweight. Each VM includes a full Guest Operating System on top of a Hypervisor. Slow to boot.
- **Docker Containers:** Lightweight. Containers share the Host OS kernel. Extremely fast to boot, uses far fewer resources.

### 78. 💡 DSA: Check if a given string is a palindrome.
```java
public boolean isPalindrome(String str) {
    int left = 0, right = str.length() - 1;
    while (left < right) {
        if (str.charAt(left) != str.charAt(right)) return false;
        left++; right--;
    }
    return true;
}
```

### 79. What is `OutOfMemoryError` vs `StackOverflowError`?
- **`OutOfMemoryError`:** Occurs when the JVM Heap space is full and the Garbage Collector cannot free up enough space to create new objects.
- **`StackOverflowError`:** Occurs when the JVM Stack memory is full. This almost exclusively happens due to infinite, uncontrolled recursion (a method calling itself without a base case).

### 80. What is the Spring Security Filter Chain?
- Spring Security is entirely based on a chain of Servlet Filters.
- When an HTTP request arrives, it passes through a sequence of filters (`UsernamePasswordAuthenticationFilter`, `BasicAuthenticationFilter`, `ExceptionTranslationFilter`, etc.) before ever reaching your Controller. 
- These filters handle authentication (who are you?), authorization (do you have access?), and CSRF protection.

### 81. Explain the Observer Design Pattern.
- **Behavioral Pattern:** Defines a one-to-many dependency between objects. When the subject (the one being observed) changes state, all its dependents (observers) are automatically notified and updated.
- Examples: Event Listeners in UI, or a Pub/Sub mechanism.

### 82. What is Cross-Site Scripting (XSS) and how to prevent it?
- **XSS:** An attack where a malicious user injects executable client-side scripts (usually JavaScript) into a web page viewed by other users. This script can steal session cookies or manipulate the DOM.
- **Prevention:** Always sanitize/encode user input before rendering it on the page. Use Content Security Policy (CSP) headers. Modern frameworks like React automatically escape string variables preventing basic XSS.

### 83. 🔴 SCENARIO: You need to deploy a Spring Boot application to AWS/Cloud. What is the standard process?
- Write a `Dockerfile` to package the Spring Boot `.jar` with a JRE into an image.
- Push the image to a Container Registry (e.g., AWS ECR, Docker Hub) using a Jenkins/GitLab CI pipeline.
- Define a deployment configuration (e.g., Kubernetes Deployment/Service YAMLs or AWS ECS Task Definition).
- Trigger a CD pipeline to apply the configuration, letting the cloud pull the image and spin up the containers behind a Load Balancer.

### 84. Explain JWT (JSON Web Token) and how it is used in Spring Security.
- **JWT Structure:** Header, Payload, Signature.
- **Stateless:** The server validates the mathematical signature using a secret key without looking up sessions in a DB.
- **Flow:** User logs in -> Server gives JWT -> Client sends JWT in `Authorization` header on every request -> Spring `OncePerRequestFilter` intercepts and validates.

### 85. What is a Distributed Cache (e.g., Redis, Hazelcast)?
- In a microservices environment, multiple instances of a service are running behind a load balancer. If you use an in-memory cache (like `ConcurrentHashMap`), Service Instance A won't know about cached data in Service Instance B.
- A Distributed Cache (like Redis) stores the cached data externally in a centralized, highly-available, lightning-fast NoSQL store, allowing all service instances to share the same cache pool.

### 86. What is the `@Profile` annotation in Spring Boot?
- Profiles provide a way to segregate parts of your application configuration and make it available only in certain environments. 
- Example: `@Profile("dev")` means the bean will only be loaded when the active profile is "dev". You activate it via `spring.profiles.active=dev` in your `application.properties`.

### 87. React: What is `useRef` and when do you use it?
- A React hook that returns a mutable ref object whose `.current` property is initialized to the passed argument.
- **Uses:** 
  1. Accessing DOM elements directly (like focusing an input field).
  2. Storing a mutable value that does *not* cause a re-render when updated (unlike `useState`).

### 88. 💡 DSA: Detect a cycle in a Linked List (Floyd's Tortoise and Hare).
```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;          // moves 1 step
        fast = fast.next.next;     // moves 2 steps
        if (slow == fast) return true; // cycle detected
    }
    return false;
}
```

### 89. What is `@ControllerAdvice` vs `@ExceptionHandler`?
- **`@ExceptionHandler`:** Handled exceptions locally within a single specific `@Controller` class.
- **`@ControllerAdvice`:** A global interceptor. It allows you to write `@ExceptionHandler` methods once and have them apply to all Controllers across the entire application, centralizing error handling logic.

### 90. Difference between `@WebMvcTest` and `@SpringBootTest`?
- **`@SpringBootTest`:** Starts the full Spring Application Context (creates every single bean). Used for heavy integration testing.
- **`@WebMvcTest`:** A "sliced" test. It only starts the web layer (Controllers, Filters, Converters) and ignores Services and Repositories. It's used to unit test Controllers quickly, typically relying on `@MockBean` for dependencies.

### 91. What is CORS and how to enable it in Spring Boot?
- **CORS (Cross-Origin Resource Sharing):** A security mechanism implemented by browsers that restricts web pages from making requests to a different domain than the one that served the web page.
- **Spring Boot Enablement:** Can be enabled locally using the `@CrossOrigin` annotation on a Controller, or globally by implementing `WebMvcConfigurer` and overriding `addCorsMappings()`.

### 92. What is the `@Value` annotation in Spring Boot?
- It is used to inject property values from `application.properties` or `application.yml` directly into fields in your Spring Beans.
- Example: `@Value("${server.port}") private int port;`

### 93. Explain Exception Translation in Spring (`@Repository`).
- Framework-specific persistence APIs (like Hibernate, JDBC, JPA) throw wildly different, specific exceptions.
- The `@Repository` annotation triggers Spring to automatically catch these low-level native exceptions (like `SQLException`) and translate them into Spring's unified, unchecked `DataAccessException` hierarchy.

### 94. React/Angular Basics: Virtual DOM vs Real DOM?
- **Virtual DOM (React):** A lightweight, in-memory representation of the Real DOM. React compares the new Virtual DOM with the old one (Diffing) and updates the Real DOM in one batch, drastically improving performance.

### 95. What is the N+1 Query Problem in Hibernate/JPA and how do you solve it?
- **The Problem:** Fetching a list of entities (1 query) and iterating through them to fetch a lazily-loaded association, triggering N additional queries.
- **The Solution:** Use `JOIN FETCH` in JPQL or `@EntityGraph` to eagerly load the association in a single SQL join query.

### 96. 🔴 SCENARIO: You are uploading a 5GB file through a REST API, and it fails midway. How do you optimize/handle this?
- Don't upload 5GB in a single request. 
- Implement **Chunked Resumable Uploads**: Slice the file in the frontend into 5MB chunks. Send chunks sequentially with a chunk index.
- If a failure occurs, query the backend for the last successfully saved chunk, and resume the upload from that index, bypassing the need to re-upload the entire 5GB.

### 97. Explain the Bulkhead Pattern in Microservices.
- Just like a ship is divided into multiple watertight compartments (bulkheads) so that if one compartment floods, the ship doesn't sink.
- In Microservices, you isolate resources (like threading pools or connection pools) for different services. If Service A fails and exhausts its thread pool, the isolated thread pool for Service B remains completely unaffected.

### 98. Explain CQRS Pattern (Command Query Responsibility Segregation)
- A Microservices pattern that separates the data mutation operations (Commands: `POST`, `PUT`, `DELETE`) from the data reading operations (Queries: `GET`).
- Often, this involves using completely different databases for reading and writing (e.g., writing to PostgreSQL, reading from Elasticsearch). 
- Event Sourcing or Kafka is used to keep the read database synchronized with the write database.

### 99. JPA: Difference between `save()` and `persist()`?
- **`persist()`:** Makes a transient instance persistent. However, it does not guarantee that the identifier value will be assigned immediately. It returns `void`.
- **`save()`:** A Spring Data JPA/Hibernate method that will check if the entity has an ID. If it does not, it calls `persist()`. If it does, it calls `merge()` (updates). It returns the saved entity.

### 100. What is `CompletableFuture.allOf()`?
- A static method used in asynchronous programming. It takes an array of multiple `CompletableFuture`s and returns a single `CompletableFuture<Void>` that completes only when *every single one* of the provided futures completes. Used for parallel execution synchronization.

### 101. Difference between `FetchType.LAZY` and `FetchType.EAGER`?
- **LAZY:** Entity not loaded from DB until getter is called. Default for `@OneToMany`.
- **EAGER:** Entity loaded immediately via SQL JOIN. Default for `@OneToOne`.

### 102. Explain the Proxy Pattern in Spring AOP.
- When you use annotations like `@Transactional` or `@Async`, Spring does not actually inject the real instance of your class into the controller.
- Instead, it dynamically creates a **Proxy object** (a wrapper) that implements the same interfaces. The proxy intercepts the method call, starts a database transaction (the AOP advice), delegates the actual logic to your real class, and then commits the transaction.

### 103. Difference between `Runnable` and `Callable` interfaces?
- **Runnable:** Has a `run()` method. Cannot return a result. Cannot throw checked exceptions.
- **Callable:** Has a `call()` method. Returns a `Future<T>` result. Can throw checked exceptions.

### 104. Explain Dependency Injection Types and Spring Bean Scopes.
- **Injection Types:** Constructor Injection (Best practice, immutable), Setter Injection, Field Injection (Discouraged).
- **Scopes:** `Singleton` (Default, 1 per app context), `Prototype` (New instance every time), `Request` (1 per HTTP request).

### 105. What are Memory Leaks in Java? How do they happen and how do you prevent them?
- **What it is:** When objects are no longer used by the application but the Garbage Collector cannot remove them because they are still actively referenced.
- **Causes:** Storing data in un-cleared static Collections (like a `static HashMap`), unclosed database connections, or ThreadLocal variables not being cleaned up.
- **Prevention:** Always use `finally` blocks or try-with-resources to close connections, explicitly set useless static references to `null`, and use profiling tools like VisualVM to monitor heap usage.

### 106. Can you write a `try-catch` block without a `catch` block?
- Yes, provided there is a `finally` block or you are using `try-with-resources`.
- Example: `try { /* ... */ } finally { /* cleanup */ }` is perfectly valid. The exception is propagated up, but the cleanup is executed first.

### 107. What are the common Design Patterns you have used in your project?
- **Singleton:** Ensuring only one DB connection pool exists.
- **Builder Pattern:** Used for creating complex objects (`@Builder` in Lombok).
- **Strategy Pattern:** Defining interchangeable algorithms (e.g., `CreditCardPayment`, `UPIPayment` implementing `PaymentStrategy`).

### 108. Maven vs Gradle
- **Maven:** Uses XML for configuration (`pom.xml`). Very structured, relies heavily on standard conventions and phases. Slower build times but universally understood.
- **Gradle:** Uses a Groovy/Kotlin DSL (`build.gradle`). Highly flexible, script-based, and incredibly fast due to advanced incremental builds and a build cache.

### 109. What is a `ThreadDump` and how to capture it?
- A snapshot of exactly what every single thread in the JVM is executing at a specific moment. Vital for diagnosing deadlocks, high CPU usage, or infinite loops.
- **Capture tools:** Use command-line tools like `jstack <pid>`, or JVM monitoring tools like VisualVM, JConsole, or trigger it natively in Unix with `kill -3 <pid>`.

### 110. `ConcurrentHashMap` vs `HashMap` vs `Collections.synchronizedMap()`?
- **ConcurrentHashMap:** Thread-safe, highly performant. Uses Lock Stripping (locks only the specific segment being written to).
- **SynchronizedMap:** Thread-safe, but locks the *entire* map for every operation (poor performance).

### 111. React: What is a Fragment (`<>...</>`)?
- A component in React that lets you group a list of children elements without adding extra nodes (like wrapper `<div>`s) to the actual DOM. It prevents layout breaks and improves DOM performance.

### 112. CompletableFuture vs Traditional Threads (ExecutorService).
- **ExecutorService (`Future.get()`):** Blocking. The main thread waits helplessly for the result.
- **CompletableFuture (Java 8):** Non-blocking, callback-driven. Allows chaining asynchronous tasks using `.thenApply()`, `.thenAccept()`.

### 113. React: What is Redux / Context API? (State Management)
- In large frontend applications, passing props down 10 layers of components (Prop Drilling) becomes a nightmare.
- **Redux / Context API:** Provides a centralized, global "Store" for the application state. Any component, regardless of how deep it is in the tree, can directly read from or dispatch updates to this global store.

### 114. `HashSet` vs `TreeSet`
- **HashSet:** Backed by a HashMap. No ordering guarantees. Constant time `O(1)` for basic operations (add, remove, contains). Allows one `null` value.
- **TreeSet:** Backed by a TreeMap (Red-Black tree). Elements are always sorted (natural ordering or Custom Comparator). Slower `O(log n)` performance. Does NOT allow `null` values.

### 115. React: What are Higher-Order Components (HOC)?
- An advanced technique in React for reusing component logic.
- An HOC is a function that takes a component as an argument and returns a new component with injected additional props or logic (e.g., wrapping a component in a `withAuthorization` HOC to verify login before rendering).

### 116. What is the `Predicate` functional interface in Java 8?
- It is a built-in functional interface located in `java.util.function`.
- It takes a single generic argument `T` and returns a `boolean` (by executing the `test(T t)` method). It is heavily used in the Stream API's `.filter()` method.

### 117. React/Angular: What are Component Lifecycle hooks?
- Components go through a lifecycle: Mounting, Updating, and Unmounting.
- In modern React (Functional Components), this is handled by the `useEffect` hook, which allows you to execute logic when the component renders, when dependencies change, or run cleanup logic when the component unmounts.

### 118. What is `Stream.generate()` and `Stream.iterate()`?
- Methods to generate infinite streams.
- **`generate(Supplier)`:** Generates an infinite stream where each element is created by the provided Supplier (e.g., generating random numbers).
- **`iterate(seed, UnaryOperator)`:** Generates an infinite sequential stream starting with the seed, where each subsequent element is generated by applying the operator to the previous element (e.g., generating even numbers).

### 119. SQL ACID properties
- **Atomicity:** All or nothing (Transaction completely succeeds or completely rolls back).
- **Consistency:** Database moves from one valid state to another.
- **Isolation:** Concurrent transactions don't interfere with each other.
- **Durability:** Committed data is permanently saved, surviving system crashes.

### 120. Clustered vs Non-Clustered Indexes in SQL
- **Clustered Index:** Determines the physical sorting order of the data on the disk. A table can only have **one** clustered index (usually the Primary Key). Extremely fast.
- **Non-Clustered Index:** Does not alter physical sorting. It creates a separate logical structure (a B-Tree) containing the index key and a pointer to the actual data row. A table can have **multiple** non-clustered indexes.

### 121. API Gateway vs Load Balancer
- **Load Balancer:** Distributes incoming network traffic blindly across a group of backend servers at the IP/TCP layer (Layer 4) to ensure no single server gets overwhelmed.
- **API Gateway:** Operates at the application layer (Layer 7). It routes requests intelligently based on the URL path, handles JWT authentication, rate-limiting, and can combine multiple microservice responses into a single client payload.

### 122. What is the difference between `System.out.println` and `Logger`?
- **`System.out.println`:** Writes synchronously directly to the console. It blocks the executing thread, drastically slowing down application performance. Cannot be disabled dynamically or filtered by severity.
- **Logger (e.g., SLF4J/Logback):** Thread-safe, non-blocking (can run asynchronously). Supports dynamic logging levels (`DEBUG`, `INFO`, `ERROR`), and allows routing output to external files, ELK stacks, or databases automatically based on configuration.

### 123. Explain `@ModelAttribute` in Spring MVC.
- Used to bind request parameters (from an HTML form submission) to an object, or to add objects to a Spring MVC Model before rendering a View.

### 124. SQL: Primary Key vs Unique Key
- **Primary Key:** Uniquely identifies a row. Does NOT allow `null` values. Only ONE primary key is allowed per table. By default, it creates a Clustered Index.
- **Unique Key:** Ensures column values are unique. Allows ONE `null` value. A table can have MULTIPLE unique keys. By default, it creates a Non-Clustered Index.

### 125. Difference between `Session` and `Cookie`
- **Cookie:** A small piece of data stored purely on the client-side (in the browser). Usually has an expiration date.
- **Session:** A state-management mechanism where the data is stored on the server-side. The server generates a unique `Session ID` and sends it to the browser as a Cookie, linking the client to the server-side data.

### 126. React: Controlled vs Uncontrolled Components
- **Controlled:** The component's state (e.g., an input field's value) is driven entirely by React state (`useState`). Every keystroke updates the state, which re-renders the input.
- **Uncontrolled:** The component manages its own internal state via the actual DOM. You read values using a `ref` only when needed (e.g., on form submit).

### 127. Differences between `List`, `Set`, and `Map`
- **List:** Ordered collection, allows duplicates. (e.g., `ArrayList`).
- **Set:** Unordered collection, does NOT allow duplicates. (e.g., `HashSet`).
- **Map:** Stores key-value pairs. Keys are unique, values can be duplicated. (e.g., `HashMap`).

### 128. Describe the Strangler Fig pattern in Microservices
- A strategy for incrementally migrating a monolithic application to a microservices architecture.
- You build a new microservice around the edges of the monolith, and intercept calls at the API Gateway. Over time, the microservices "strangle" the monolith by replacing its functionality piece by piece until the monolith is retired.

### 129. `@RestControllerAdvice` vs `@ControllerAdvice`
- `@ControllerAdvice` requires you to explicitly add `@ResponseBody` to every `@ExceptionHandler` method if you want to return a JSON payload rather than rendering a view template.
- `@RestControllerAdvice` is a convenience annotation that combines `@ControllerAdvice` + `@ResponseBody`. It automatically serializes all exception handler returns directly to JSON.

### 130. SQL: What is a Stored Procedure?
- A compiled, pre-written SQL code block that you save in the database. 
- **Benefits:** Reduced network traffic (only the call command is sent, not the whole query), faster execution (it is pre-compiled), and enhanced security (users can execute the procedure without needing direct read/write permissions on the underlying tables).

### 131. `throw` vs `throws` keyword
- **`throw`:** Used inside a method block to actually throw an exception explicitly (e.g., `throw new Exception("Error");`).
- **`throws`:** Used in the method signature to declare that the method *might* throw a checked exception, forcing the caller to handle it (e.g., `public void run() throws IOException`).

### 132. React: `useMemo` vs `useCallback`
- Both are performance optimization hooks in React used to prevent unnecessary re-renders.
- **`useMemo`:** Returns and caches the *result* of a heavy calculation.
- **`useCallback`:** Returns and caches the *actual function instance* itself (useful when passing callbacks to child components).

### 133. Can you force Garbage Collection in Java?
- **Answer:** No, you cannot strictly force it.
- You can request it by calling `System.gc()` or `Runtime.getRuntime().gc()`, but the JVM ultimately decides when (or if) to actually run the Garbage Collector. It is merely a suggestion to the JVM.

### 134. Difference between `save()` and `saveAndFlush()` in Spring Data JPA?
- **`save()`:** Saves the entity in the Hibernate Persistence Context (L1 cache) and queues the SQL INSERT/UPDATE statement. The statement is not executed until the transaction is committed or flushed.
- **`saveAndFlush()`:** Saves the entity and *immediately* forces the SQL statement to execute and sync with the database during the current transaction.

### 135. How do you create a Custom Exception in Java?
- Create a class that extends `Exception` (if you want a Checked Exception) or `RuntimeException` (if you want an Unchecked Exception). 
- Provide a constructor that takes a `String message` and passes it to the `super(message)` constructor.

### 136. How to avoid Deadlocks in Java?
- **Deadlock:** Occurs when two or more threads are blocked forever, waiting for each other to release a lock.
- **Avoidance Strategies:**
  1. Always acquire locks in the exact same order across all threads.
  2. Avoid nested locks (locking an object while already holding another lock).
  3. Use `tryLock()` with a timeout using the `java.util.concurrent.locks.Lock` interface instead of traditional `synchronized` blocks.

### 137. What is the `ThreadLocal` class in Java?
- Provides thread-local variables. Each thread accessing a `ThreadLocal` variable has its own independently initialized copy of the variable.
- Widely used to store user context (like a user ID, transaction ID, or Security Context) across a single thread's execution without having to pass it explicitly through every method signature.

### 138. React: Difference between `Props` and `State`
- **Props (Properties):** Passed from a parent component to a child component. They are read-only (immutable) from the child's perspective.
- **State:** Managed entirely within the component itself. It is mutable (can be updated via `setState` or `useState`), and when state changes, the component automatically re-renders.

### 139. Describe the Event Sourcing Pattern in Microservices
- Instead of storing just the *current state* of data in a database (like `balance = 500`), Event Sourcing stores a sequence of *state-changing events* (like `Deposit 1000`, `Withdraw 500`). 
- The current state is derived by replaying all the events. It provides an impeccable audit trail.

### 140. Method Overloading vs Overriding rules for Exceptions
- **Overloading:** Methods can throw entirely different exceptions. There are no rules restricting exceptions in overloaded methods.
- **Overriding:** If the parent method throws a Checked Exception, the child method can only throw the same exception or a subclass of it (it cannot throw a broader exception). However, there are no restrictions on Unchecked Exceptions.

### 141. What are WebSockets? How do they differ from REST?
- **REST:** Built on HTTP, which is strictly request-response. The client must ask for data every time.
- **WebSockets:** A protocol providing full-duplex, bi-directional, continuous communication over a single TCP connection. Once the connection is established, the server can push data to the client instantly without the client needing to ask (essential for chat apps and live trading dashboards).

### 142. Explain `CountDownLatch` vs `CyclicBarrier` in Java Concurrency
- **`CountDownLatch`:** Used when one or more threads need to wait for a fixed number of operations performed by other threads to complete. It cannot be reused once the count reaches zero.
- **`CyclicBarrier`:** Used when multiple threads need to wait for each other to reach a common execution barrier before proceeding. Unlike CountDownLatch, it *can* be reused (cyclic).

### 143. Difference between `Serialization` and `Externalization` in Java?
- **Serialization (`Serializable`):** A marker interface. The JVM handles the entire serialization process automatically. It's slower and provides less control over what is serialized.
- **Externalization (`Externalizable`):** You must implement `writeExternal()` and `readExternal()` methods to manually define exactly how the object's state is serialized and deserialized. Much faster and provides complete control.

### 144. What is the method to find the second highest salary in a dataset?
**Using SQL:**
```sql
SELECT MAX(salary) FROM Employee WHERE salary < (SELECT MAX(salary) FROM Employee);
```

### 145. Differences between `DELETE` and `TRUNCATE` in SQL?
- **DELETE:** DML command. Deletes row by row, logs deletions, can use `WHERE` clause, and can be rolled back.
- **TRUNCATE:** DDL command. Wipes entire table by deallocating data pages, extremely fast, usually cannot be rolled back, resets Identity counters.

### 146. In serialization, if a variable is marked as `transient`, how will it be transmitted?
- It will **NOT** be transmitted. The `transient` keyword forces the JVM to ignore the variable during serialization. Upon deserialization, it gets default values (e.g., `null`).

### 147. Given an array of numbers, find the frequency of each number.
```java
public static void findFrequency(int[] arr) {
    Map<Integer, Integer> frequencyMap = new HashMap<>();
    for (int num : arr) {
        frequencyMap.put(num, frequencyMap.getOrDefault(num, 0) + 1);
    }
}
```

### 148. Write a program to return the length of the longest word from a string whose length is even.
*(Iterate through words split by space, check `word.length() % 2 == 0` and keep track of max length).*

### 149. Sort a list of students based on their first name using Java 8.
```java
List<Student> sorted = students.stream()
    .sorted(Comparator.comparing(Student::getFirstName))
    .collect(Collectors.toList());
```

### 150. How do you find a memory leak in a production Java application?
- Capture a **Heap Dump** when the application crashes (using `-XX:+HeapDumpOnOutOfMemoryError`) or manually trigger it using tools like `jmap`.
- Load the heap dump into a memory profiler like **Eclipse MAT** (Memory Analyzer Tool) or **VisualVM**.
- Look for the "Dominator Tree" or "Leak Suspects" report to identify which objects are retaining the massive amounts of memory and trace their GC roots back to the culprit class.
