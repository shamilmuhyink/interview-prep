# IBS Software Services: Java Full Stack Developer Interview Questions (Ranked by Frequency)

Based on interview experiences from AmbitionBox, Glassdoor, and other platforms, here is an extensive list of the most frequently asked questions during IBS Software Java Full Stack Developer interviews. The questions are categorized to cover Core Java, Spring Boot, Angular, Microservices, and Databases.

## 1. Core Java & Advanced Java

### Q1: Explain the differences between `String`, `StringBuilder`, and `StringBuffer`. Why is `String` immutable?
**Answer:**
- **`String`**: Immutable. Any operation creates a new `String` object.
- **`StringBuilder`**: Mutable and non-synchronized. Faster than `StringBuffer`, used in single-threaded environments.
- **`StringBuffer`**: Mutable and synchronized (thread-safe). Slower due to overhead, used in multi-threaded environments.
- **Why `String` is immutable**: Security (used in class loading, network connections), Synchronization (inherently thread-safe), String Pool (allows caching).

### Q2: What are the Core OOPs concepts? Give real-world or Java examples.
**Answer:**
- **Encapsulation**: Wrapping data and code together. E.g., a `class` with `private` fields. Real world: A medical capsule.
- **Inheritance**: One class acquiring properties of another class. E.g., `Dog extends Animal`.
- **Polymorphism**: The ability to take many forms (method overloading/overriding).
- **Abstraction**: Hiding internal details and showing only functionality (abstract classes/interfaces).

### Q3: Explain Java 8 Streams API and Lambdas. How do you use them to process collections?
**Answer:**
- **Lambdas**: Anonymous functions that implement functional interfaces easily. Ex: `(a, b) -> a + b;`
- **Streams API**: Used to process collections declaratively. Pipelines operations (e.g., `filter`, `map`, `collect`).
- **Example Usage**: `names.stream().filter(n -> n.startsWith("A")).collect(Collectors.toList());`

### Q4: How does Exception Handling work in Java? Checked vs Unchecked exceptions?
**Answer:**
- **Checked Exceptions**: Checked at compile-time (e.g., `IOException`). Must be handled or declared.
- **Unchecked Exceptions**: Runtime exceptions (e.g., `NullPointerException`). Not checked at compile-time.
- **final vs finally vs finalize**: `final` restricts modification, `finally` always executes after try-catch, and `finalize()` is called by Garbage Collector.

### Q5: What is the difference between `HashMap` and `ConcurrentHashMap`?
**Answer:**
- **`HashMap`**: Non-synchronized, not thread-safe. Fast but can result in `ConcurrentModificationException` in multi-threaded environments.
- **`ConcurrentHashMap`**: Thread-safe without locking the whole map (uses bucket-level locking or CAS operations). Safe for concurrent reads and writes.

### Q6: How does `HashMap` work internally in Java?
**Answer:**
It uses an array of Nodes (buckets). When `put(K, V)` is called, it calculates the hash of the key to find the bucket index. If the bucket is empty, the node is placed there. If there's a collision, it stores the node in a Linked List (or a Balanced Tree if the list size exceeds a threshold, introduced in Java 8).

### Q7: Explain the difference between `ArrayList` and `LinkedList`.
**Answer:**
- **`ArrayList`**: Backed by a dynamic array. Fast random access O(1), but insertion/deletion in the middle is slow O(N) due to shifting.
- **`LinkedList`**: Backed by a doubly-linked list. Random access is slow O(N), but insertion/deletion is fast O(1) if the node is known.

### Q8: What is the `volatile` keyword in Java?
**Answer:**
`volatile` is an indicator to the Java compiler and Thread that the value of the variable must always be read from the main memory, not from the thread's local cache. It guarantees visibility of changes to variables across threads.

### Q9: Explain the concept of thread-safety in Java. How can we achieve it?
**Answer:**
Thread safety means a piece of code functions correctly during simultaneous execution by multiple threads. It can be achieved by:
- Using `synchronized` keyword (methods or blocks).
- Using `volatile` keyword.
- Using Atomic classes (`AtomicInteger`, etc.).
- Using thread-safe collections (`ConcurrentHashMap`, `CopyOnWriteArrayList`).
- Using immutable objects.

