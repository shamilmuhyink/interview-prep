# IBS Software Services: Round 2 (Technical & Managerial) Interview Questions

This document contains conceptual technical questions covering Core Java, Spring Boot, Databases, Frontend (Angular), Software Architecture, Cloud (AWS), DevOps, Agile Methodologies, and Leadership.

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

### Q3: Explain Compile-time vs Run-time Polymorphism with examples.
**Answer:**
- **Compile-time (Static) Polymorphism**: Achieved via **Method Overloading**. The compiler determines which method to call based on the method signature (number or type of parameters) within the same class.
- **Run-time (Dynamic) Polymorphism**: Achieved via **Method Overriding**. The JVM determines which method to call at runtime based on the actual object type, not the reference type. It occurs when a subclass provides a specific implementation for a method declared in its superclass.

### Q4: Explain Java 8 Streams API and Lambdas. How do you use them to process collections?
**Answer:**
- **Lambdas**: Anonymous functions that implement functional interfaces easily. Ex: `(a, b) -> a + b;`
- **Streams API**: Used to process collections declaratively. Pipelines operations (e.g., `filter`, `map`, `collect`).
- **Example Usage**: `names.stream().filter(n -> n.startsWith("A")).collect(Collectors.toList());`

### Q5: What is Functional Programming in Java? What features support it?
**Answer:**
Functional programming is a declarative paradigm emphasizing pure functions, immutability, and higher-order functions (functions that take or return other functions). 
Java 8 introduced it via:
- **Lambdas**: Treating functionality as a method argument.
- **Functional Interfaces**: Interfaces with a single abstract method (e.g., `Predicate`, `Function`, `Consumer`).
- **Streams API**: Enables declarative operations on collections without modifying the original data.

### Q6: How does Exception Handling work in Java? Checked vs Unchecked exceptions?
**Answer:**
- **Checked Exceptions**: Checked at compile-time (e.g., `IOException`). Must be handled or declared.
- **Unchecked Exceptions**: Runtime exceptions (e.g., `NullPointerException`). Not checked at compile-time.
- **final vs finally vs finalize**: `final` restricts modification, `finally` always executes after try-catch, and `finalize()` is called by Garbage Collector.

### Q7: What is the difference between `HashMap` and `ConcurrentHashMap`?
**Answer:**
- **`HashMap`**: Non-synchronized, not thread-safe. Fast but can result in `ConcurrentModificationException` in multi-threaded environments.
- **`ConcurrentHashMap`**: Thread-safe without locking the whole map (uses bucket-level locking or CAS operations). Safe for concurrent reads and writes.

### Q8: How does `HashMap` work internally in Java?
**Answer:**
It uses an array of Nodes (buckets). When `put(K, V)` is called, it calculates the hash of the key to find the bucket index. If the bucket is empty, the node is placed there. If there's a collision, it stores the node in a Linked List (or a Balanced Tree if the list size exceeds a threshold, introduced in Java 8).

### Q9: Explain the difference between `ArrayList` and `LinkedList`.
**Answer:**
- **`ArrayList`**: Backed by a dynamic array. Fast random access O(1), but insertion/deletion in the middle is slow O(N) due to shifting.
- **`LinkedList`**: Backed by a doubly-linked list. Random access is slow O(N), but insertion/deletion is fast O(1) if the node is known.

### Q10: What is the `volatile` keyword in Java?
**Answer:**
`volatile` is an indicator to the Java compiler and Thread that the value of the variable must always be read from the main memory, not from the thread's local cache. It guarantees visibility of changes to variables across threads.

### Q11: Explain the concept of thread-safety in Java. How can we achieve it?
**Answer:**
Thread safety means a piece of code functions correctly during simultaneous execution by multiple threads. It can be achieved by:
- Using `synchronized` keyword (methods or blocks).
- Using `volatile` keyword.
- Using Atomic classes (`AtomicInteger`, etc.).
- Using thread-safe collections (`ConcurrentHashMap`, `CopyOnWriteArrayList`).
- Using immutable objects.

### Q12: What is a Thread Pool? How do you create one using `ExecutorService`?
**Answer:**
A thread pool is a group of pre-instantiated, reusable threads. It avoids the overhead of creating and destroying threads repeatedly.
Created via: `ExecutorService executor = Executors.newFixedThreadPool(10);`

### Q13: What is the difference between `Callable` and `Runnable`?
**Answer:**
- **`Runnable`**: Cannot return a result and cannot throw a checked exception. Method: `public void run()`
- **`Callable`**: Can return a result (via Future) and can throw a checked exception. Method: `public Object call()`

