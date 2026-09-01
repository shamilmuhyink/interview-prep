# Deloitte Technical Round - Frequently Asked Questions (Java + Angular)

> **Sources:** AmbitionBox, Glassdoor, GeeksforGeeks, InterviewQuery — ranked by frequency of appearance across candidate reports.

---

### 1. Explain the four pillars of Object-Oriented Programming (OOP) with real-world examples.
**Answer:**
- **Encapsulation:** Wrapping data (variables) and code (methods) together into a single unit (class). Example: A bank account where balance is hidden and accessed only through specific methods (deposit/withdraw).
- **Abstraction:** Hiding internal implementation details and showing only functionality to the user. Example: Driving a car — you use the steering and pedals without knowing how the engine works internally.
- **Inheritance:** The mechanism by which one class acquires the properties and behaviors of another class. Example: A parent class `Vehicle` and child classes `Car` and `Bike` which inherit properties like `wheels` or `engine`.
- **Polymorphism:** The ability of a single function or object to take on multiple forms. In code, this is achieved via method overloading (compile-time) and overriding (run-time).

---

### 2. What are Java 8 Streams? Write code using filter, map, and collect.
**Answer:**
Java Streams provide a functional, declarative approach to process collections. Common operations include `filter` (to select elements), `map` (to transform elements), and `collect` (to aggregate results).
```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6);

// Filter even numbers, square them, and collect to a list
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
```

---

### 3. What is the difference between an Abstract Class and an Interface?
**Answer:**
- **Abstract Class:** Can have both abstract and concrete methods. Can have state (instance variables). A class can extend only one abstract class. Used when you want to share code among closely related classes.
- **Interface:** Prior to Java 8, could only have abstract methods. Now supports `default` and `static` methods. Does not maintain state (variables are implicitly `public static final`). A class can implement multiple interfaces. Used to define a contract for unrelated classes.

---

### 4. How does a HashMap work internally in Java? Difference between HashMap and TreeMap?
**Answer:**
`HashMap` works on the principle of Hashing. It uses an array of `Node` (or `Map.Entry`) objects.
- When `put(key, value)` is called, it calculates the hash code of the key to determine the bucket index.
- If the bucket is empty, the node is stored there. If not (a collision), it checks `equals()` — if keys match, the value is updated; if not, the node is appended to a linked list.
- Since Java 8, if a bucket's linked list exceeds 8 entries, it transforms into a **Red-Black tree** (O(log n) lookup).

| Feature       | HashMap                        | TreeMap                          |
| ------------- | ------------------------------ | -------------------------------- |
| Order         | No guaranteed order            | Sorted by keys (natural/custom)  |
| Null keys     | Allows one null key            | Does **not** allow null keys     |
| Performance   | O(1) average for get/put       | O(log n) for get/put             |
| Internal DS   | Hash table (array + linked list / tree) | Red-Black Tree          |

---

### 5. Write a SQL query to find the second highest (or Nth highest) salary.
**Answer:**
```sql
-- Second highest using subquery
SELECT MAX(salary) FROM Employees WHERE salary < (SELECT MAX(salary) FROM Employees);

-- Nth highest using DENSE_RANK (works in all modern databases)
WITH RankedSalaries AS (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rnk
    FROM Employees
)
SELECT DISTINCT salary FROM RankedSalaries WHERE rnk = N;

-- Using LIMIT/OFFSET (MySQL specific, e.g., 3rd highest)
SELECT DISTINCT salary FROM Employees ORDER BY salary DESC LIMIT 1 OFFSET 2;
```

---

### 6. Explain the concepts of Joins in SQL. What are the different types?
**Answer:**
A JOIN clause combines rows from two or more tables based on a related column.
- **INNER JOIN:** Returns records with matching values in both tables.
- **LEFT (OUTER) JOIN:** Returns all records from the left table, and matched records from the right.
- **RIGHT (OUTER) JOIN:** Returns all records from the right table, and matched records from the left.
- **FULL (OUTER) JOIN:** Returns all records when there is a match in either table.
- **CROSS JOIN:** Returns the Cartesian product of both tables.
- **SELF JOIN:** Joins a table with itself, useful for hierarchical data.