### Q10: What is a Thread Pool? How do you create one using `ExecutorService`?
**Answer:**
A thread pool is a group of pre-instantiated, reusable threads. It avoids the overhead of creating and destroying threads repeatedly.
Created via: `ExecutorService executor = Executors.newFixedThreadPool(10);`

### Q11: What is the difference between `Callable` and `Runnable`?
**Answer:**
- **`Runnable`**: Cannot return a result and cannot throw a checked exception. Method: `public void run()`
- **`Callable`**: Can return a result (via Future) and can throw a checked exception. Method: `public Object call()`

### Q12: Explain the Java Memory Model (Heap vs Stack).
**Answer:**
- **Heap**: Used for dynamic memory allocation for Java objects and JRE classes at runtime. Shared among all threads.
- **Stack**: Used for execution of threads. Contains primitive values and references to objects in the heap. Each thread has its own stack.

### Q13: How does Garbage Collection work in Java?
**Answer:**
GC is an automatic process that reclaims memory allocated to objects that are no longer reachable. Java uses a Generational Garbage Collection strategy (Young Generation, Old/Tenured Generation, Metaspace). Minor GC happens in Young Gen, and Major GC happens in Old Gen.

### Q14: What are default and static methods in Interfaces (Java 8)?
**Answer:**
Java 8 introduced them to add new methods to interfaces without breaking existing implementations.
- **`default`**: Can be overridden by implementing classes.
- **`static`**: Belong to the interface itself, cannot be overridden.

### Q15: What is `Optional` in Java 8 and why is it used?
**Answer:**
`Optional` is a container object used to represent a value that may or may not be present (null). It is used to avoid `NullPointerException` and write cleaner, null-safe code.

### Q16: Explain the difference between `Comparable` and `Comparator`.
**Answer:**
- **`Comparable`**: Used to define the natural/default sorting logic of an object. Modifies the original class (implements `Comparable<T>` and `compareTo(T o)`).
- **`Comparator`**: Used to define multiple external sorting logics. Doesn't modify the original class (implements `Comparator<T>` and `compare(T o1, T o2)`).

### Q17: What is the difference between `wait()` and `sleep()`?
**Answer:**
- **`wait()`**: Called on an Object. Releases the lock. Must be called from a synchronized context. Can be woken up by `notify()`.
- **`sleep()`**: Called on a Thread. Does NOT release the lock. Pauses execution for a specific time.

### Q18: Explain Serialization and Deserialization in Java. What is `transient`?
**Answer:**
- **Serialization**: Converting an object's state to a byte stream to be saved to a file or sent over a network. (Implements `Serializable`).
- **Deserialization**: Recreating the object from the byte stream.
- **`transient`**: A keyword used to prevent a variable from being serialized.

### Q19: What are the differences between `throw` and `throws`?
**Answer:**
- **`throw`**: Used to explicitly throw an exception from within a method block.
- **`throws`**: Used in a method signature to declare that the method might throw exceptions, pushing the responsibility to the caller.

## 2. Coding & Problem Solving

### Q20: Write a program to swap two numbers without using a third/temporary variable.
**Answer:**
```java
public void swap(int a, int b) {
    a = a + b;
    b = a - b;
    a = a - b;
}
```

### Q21: How do you remove duplicate elements from an Array?
**Answer:**
Using a HashSet (O(N) time complexity):
```java
public Integer[] removeDuplicates(int[] arr) {
    Set<Integer> set = new LinkedHashSet<>();
    for(int num : arr) set.add(num);
    return set.toArray(new Integer[0]);
}
```

### Q22: Write a logic to check if a String is a Palindrome.
**Answer:**
```java
public boolean isPalindrome(String str) {
    int left = 0, right = str.length() - 1;
    while(left < right) {
        if(str.charAt(left++) != str.charAt(right--)) return false;
    }
    return true;
}
```

### Q23: How do you find the second highest number in an array?
**Answer:**
```java
public int secondHighest(int[] arr) {
    int highest = Integer.MIN_VALUE, secondHighest = Integer.MIN_VALUE;
    for(int num : arr) {
        if(num > highest) {
            secondHighest = highest;
            highest = num;
        } else if (num > secondHighest && num != highest) {
            secondHighest = num;
        }
    }
    return secondHighest;
}
```