### Q14: Explain the Java Memory Model (Heap vs Stack).
**Answer:**
- **Heap**: Used for dynamic memory allocation for Java objects and JRE classes at runtime. Shared among all threads.
- **Stack**: Used for execution of threads. Contains primitive values and references to objects in the heap. Each thread has its own stack.

### Q15: How does Garbage Collection work in Java?
**Answer:**
GC is an automatic process that reclaims memory allocated to objects that are no longer reachable. Java uses a Generational Garbage Collection strategy (Young Generation, Old/Tenured Generation, Metaspace). Minor GC happens in Young Gen, and Major GC happens in Old Gen.

### Q16: What are default and static methods in Interfaces (Java 8)?
**Answer:**
Java 8 introduced them to add new methods to interfaces without breaking existing implementations.
- **`default`**: Can be overridden by implementing classes.
- **`static`**: Belong to the interface itself, cannot be overridden.

### Q17: What is `Optional` in Java 8 and why is it used?
**Answer:**
`Optional` is a container object used to represent a value that may or may not be present (null). It is used to avoid `NullPointerException` and write cleaner, null-safe code.

### Q18: Explain the difference between `Comparable` and `Comparator`.
**Answer:**
- **`Comparable`**: Used to define the natural/default sorting logic of an object. Modifies the original class (implements `Comparable<T>` and `compareTo(T o)`).
- **`Comparator`**: Used to define multiple external sorting logics. Doesn't modify the original class (implements `Comparator<T>` and `compare(T o1, T o2)`).

### Q19: What is the difference between `wait()` and `sleep()`?
**Answer:**
- **`wait()`**: Called on an Object. Releases the lock. Must be called from a synchronized context. Can be woken up by `notify()`.
- **`sleep()`**: Called on a Thread. Does NOT release the lock. Pauses execution for a specific time.

### Q20: Explain Serialization and Deserialization in Java. What is `transient`?
**Answer:**
- **Serialization**: Converting an object's state to a byte stream to be saved to a file or sent over a network. (Implements `Serializable`).
- **Deserialization**: Recreating the object from the byte stream.
- **`transient`**: A keyword used to prevent a variable from being serialized.

### Q21: What are the differences between `throw` and `throws`?
**Answer:**
- **`throw`**: Used to explicitly throw an exception from within a method block.
- **`throws`**: Used in a method signature to declare that the method might throw exceptions, pushing the responsibility to the caller.

### Q22: How do you change the default port of a Spring Boot application?
**Answer:**
You can change the default port (8080) by adding `server.port=8081` in `application.properties` or `server: port: 8081` in `application.yml`. You can also pass it as a command-line argument: `java -jar app.jar --server.port=8081`.

### Q23: How do you read custom properties from application.properties or YAML in Spring Boot?
**Answer:**
- **`@Value("${property.name}")`**: Injects a specific property value directly into a field.
- **`@ConfigurationProperties(prefix="my.config")`**: Binds a set of related properties to a POJO class, providing type safety and structured configuration.
- **`Environment` object**: You can `@Autowired Environment env` and call `env.getProperty("property.name")`.

### Q24: What are the mechanisms to call a 3rd party REST API in Spring Boot?
**Answer:**
- **`RestTemplate`**: A synchronous, blocking client (currently in maintenance mode).
- **`WebClient`**: A modern, non-blocking, reactive client introduced in Spring WebFlux.
- **`OpenFeign` / `FeignClient`**: A declarative REST client where you write an interface with Spring MVC annotations, and Spring provides the implementation at runtime.

### Q25: What is the difference between `@RequestParam` and `@PathVariable` in Spring?
**Answer:**
- `@RequestParam`: Extracts values from query params (e.g., `/users?id=10`). 
- `@PathVariable`: Extracts values from the URI path (e.g., `/users/10`).

### Q26: How do you implement Global Exception Handling in a Spring Boot application?
**Answer:**
Use `@ControllerAdvice` at the class level and `@ExceptionHandler(ExceptionClass.class)` at the method level to intercept and handle exceptions across all controllers.

### Q27: What is Dependency Injection (DI) and Inversion of Control (IoC)?
**Answer:**
- **IoC**: A design principle where the control of object creation is transferred to a container/framework.
- **DI**: A pattern used to implement IoC, where the container injects dependencies into an object instead of the object creating them itself.

### Q28: Explain the different Bean Scopes in Spring.
**Answer:**
Singleton (default, one per container), Prototype (new instance every time), Request (one per HTTP request), Session (one per HTTP session), GlobalSession.