---

### 7. What is the difference between `String`, `StringBuilder`, and `StringBuffer`? Why is String immutable?
**Answer:**
| Feature       | String           | StringBuilder    | StringBuffer     |
| ------------- | ---------------- | ---------------- | ---------------- |
| Mutability    | Immutable        | Mutable          | Mutable          |
| Thread Safety | Yes (immutable)  | No               | Yes (synchronized)|
| Performance   | Slow (creates new objects) | Fastest | Slower than SB   |
| Use case      | Few modifications | Single-threaded  | Multi-threaded   |

**Why is String immutable?**
- **String Pool:** Immutability allows Java to cache strings in the String Pool, saving memory when the same string literal is reused.
- **Security:** Strings are used in class loading, network connections (URLs), and database connection parameters. Making them immutable prevents malicious modification.
- **Hashcode caching:** Since a String's value never changes, its hash code is cached at creation, making `HashMap` key lookups faster.

---

### 8. What is the difference between Method Overloading and Method Overriding?
**Answer:**
| Feature       | Overloading (Compile-time)     | Overriding (Run-time)            |
| ------------- | ------------------------------ | -------------------------------- |
| Scope         | Within the same class          | Between parent and child class   |
| Parameters    | Must differ                    | Must be the same                 |
| Return type   | Can differ                     | Same or covariant                |
| Binding       | Static (at compile time)       | Dynamic (at run time)            |
| `static` methods | Can be overloaded            | Cannot be overridden             |

---

### 9. Write a program to reverse a string and check if it is a palindrome using O(n/2).
**Answer:**
```java
// Reversing a string
public String reverse(String str) {
    return new StringBuilder(str).reverse().toString();
}

// Checking Palindrome in O(n/2) - Two-pointer approach
public boolean isPalindrome(String str) {
    int left = 0, right = str.length() - 1;
    while (left < right) {
        if (str.charAt(left) != str.charAt(right)) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

---

### 10. Explain Hibernate ORM and write code for a One-to-Many relationship.
**Answer:**
Hibernate is an ORM (Object Relational Mapping) framework that maps Java objects to database tables, eliminating the need for manual JDBC code.
```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Employee> employees = new ArrayList<>();
}

@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
}
```
- `@OneToMany(mappedBy)` on the parent side indicates the inverse relationship.
- `@ManyToOne` with `@JoinColumn` on the child side owns the foreign key.
- `CascadeType.ALL` propagates persist/merge/remove operations from parent to children.

---

### 11. How many ways can you define a Bean in a Spring Boot application?
**Answer:**
There are 3 primary ways:
1. **Stereotype Annotations (`@Component`, `@Service`, `@Repository`, `@Controller`):** Annotate a class and let `@ComponentScan` (implicit in `@SpringBootApplication`) discover it.
2. **`@Bean` methods in `@Configuration` classes:** Manually declare beans inside a `@Configuration` class.
3. **Auto-configuration:** Spring Boot's `@EnableAutoConfiguration` (part of `@SpringBootApplication`) automatically configures beans from framework-provided starters based on classpath jars.

```java
// Way 1: Stereotype
@Service
public class UserService { }

// Way 2: @Bean method
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

---

### 12. What is Dependency Injection (DI) and Inversion of Control (IoC) in Spring?
**Answer:**
- **IoC:** A design principle where the framework (IoC container) takes control of object creation and lifecycle management, instead of the programmer creating them manually with `new`.
- **DI:** A specific implementation of IoC. Dependencies are injected into objects at runtime by the Spring container.