### Q24: Check if two strings are Anagrams.
**Answer:**
Convert both strings to char arrays, sort them, and check if they are equal using `Arrays.equals(charArray1, charArray2)`.

### Q25: Write a program to reverse a String without using built-in methods.
**Answer:**
```java
public String reverse(String str) {
    char[] chars = str.toCharArray();
    String reversed = "";
    for(int i = chars.length - 1; i >= 0; i--) reversed += chars[i];
    return reversed;
}
```

### Q26: Find the first non-repeating character in a String.
**Answer:**
Use a `LinkedHashMap<Character, Integer>` to store the character counts while maintaining insertion order, then iterate and return the first char with count 1.

### Q27: How to find if a Linked List has a cycle?
**Answer:**
Use Floyd’s Cycle-Finding Algorithm (Tortoise and Hare). Move a `slow` pointer by 1 step and a `fast` pointer by 2 steps. If they meet, there is a cycle.

## 3. Spring Boot & Backend Frameworks

### Q28: What is the difference between `@RequestParam` and `@PathVariable` in Spring?
**Answer:**
- `@RequestParam`: Extracts values from query params (e.g., `/users?id=10`). 
- `@PathVariable`: Extracts values from the URI path (e.g., `/users/10`).

### Q29: How do you implement Global Exception Handling in a Spring Boot application?
**Answer:**
Use `@ControllerAdvice` at the class level and `@ExceptionHandler(ExceptionClass.class)` at the method level to intercept and handle exceptions across all controllers.

### Q30: Differentiate between Hibernate `Session` and `SessionFactory`.
**Answer:**
- **SessionFactory**: Thread-safe, heavyweight, created once per app.
- **Session**: Non-thread-safe, lightweight, represents a single database transaction/unit of work.

### Q31: What is Dependency Injection (DI) and Inversion of Control (IoC)?
**Answer:**
- **IoC**: A design principle where the control of object creation is transferred to a container/framework.
- **DI**: A pattern used to implement IoC, where the container injects dependencies into an object instead of the object creating them itself.

### Q32: Explain the different Bean Scopes in Spring.
**Answer:**
Singleton (default, one per container), Prototype (new instance every time), Request (one per HTTP request), Session (one per HTTP session), GlobalSession.

### Q33: What is Spring AOP?
**Answer:**
Aspect-Oriented Programming separates cross-cutting concerns (like logging, security, transactions) from business logic using Aspects, Advices (what and when to run), and Pointcuts (where to run).

### Q34: How does Spring Boot Auto-configuration work?
**Answer:**
Spring Boot uses `@EnableAutoConfiguration` (part of `@SpringBootApplication`) to automatically configure the Spring application based on the dependencies present in the classpath (e.g., if H2 is on the classpath, it configures an in-memory DB).

### Q35: What is the purpose of Spring Boot Actuator?
**Answer:**
It provides production-ready features to help monitor and manage the application (e.g., health checks, metrics, thread dumps, environment details) via HTTP endpoints.

### Q36: Difference between `@Component`, `@Service`, `@Repository`, and `@Controller`.
**Answer:**
All are specialized `@Component` annotations.
- `@Service`: Denotes business logic.
- `@Repository`: Denotes Data Access Object (DAO), translates DB exceptions to Spring's DataAccessException.
- `@Controller`: Denotes a presentation layer component handling web requests.

### Q37: How do you secure a REST API using Spring Security and JWT?
**Answer:**
Implement a filter (`OncePerRequestFilter`) to intercept requests, extract the JWT from the `Authorization` header, validate its signature and expiration, extract user roles, and set the authentication context in `SecurityContextHolder`.

### Q38: What is Spring Data JPA?
**Answer:**
A higher-level abstraction over JPA that reduces boilerplate code. You only need to declare repository interfaces extending `JpaRepository` and Spring automatically provides the CRUD implementations.

### Q39: What is `@Transactional` propagation?
**Answer:**
Defines how transactions relate to each other. E.g., `REQUIRED` (use existing or create new), `REQUIRES_NEW` (always create new, suspend existing).

## 4. Frontend (Angular)