### Q29: What is Spring AOP?
**Answer:**
Aspect-Oriented Programming separates cross-cutting concerns (like logging, security, transactions) from business logic using Aspects, Advices (what and when to run), and Pointcuts (where to run).

### Q30: How does Spring Boot Auto-configuration work?
**Answer:**
Spring Boot uses `@EnableAutoConfiguration` (part of `@SpringBootApplication`) to automatically configure the Spring application based on the dependencies present in the classpath (e.g., if H2 is on the classpath, it configures an in-memory DB).

### Q31: What is the purpose of Spring Boot Actuator?
**Answer:**
It provides production-ready features to help monitor and manage the application (e.g., health checks, metrics, thread dumps, environment details) via HTTP endpoints.

### Q32: Difference between `@Component`, `@Service`, `@Repository`, and `@Controller`.
**Answer:**
All are specialized `@Component` annotations.
- `@Service`: Denotes business logic.
- `@Repository`: Denotes Data Access Object (DAO), translates DB exceptions to Spring's DataAccessException.
- `@Controller`: Denotes a presentation layer component handling web requests.

### Q33: What is Spring Data JPA?
**Answer:**
A higher-level abstraction over JPA that reduces boilerplate code. You only need to declare repository interfaces extending `JpaRepository` and Spring automatically provides the CRUD implementations.

### Q34: Differentiate between Hibernate `Session` and `SessionFactory`.
**Answer:**
- **SessionFactory**: Thread-safe, heavyweight, created once per app.
- **Session**: Non-thread-safe, lightweight, represents a single database transaction/unit of work.

### Q35: What is `@Transactional` propagation?
**Answer:**
Defines how transactions relate to each other. E.g., `REQUIRED` (use existing or create new), `REQUIRES_NEW` (always create new, suspend existing).

### Q36: How do you secure a REST API using Spring Security and JWT?
**Answer:**
Implement a filter (`OncePerRequestFilter`) to intercept requests, extract the JWT from the `Authorization` header, validate its signature and expiration, extract user roles, and set the authentication context in `SecurityContextHolder`.

### Q37: Explain the exact flow of JWT Authentication in Spring Security.
**Answer:**
1. **Authentication**: The client sends credentials (username/password) to a login endpoint.
2. **Validation**: The `AuthenticationManager` verifies credentials against the DB (via `UserDetailsService`).
3. **Token Generation**: If valid, the server generates a JWT signed with a secret key and returns it.
4. **Authorization**: For subsequent requests, the client includes the JWT in the `Authorization: Bearer <token>` header.
5. **Token Verification**: A custom filter intercepts the request, verifies the token's signature/expiration, and if valid, sets the `SecurityContext` to allow access.

### Q38: What is the difference between SQL and NoSQL databases, and when would you choose NoSQL?
**Answer:**
- **SQL**: Relational, uses predefined schemas, and is vertically scalable. Best for complex, ACID-compliant transactions (e.g., financial systems).
- **NoSQL**: Non-relational, schema-less (Document, Key-Value, Graph), and horizontally scalable. 
- **When to choose NoSQL**: I choose NoSQL (like MongoDB or DynamoDB) when handling unstructured data, requiring high-velocity data ingestion, or needing a highly scalable, distributed storage system without complex multi-table JOINs.

### Q39: What is the purpose of Database Indexing?
**Answer:**
Indexes are data structures (usually B-Trees) that improve the speed of data retrieval operations on a database table at the cost of additional storage space and slower writes (INSERT/UPDATE/DELETE).

### Q40: Write a SQL query using JOIN, GROUP BY, and HAVING to find departments with more than 5 employees.
**Answer:**
```sql
SELECT d.department_name, COUNT(e.emp_id) as total_employees
FROM Department d
JOIN Employee e ON d.dept_id = e.dept_id
GROUP BY d.department_name
HAVING COUNT(e.emp_id) > 5;
```

### Q41: Explain Component Lifecycle Hooks in Angular.
**Answer:**
- `ngOnChanges()`: Called when an input binding value changes.
- `ngOnInit()`: Component initialization.
- `ngDoCheck()`: Custom change detection.
- `ngAfterViewInit()`: After Angular initializes the views.
- `ngOnDestroy()`: Cleanup (unsubscribing observables).

### Q42: How do you handle form validation in Angular and Spring Boot?
**Answer:**
- **Angular**: Use Reactive Forms, define logic in the component class, and apply `Validators` (e.g., `Validators.required`).
- **Spring Boot**: Use Validation API (`@Valid`, `@NotNull`) on DTOs, catch `MethodArgumentNotValidException` via `@ControllerAdvice`.