**Types of Injection:**
- **Constructor Injection (Recommended):** Dependencies are final and immutable. Makes testing easy.
- **Setter Injection:** Optional dependencies.
- **Field Injection (`@Autowired`):** Concise but harder to test and makes dependencies invisible.

```java
@Service
public class OrderService {
    private final PaymentService paymentService; // Constructor Injection

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

---

### 13. What is MVC pattern? Explain routing in MVC and Angular.
**Answer:**
**MVC (Model-View-Controller):**
- **Model:** Manages data, validation, and business rules.
- **View:** Renders UI using model data. Stays presentation-focused.
- **Controller:** Handles incoming HTTP requests, interacts with the Model, and returns the appropriate View.

**Routing:**
- **In Spring MVC:** Routes map incoming HTTP requests to controller methods using `@RequestMapping`, `@GetMapping`, `@PostMapping`, etc.
- **In Angular:** The `RouterModule` maps URL paths to components. Defined in `app-routing.module.ts`:
```typescript
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users', component: UserListComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: '**', component: NotFoundComponent } // Wildcard
];
```

---

### 14. Explain the difference between GET and POST methods in REST APIs.
**Answer:**
| Feature       | GET                          | POST                           |
| ------------- | ---------------------------- | ------------------------------ |
| Purpose       | Retrieve data                | Create/submit data             |
| Idempotent    | Yes                          | No                             |
| Parameters    | In URL query string          | In request body                |
| Caching       | Can be cached                | Not cached                     |
| Security      | Less secure (visible in URL) | More secure                    |
| Data limit    | URL length restrictions      | No size limit                  |

---

### 15. Explain the concept of Normalization in databases.
**Answer:**
Normalization organizes data to reduce redundancy and improve integrity.
- **1NF:** Each column must contain atomic (indivisible) values. No repeating groups.
- **2NF:** Must be in 1NF. All non-key attributes must be fully dependent on the entire primary key (no partial dependency).
- **3NF:** Must be in 2NF. No transitive dependency (non-key attributes should not depend on other non-key attributes).
- **BCNF:** A stronger version of 3NF. Every determinant must be a candidate key.

---

### 16. What is a Singleton Design Pattern and how do you implement it in Java?
**Answer:**
Ensures a class has only one instance and provides a global point of access.
```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {} // Private constructor

    public static Singleton getInstance() {
        if (instance == null) {                   // First check (no lock)
            synchronized (Singleton.class) {
                if (instance == null) {           // Double-checked locking
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
- In Spring, all beans are Singletons by default (singleton scope).

---

### 17. Write a program to find two numbers in an array that add up to a target sum (Two Sum).
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
**Time Complexity:** O(n) | **Space Complexity:** O(n)

---

### 18. Write a program to find the middle element of a LinkedList in one pass. Also detect a loop.
**Answer:**
Both use the **Floyd's Tortoise and Hare** (fast/slow pointer) approach.
```java
// Find middle element
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

### 19. What are the different types of indexes in a database?
**Answer:**
An index is a data structure (often a B-Tree) that speeds up data retrieval.
- **Primary Index:** Auto-created for the primary key. Unique and not null.
- **Unique Index:** Ensures all values in the indexed column are unique.
- **Clustered Index:** Determines the physical order of data. Only one per table.
- **Non-Clustered Index:** Contains sorted pointers to actual data rows. Multiple per table.
- **Composite Index:** Index on multiple columns, useful for queries filtering on those columns together.

---

### 20. What are Access Modifiers in Java?
**Answer:**
| Modifier    | Same Class | Same Package | Subclass (other pkg) | Everywhere |
| ----------- | ---------- | ------------ | -------------------- | ---------- |
| `private`   | ✅         | ❌           | ❌                   | ❌         |
| `default`   | ✅         | ✅           | ❌                   | ❌         |
| `protected` | ✅         | ✅           | ✅                   | ❌         |
| `public`    | ✅         | ✅           | ✅                   | ✅         |

---

### 21. What is the difference between an Array and an ArrayList in Java?
**Answer:**
| Feature       | Array                        | ArrayList                      |
| ------------- | ---------------------------- | ------------------------------ |
| Size          | Fixed at creation            | Dynamic (grows/shrinks)        |
| Type          | Can hold primitives & objects| Only objects (autoboxing for primitives) |
| Performance   | Faster (direct access)       | Slightly slower (wrapper overhead)|
| Methods       | No built-in methods          | Rich API (`add`, `remove`, `contains`) |
| Generics      | Not type-safe with generics  | Type-safe with generics        |

---

### 22. How does Multithreading work in Java? `Runnable` vs `Thread`?
**Answer:**
- **Extending `Thread`:** Override `run()`. Cannot extend another class (no multiple inheritance).
- **Implementing `Runnable`:** Implement `run()` and pass to a `Thread` object. Preferred — separates task from runner, allows extending another class.

```java
// Using Runnable (preferred)
Runnable task = () -> System.out.println("Running in: " + Thread.currentThread().getName());
Thread thread = new Thread(task);
thread.start();
```
**Key Concepts:** `synchronized`, `volatile`, `wait()/notify()`, `ExecutorService`, `CompletableFuture`.

---

### 23. Explain the Java Memory Model — Stack vs Heap.
**Answer:**
- **Stack:** Stores method call frames, local variables, and references. Follows LIFO. Each thread has its own stack. Memory auto-freed when method completes. Fast access.
- **Heap:** Stores all objects created with `new`. Shared across all threads. Managed by the Garbage Collector. Slower access.

```
Thread 1 Stack           Heap
┌──────────────┐     ┌──────────────────┐
│ int x = 5    │     │ Employee obj     │
│ ref → obj ───┼────▶│  name = "John"   │
└──────────────┘     │  age = 30        │
                     └──────────────────┘
```

---

### 24. Explain Exception Handling in Java — Checked vs Unchecked Exceptions.
**Answer:**
- **Checked Exceptions:** Checked at compile-time. Must be handled with try-catch or declared with `throws`. Examples: `IOException`, `SQLException`.
- **Unchecked Exceptions:** Occur at runtime. Represent programming bugs. Examples: `NullPointerException`, `ArrayIndexOutOfBoundsException`.
- **Error:** Serious problems not meant to be caught. Examples: `OutOfMemoryError`, `StackOverflowError`.

```
Throwable
├── Error (OutOfMemoryError, StackOverflowError)
└── Exception
    ├── Checked (IOException, SQLException)
    └── RuntimeException / Unchecked (NullPointerException, ClassCastException)
```

---

### 25. What is Connection Pooling? Why is it important?
**Answer:**
Connection pooling is a technique to reuse a pool of pre-created database connections instead of opening/closing a new one for every request.
- **Without pooling:** Each request creates a new DB connection (expensive: TCP handshake, authentication) and destroys it after use.
- **With pooling (e.g., HikariCP in Spring Boot):** A pool of connections is created at startup. Requests borrow a connection, use it, and return it to the pool.

**Benefits:** Drastically reduces connection overhead, improves response time, and limits the number of concurrent DB connections to prevent overloading.
```yaml
# application.yml (Spring Boot + HikariCP)
spring:
  datasource:
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
```

---

### 26. Write a program to group anagrams from a given list of words.
**Answer:**
```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars); // "eat" -> "aet", "tea" -> "aet"
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
// Input:  ["eat","tea","tan","ate","nat","bat"]
// Output: [["eat","tea","ate"],["tan","nat"],["bat"]]
```
**Time Complexity:** O(N * K log K) where N = number of strings, K = max length of a string.

---

### 27. What are Angular Lifecycle Hooks? Explain the most important ones.
**Answer:**
Angular components/directives go through a lifecycle managed by Angular. Key hooks:
| Hook                | When it fires                                      |
| ------------------- | -------------------------------------------------- |
| `ngOnChanges`       | When an `@Input` property value changes             |
| `ngOnInit`          | Once, after the first `ngOnChanges` — initialization logic |
| `ngDoCheck`         | On every change detection run                       |
| `ngAfterViewInit`   | After the component's view (and child views) are initialized |
| `ngOnDestroy`       | Just before the component is destroyed — cleanup (unsubscribe observables, detach event listeners) |

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

### 28. What is Angular Data Binding? Explain the types.
**Answer:**
Data binding syncs data between the component class (TypeScript) and the template (HTML).

| Type                  | Syntax                   | Direction            |
| --------------------- | ------------------------ | -------------------- |
| Interpolation         | `{{ expression }}`       | Component → View     |
| Property Binding      | `[property]="expr"`      | Component → View     |
| Event Binding         | `(event)="handler()"`    | View → Component     |
| Two-way Binding       | `[(ngModel)]="property"` | Both directions      |

```html
<!-- Interpolation -->
<h1>{{ title }}</h1>

<!-- Property Binding -->
<img [src]="imageUrl">

<!-- Event Binding -->
<button (click)="onClick()">Click Me</button>

<!-- Two-way Binding (requires FormsModule) -->
<input [(ngModel)]="username">
```

---

### 29. Explain `HashMap` vs `ConcurrentHashMap` in Java.
**Answer:**
| Feature           | HashMap                     | ConcurrentHashMap             |
| ----------------- | --------------------------- | ----------------------------- |
| Thread Safety     | Not thread-safe             | Thread-safe                   |
| Null keys/values  | Allows 1 null key, N null values | No null keys or values    |
| Locking mechanism | None                        | Segment-level / bucket-level locking (Java 8+) |
| Performance       | Faster in single-threaded   | Efficient in multi-threaded   |
| Iterator          | Fail-fast (throws `ConcurrentModificationException`) | Fail-safe (weakly consistent) |

---

### 30. What design patterns have you used? Explain Factory and Observer patterns.
**Answer:**
**Factory Pattern:** Creates objects without exposing instantiation logic to the client.
```java
public class NotificationFactory {
    public static Notification create(String type) {
        return switch (type) {
            case "EMAIL" -> new EmailNotification();
            case "SMS"   -> new SmsNotification();
            case "PUSH"  -> new PushNotification();
            default -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}
```

**Observer Pattern:** Defines a one-to-many dependency. When the subject changes state, all registered observers are notified.
- Used in event-driven systems, Angular's `EventEmitter`, and RxJS `Observable`.
- Spring's `ApplicationEventPublisher` is a built-in implementation.

---

### 31. Explain `@Component` vs `@Service` vs `@Repository` vs `@Controller` in Spring.
**Answer:**
All are specializations of `@Component` and are auto-detected by `@ComponentScan`.
| Annotation     | Layer            | Special Behavior                                    |
| -------------- | ---------------- | --------------------------------------------------- |
| `@Component`   | Generic          | Base annotation for any Spring-managed bean          |
| `@Service`     | Business Logic   | No extra behavior, but indicates service layer intent|
| `@Repository`  | Data Access (DAO)| Translates persistence exceptions to Spring's `DataAccessException` |
| `@Controller`  | Web / MVC        | Marks a class as a web controller handling HTTP requests |

---

### 32. How do you optimize a slow SQL query?
**Answer:**
1. **Use `EXPLAIN` / `EXPLAIN ANALYZE`** to understand the query execution plan.
2. **Add appropriate indexes** on columns used in `WHERE`, `JOIN`, and `ORDER BY`.
3. **Avoid `SELECT *`** — select only required columns.
4. **Fix N+1 query problems** — use `JOIN FETCH` in JPA or batch fetching.
5. **Use pagination** (`LIMIT/OFFSET` or keyset pagination) for large result sets.
6. **Denormalize** read-heavy tables if necessary.
7. **Cache results** using Redis/Ehcache for data that rarely changes.

---

### 33. What is the difference between `final`, `finally`, and `finalize` in Java?
**Answer:**
- **`final`:** A keyword used with variables (constant), methods (cannot be overridden), and classes (cannot be inherited).
- **`finally`:** A block in try-catch that always executes regardless of whether an exception is thrown — used for resource cleanup.
- **`finalize()`:** A method in `Object` class called by the Garbage Collector before destroying an object. **Deprecated since Java 9** — use `try-with-resources` or `Cleaner` instead.

---

### 34. What are Angular Services and how does Dependency Injection work in Angular?
**Answer:**
Services are classes decorated with `@Injectable()` that encapsulate reusable business logic, data fetching, or state management.
```typescript
@Injectable({ providedIn: 'root' }) // Singleton — available app-wide
export class UserService {
  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
}
```
Angular's DI system injects the service into components via the constructor:
```typescript
@Component({ selector: 'app-users', templateUrl: './users.component.html' })
export class UsersComponent {
  constructor(private userService: UserService) {} // Injected automatically
}
```

---

### 35. Sort an array without using built-in sorting — implement Bubble Sort.
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
        if (!swapped) break; // Optimization: already sorted
    }
}
```
**Time Complexity:** O(n²) worst/average, O(n) best (already sorted with optimization).

---

### 36. What are Functional Interfaces and Lambda Expressions in Java 8?
**Answer:**
A **Functional Interface** has exactly one abstract method and can be used as the target for lambda expressions.
```java
@FunctionalInterface
interface Calculator {
    int operate(int a, int b);
}

// Lambda expression
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

System.out.println(add.operate(5, 3));      // 8
System.out.println(multiply.operate(5, 3)); // 15
```
**Built-in Functional Interfaces:** `Predicate<T>`, `Function<T,R>`, `Consumer<T>`, `Supplier<T>`.

---

### 37. What is Spring Boot Actuator? What endpoints does it provide?
**Answer:**
Spring Boot Actuator provides production-ready features to monitor and manage your application.
- `/actuator/health` — Health status of the application.
- `/actuator/info` — Application info.
- `/actuator/metrics` — Application metrics (JVM memory, CPU, HTTP requests).
- `/actuator/env` — Environment properties.
- `/actuator/loggers` — View and change log levels at runtime.

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics
```

---

### 38. What are Angular Pipes? How do you create a custom pipe?
**Answer:**
Pipes transform data in templates. Built-in pipes: `date`, `uppercase`, `lowercase`, `currency`, `async`.
```typescript
// Custom pipe
@Pipe({ name: 'truncate' })
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number = 50): string {
    return value.length > limit ? value.substring(0, limit) + '...' : value;
  }
}
```
```html
<!-- Usage in template -->
<p>{{ longText | truncate:100 }}</p>
<p>{{ today | date:'dd/MM/yyyy' }}</p>
```

---

### 39. What are Triggers, Cursors, and Stored Procedures in SQL?
**Answer:**
- **Trigger:** A block of SQL that automatically fires in response to certain events (INSERT, UPDATE, DELETE) on a table. Used for auditing, validation, or cascading changes.
- **Cursor:** A database object used to retrieve rows one at a time (row-by-row processing). Slower than set-based operations — use only when row-level logic is needed.
- **Stored Procedure:** A precompiled set of SQL statements stored on the database server. Improves performance (compiled once, executed many times), promotes reusability, and reduces network traffic.

---

### 40. How do you handle Global Exception Handling in a Spring Boot REST API?
**Answer:**
Use `@ControllerAdvice` (or `@RestControllerAdvice`) with `@ExceptionHandler`.
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(HttpStatus.NOT_FOUND.value(), ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        String msg = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return new ResponseEntity<>(new ErrorResponse(400, msg), HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        return new ResponseEntity<>(new ErrorResponse(500, "Internal Server Error"), HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```