### Q40: Explain Component Lifecycle Hooks in Angular.
**Answer:**
- `ngOnChanges()`: Called when an input binding value changes.
- `ngOnInit()`: Component initialization.
- `ngDoCheck()`: Custom change detection.
- `ngAfterViewInit()`: After Angular initializes the views.
- `ngOnDestroy()`: Cleanup (unsubscribing observables).

### Q41: How do you handle form validation in Angular and Spring Boot?
**Answer:**
- **Angular**: Use Reactive Forms, define logic in the component class, and apply `Validators` (e.g., `Validators.required`).
- **Spring Boot**: Use Validation API (`@Valid`, `@NotNull`) on DTOs, catch `MethodArgumentNotValidException` via `@ControllerAdvice`.

### Q42: What is the difference between Directives and Components in Angular?
**Answer:**
A Component is a directive with a template (HTML). Directives (like `*ngIf`, `*ngFor` or custom attributes) add behavior to an existing DOM element without their own template.

### Q43: Explain the difference between Observables and Promises.
**Answer:**
- **Promises**: Handle a single asynchronous event. Eager execution, cannot be cancelled.
- **Observables**: Handle a stream of events over time. Lazy execution, can be cancelled (unsubscribed), and support operators (map, filter).

### Q44: What is RxJS? Name a few commonly used RxJS operators.
**Answer:**
RxJS is a library for reactive programming using Observables. Commonly used operators include `map` (transform data), `filter` (filter data), `mergeMap` / `switchMap` (flattening inner observables).

### Q45: How does Routing work in Angular? Explain Route Guards.
**Answer:**
Routing enables navigation between views. Route Guards (like `CanActivate`, `CanDeactivate`) act as interceptors to allow or deny navigation based on conditions (e.g., checking if a user is logged in).

### Q46: What are Angular Interceptors?
**Answer:**
Classes that implement `HttpInterceptor`. They intercept outgoing HTTP requests (to add tokens/headers) and incoming HTTP responses (to handle global errors).

## 5. Microservices, System Design, & DevOps

### Q47: What is the API Gateway pattern?
**Answer:**
A single entry point for all clients. It routes requests to appropriate microservices, aggregates responses, and handles cross-cutting concerns like authentication, rate limiting, and SSL termination.

### Q48: Explain the Circuit Breaker pattern.
**Answer:**
Prevents a microservice from repeatedly calling a failing service, preventing cascading failures. If a service fails consecutively, the circuit "opens," failing fast. After a timeout, it "half-opens" to test if the service is back. (Implemented using Resilience4j).

### Q49: What is Service Discovery? How does Eureka work?
**Answer:**
In a dynamic environment, IP addresses of services change. Services register themselves with a Service Registry (like Netflix Eureka). Clients query Eureka to find the exact IP/port of a required service.

### Q50: How do you handle distributed transactions in Microservices?
**Answer:**
Standard ACID transactions don't work across multiple databases. 
Use the **Saga Pattern**: A sequence of local transactions where each updates data within a single service and publishes an event to trigger the next transaction. If one fails, compensating transactions are triggered to undo changes.

### Q51: What is the difference between Kafka and RabbitMQ?
**Answer:**
- **RabbitMQ**: Traditional message broker (Smart Broker, Dumb Consumer). Excellent for complex routing and task queues. Messages are deleted after consumption.
- **Kafka**: Distributed streaming platform (Dumb Broker, Smart Consumer). Excellent for high-throughput, event-driven architectures. Messages are persisted and can be replayed.

### Q52: What is Docker? How do you containerize a Spring Boot app?
**Answer:**
Docker packages an application and its dependencies into a standardized unit (container). 
Create a `Dockerfile`: Define a base image (e.g., `openjdk:17`), copy the `.jar` file into the image, and define the `ENTRYPOINT` (`java -jar app.jar`).

### Q53: What is the purpose of Database Indexing?
**Answer:**
Indexes are data structures (usually B-Trees) that improve the speed of data retrieval operations on a database table at the cost of additional storage space and slower writes (INSERT/UPDATE/DELETE).

### Q54: Explain CI/CD pipelines.
**Answer:**
- **Continuous Integration (CI)**: Automatically building and testing code every time a developer commits changes (e.g., via Jenkins, GitHub Actions).
- **Continuous Deployment (CD)**: Automatically deploying the validated code to staging or production environments.