### Q43: What is the difference between Directives and Components in Angular?
**Answer:**
A Component is a directive with a template (HTML). Directives (like `*ngIf`, `*ngFor` or custom attributes) add behavior to an existing DOM element without their own template.

### Q44: Explain the difference between Observables and Promises.
**Answer:**
- **Promises**: Handle a single asynchronous event. Eager execution, cannot be cancelled.
- **Observables**: Handle a stream of events over time. Lazy execution, can be cancelled (unsubscribed), and support operators (map, filter).

### Q45: What is RxJS? Name a few commonly used RxJS operators.
**Answer:**
RxJS is a library for reactive programming using Observables. Commonly used operators include `map` (transform data), `filter` (filter data), `mergeMap` / `switchMap` (flattening inner observables).

### Q46: How does Routing work in Angular? Explain Route Guards.
**Answer:**
Routing enables navigation between views. Route Guards (like `CanActivate`, `CanDeactivate`) act as interceptors to allow or deny navigation based on conditions (e.g., checking if a user is logged in).

### Q47: What are Angular Interceptors?
**Answer:**
Classes that implement `HttpInterceptor`. They intercept outgoing HTTP requests (to add tokens/headers) and incoming HTTP responses (to handle global errors).

### Q48: What are some common software architectural patterns you've implemented?
**Answer:**
- **Microservices Architecture**: Breaking down a monolith into small, independently deployable services.
- **Event-Driven Architecture**: Decoupling services using message queues/topics (like Kafka or RabbitMQ) to react to events asynchronously.
- **MVC (Model-View-Controller)**: Separating the application's data, user interface, and control logic.
- **CQRS (Command Query Responsibility Segregation)**: Separating read operations (Queries) and write operations (Commands) into different models for optimization.

### Q49: What are Enterprise Integration Patterns (EIP) and how have you used them?
**Answer:**
EIPs are standard solutions for designing and implementing integrations between different enterprise systems. Common patterns include Message Router, Publish-Subscribe Channel, Message Translator, and Scatter-Gather. I have utilized frameworks like Apache Camel or Spring Integration to implement these patterns, decoupling services and effectively routing messages asynchronously via message brokers.

### Q50: What is the API Gateway pattern?
**Answer:**
A single entry point for all clients. It routes requests to appropriate microservices, aggregates responses, and handles cross-cutting concerns like authentication, rate limiting, and SSL termination.

### Q51: How do you implement an API Gateway in Spring Cloud and handle routing?
**Answer:**
In **Spring Cloud Gateway**, you define routes either in `application.yml` or via Java Config. A route consists of:
- **ID**: Unique identifier.
- **URI**: The destination microservice.
- **Predicates**: Conditions to match the request (e.g., `Path=/api/users/**`).
- **Filters**: Modify the request/response (e.g., `AddRequestHeader`, `RewritePath`, or custom filters).

### Q52: What is Service Discovery? How does Eureka work?
**Answer:**
In a dynamic environment, IP addresses of services change. Services register themselves with a Service Registry (like Netflix Eureka). Clients query Eureka to find the exact IP/port of a required service.

### Q53: Explain the Circuit Breaker pattern.
**Answer:**
Prevents a microservice from repeatedly calling a failing service, preventing cascading failures. If a service fails consecutively, the circuit "opens," failing fast. After a timeout, it "half-opens" to test if the service is back. (Implemented using Resilience4j).

### Q54: How do you handle distributed transactions in Microservices?
**Answer:**
Standard ACID transactions don't work across multiple databases. 
Use the **Saga Pattern**: A sequence of local transactions where each updates data within a single service and publishes an event to trigger the next transaction. If one fails, compensating transactions are triggered to undo changes.

### Q55: What is the difference between Kafka and RabbitMQ?
**Answer:**
- **RabbitMQ**: Traditional message broker (Smart Broker, Dumb Consumer). Excellent for complex routing and task queues. Messages are deleted after consumption.
- **Kafka**: Distributed streaming platform (Dumb Broker, Smart Consumer). Excellent for high-throughput, event-driven architectures. Messages are persisted and can be replayed.

### Q56: Explain the use cases for AWS Lambda, S3, and managed Kubernetes (EKS) in a Microservices architecture.
**Answer:**
- **AWS Lambda**: Serverless compute used for event-driven, short-lived tasks (e.g., triggering a data processing script when a file is uploaded, or running a lightweight cron job).
- **Amazon S3**: Highly durable object storage used for storing static frontend assets, application backups, or acting as a data lake for raw analytics data.
- **Amazon EKS (Kubernetes)**: A managed container orchestration service used to deploy, scale, and automatically manage our dockerized Spring Boot microservices across clusters.

### Q57: What is Docker? How do you containerize a Spring Boot app?
**Answer:**
Docker packages an application and its dependencies into a standardized unit (container). 
Create a `Dockerfile`: Define a base image (e.g., `openjdk:17`), copy the `.jar` file into the image, and define the `ENTRYPOINT` (`java -jar app.jar`).

### Q58: How do you build a runnable JAR file using Maven?
**Answer:**
You can use `mvn clean install` or `mvn clean package`. In a Spring Boot application, the `spring-boot-maven-plugin` repacks the standard JAR into an executable, "fat" JAR containing all necessary dependencies and an embedded web server (like Tomcat). It can be run using `java -jar application.jar`.

### Q59: Explain CI/CD pipelines.
**Answer:**
- **Continuous Integration (CI)**: Automatically building and testing code every time a developer commits changes (e.g., via Jenkins, GitHub Actions).
- **Continuous Deployment (CD)**: Automatically deploying the validated code to staging or production environments.

### Q60: What is the difference between `git merge` and `git rebase`?
**Answer:**
- **`git merge`**: Combines two branches and creates a new "merge commit". It preserves the exact history but can result in a messy, non-linear commit graph.
- **`git rebase`**: Takes the commits from the current branch and reapplies them sequentially on top of another branch. It rewrites project history to create a clean, linear progression without extra merge commits.

### Q61: Explain Agile Methodology and its standard ceremonies.
**Answer:**
Agile is an iterative approach to software development focused on delivering value quickly in small increments. Scrum is a popular framework involving:
- **Sprint**: A fixed timebox (e.g., 2 weeks) to complete work.
- **Sprint Planning**: Defining the sprint goal and selecting tasks from the product backlog.
- **Daily Standup**: A 15-minute daily sync on what was done, what will be done, and blockers.
- **Sprint Review**: Demoing completed work to stakeholders.
- **Sprint Retrospective**: Reflecting on what went well and what can be improved.

### Q62: As a Lead Developer, how do you handle technical disagreements within a Scrum team and mentor junior developers?
**Answer:**
I handle disagreements by facilitating open discussions, relying on data and POCs (Proof of Concepts) rather than opinions, and ensuring architectural decisions align with broader business goals. For mentorship, I conduct regular pair programming sessions, use code reviews as a collaborative learning tool, and provide constructive, actionable feedback during sprint retrospectives.

### Q63: Explain the 5 SOLID Principles of Object-Oriented Design.
**Answer:**
- **S**ingle Responsibility Principle: A class should have one, and only one, reason to change.
- **O**pen/Closed Principle: Software entities should be open for extension but closed for modification.
- **L**iskov Substitution Principle: Objects of a superclass shall be replaceable with objects of its subclasses without breaking the application.
- **I**nterface Segregation Principle: No client should be forced to depend on methods it does not use (create small, specific interfaces).
- **D**ependency Inversion Principle: High-level modules should not depend on low-level modules. Both should depend on abstractions (e.g., using Interfaces).

### Q64: What is the exact difference between `ClassNotFoundException` and `NoClassDefFoundError`?
**Answer:**
- **`ClassNotFoundException`**: A checked exception that occurs when an application tries to load a class at runtime using `Class.forName()` or `loadClass()` but the `.class` file is not found in the classpath.
- **`NoClassDefFoundError`**: An error (unchecked) that occurs when a class was successfully compiled and present during compile-time, but the JVM cannot find it at runtime (e.g., the `.jar` was removed or classpath changed).

### Q65: Which Design Patterns have you commonly implemented in your projects?
**Answer:**
- **Singleton Pattern**: Used for creating a single instance of a configuration manager or database connection pool. In Spring, beans are Singleton by default.
- **Factory Pattern**: Used when we have a superclass with multiple subclasses and want to return one of the subclasses based on input.
- **Builder Pattern**: Used to construct a complex object step by step, avoiding multiple constructors with many parameters.

### Q66: Explain the difference between pure Hibernate and Spring Data JPA.
**Answer:**
- **Hibernate**: It is a JPA implementation (an ORM framework). It provides the actual logic to map Java objects to database tables and handles the underlying JDBC operations.
- **Spring Data JPA**: It is a higher-level abstraction on top of JPA providers like Hibernate. It eliminates boilerplate code by allowing developers to write repository interfaces, and Spring automatically provides the CRUD implementation at runtime.
