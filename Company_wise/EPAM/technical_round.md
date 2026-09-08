# EPAM Technical Round Questions

> **Interview Format Note:** The **First Round (L1)** is typically a high-paced technical screening focusing heavily on **live coding (DSA/Programming)** and **rapid-fire conceptual questions** across the core stack. Candidates are expected to write code and answer quick-fire theory questions back-to-back.
> **Skill Set:** Java, Hibernate, Data Structures, Programming, Spring Boot, Oracle PL/SQL, Microservices, REST API, Docker, Kubernetes, Infrastructure, Oracle Web Services
> **Sources:** AmbitionBox, Glassdoor, GeeksforGeeks, LeetCode
> **Note:** Questions are ranked strictly by interview frequency (descending) based on aggregated data.

---

### 1. 🌐 How does a HashMap work internally in Java? What happens when there are collisions?
**Answer:**
- **Internals:** `HashMap` uses an array of `Node` (or bucket) objects. On `put(key, value)`, it calculates the hash code, derives the bucket index, and places the node.
- **Collisions:** If multiple keys map to the same index (collision), they are stored as a Linked List.
- **Java 8 Optimization (Treeification):** If a bucket's linked list grows beyond 8 elements (`TREEIFY_THRESHOLD`), it transforms into a Red-Black tree to improve worst-case search time from O(n) to O(log n).

| Scenario | Data Structure | Time Complexity (Search) |
| --- | --- | --- |
| Few collisions (<=8) | Linked List | O(n) |
| High collisions (>8) | Red-Black Tree | O(log n) |

---

### 2. 🏢 What is the difference between HashMap and ConcurrentHashMap? How is thread safety achieved?
**Answer:**
- **HashMap:** Not thread-safe. Concurrent modifications can lead to a `ConcurrentModificationException` or an infinite loop during resizing (pre-Java 8).
- **ConcurrentHashMap:** Thread-safe, designed for highly concurrent environments.

**How thread safety is achieved (Java 8+):**
- It abandons segment-level locking (used in Java 7) and uses **CAS (Compare-And-Swap)** operations and `synchronized` blocks at the **bucket level** (Node level).
- This means multiple threads can write to different buckets simultaneously without blocking each other.
- Null keys and null values are **not** allowed.

---

### 3. 🌐 Write a Java Streams API pipeline to filter even numbers, square them, and collect them to a list.
**Answer:**
- The Streams API (introduced in Java 8) provides a declarative way to process collections.

```java
import java.util.List;
import java.util.stream.Collectors;

public class StreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8);

        List<Integer> result = numbers.stream()
                .filter(n -> n % 2 == 0) // Filter even
                .map(n -> n * n)         // Square them
                .collect(Collectors.toList());

        System.out.println(result); // Output: [4, 16, 36, 64]
    }
}
```

---

### 4. 🏢 Explain Spring Boot Auto-configuration. How does it work internally?
**Answer:**
- **Purpose:** Automatically configures your Spring application based on the jar dependencies present in the classpath.
- **Trigger:** The `@EnableAutoConfiguration` annotation (which is part of `@SpringBootApplication`).
- **Internal Working:** 
  - At startup, Spring searches for the `META-INF/spring.factories` file (or `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` in Spring Boot 2.7/3.0+).
  - It loads configuration classes listed there.
  - It applies **Conditions** (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`) to decide whether a bean should be created.
  - E.g., if `HikariDataSource` is on the classpath and no other `DataSource` bean exists, it creates one automatically.

---

### 5. 🌐 What are Microservices? How do they differ from a Monolithic architecture?
**Answer:**
- **Monolithic:** A single deployable unit containing all business logic, UI, and data access. Easy to start, hard to scale independently.
- **Microservices:** An architectural style that structures an application as a collection of loosely coupled, independently deployable services, each owning its database.

| Feature | Monolithic | Microservices |
| --- | --- | --- |
| Deployment | Single large unit | Independent units |
| Scaling | Scale entire app | Scale specific services independently |
| Technology | Single stack | Polyglot (can use different tech per service) |
| Failure Impact | One bug can crash the whole app | Isolated (handled via Circuit Breakers) |

---

### 6. 🌐 Explain REST API constraints. How is it different from SOAP?
**Answer:**
- **REST (Representational State Transfer)** is an architectural style based on HTTP, utilizing standard methods (GET, POST, PUT, DELETE).
- **Constraints of REST:**
  1. Client-Server Architecture (Decoupled).
  2. Stateless (No client context stored on server between requests).
  3. Cacheability (Responses must define themselves as cacheable or not).
  4. Layered System (Client cannot tell if it's connected to end server or intermediary).
  5. Uniform Interface (Resource identification in requests, HATEOAS).

| Feature | REST | SOAP (Web Services) |
| --- | --- | --- |
| Protocol | Uses HTTP mostly | Uses WSDL, XML over HTTP/SMTP |
| Data Format | JSON, XML, HTML, plain text | XML only |
| Security | HTTPS, OAuth2, JWT | WS-Security, SSL |
| State | Stateless | Can be stateful |

---

### 7. 🌐 Explain the Java Exception Hierarchy. Difference between Checked and Unchecked exceptions?
**Answer:**
- The root class is `Throwable`, which has two main subclasses: `Error` and `Exception`.

```text
Throwable
├── Error (e.g., OutOfMemoryError - usually unrecoverable)
└── Exception
    ├── Checked Exceptions (e.g., IOException, SQLException)
    └── RuntimeException (Unchecked) (e.g., NullPointerException)
```

| Type | When Checked | Handling | Example |
| --- | --- | --- | --- |
| Checked | Compile-time | Must use `try-catch` or `throws` | `IOException` |
| Unchecked | Runtime | Not strictly required to handle | `NullPointerException` |

---

### 8. 🏢 What is the `volatile` keyword in Java? How does it relate to thread visibility?
**Answer:**
- **Visibility Problem:** In multithreading, each thread may cache variables in its local CPU cache. If one thread updates a variable, others might not see the updated value immediately.
- **The Solution:** Declaring a variable as `volatile` forces all reads and writes to go straight to the **main memory**, bypassing CPU caches.
- **Limitation:** `volatile` guarantees visibility but **not atomicity**. For compound operations like `count++`, you still need `synchronized` or `AtomicInteger`.

---

### 9. 🏢 Explain the Saga Pattern. How do you handle distributed transactions in Microservices?
**Answer:**
- In microservices, the traditional two-phase commit (2PC) is often too slow and locks resources. 
- **Saga Pattern** breaks a distributed transaction into a sequence of local transactions.
- If a local transaction fails, the Saga executes **compensating transactions** to undo the changes made by previous local transactions.

**Implementation Types:**
1. **Choreography:** Services publish and subscribe to domain events independently. (No central coordinator).
2. **Orchestration:** A central Orchestrator service tells participating services what local transactions to execute.

---

### 10. 🌐 Explain `@Transactional` propagation and isolation levels in Spring / Hibernate.
**Answer:**
- `@Transactional` provides declarative transaction management.

**Key Propagation Types:**
- `REQUIRED` (Default): Joins the active transaction if one exists; otherwise, creates a new one.
- `REQUIRES_NEW`: Suspends the active transaction and always creates a new one.

**Isolation Levels (addressing concurrency issues like dirty reads):**
- `READ_UNCOMMITTED`: Lowest isolation, allows dirty reads.
- `READ_COMMITTED`: Prevents dirty reads.
- `REPEATABLE_READ`: Prevents dirty and non-repeatable reads.
- `SERIALIZABLE`: Highest isolation, locks tables/rows, prevents phantom reads (slowest).

---

### 11. 🌐 Oracle PL/SQL: Explain the difference between Implicit and Explicit Cursors. What are Triggers?
**Answer:**
- **Cursors** are used to retrieve and process data row by row in PL/SQL.
  - **Implicit Cursors:** Automatically created by Oracle for DML statements (`INSERT`, `UPDATE`, `DELETE`) and single-row `SELECT INTO`. Attributes like `SQL%ROWCOUNT`, `SQL%FOUND` are used to check status.
  - **Explicit Cursors:** Declared and managed by the programmer for queries returning multiple rows. Steps: `DECLARE`, `OPEN`, `FETCH`, `CLOSE`.
- **Triggers:** Named PL/SQL blocks executed automatically in response to specific events on a table or view (e.g., `BEFORE INSERT`, `AFTER UPDATE`). Used heavily for auditing or enforcing complex business rules.

```sql
-- Explicit Cursor Example
DECLARE
  CURSOR emp_cur IS SELECT first_name FROM employees WHERE department_id = 10;
  v_name employees.first_name%TYPE;
BEGIN
  OPEN emp_cur;
  LOOP
    FETCH emp_cur INTO v_name;
    EXIT WHEN emp_cur%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE(v_name);
  END LOOP;
  CLOSE emp_cur;
END;
```

---

### 12. 🏢 Explain Docker Image vs Container. Why use multi-stage builds?
**Answer:**
- **Image:** A read-only template with instructions for creating a container (contains code, runtime, libraries, env vars).
- **Container:** A runnable, isolated instance of an image.

**Multi-stage builds:**
- Used to keep the final Docker image small and secure.
- **Stage 1 (Builder):** Uses a heavy JDK image to compile the code (`mvn clean package`).
- **Stage 2 (Runtime):** Uses a lightweight JRE image, copying only the compiled `.jar` from Stage 1.

```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# Stage 2: Run
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/app.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### 13. 🌐 Kubernetes (Infrastructure): What are Pods, Deployments, and Services?
**Answer:**
- **Pod:** The smallest deployable computing unit in K8s. Represents a single instance of a running process, usually wrapping one (or occasionally tightly-coupled multiple) containers.
- **Deployment:** A higher-level abstraction that manages ReplicaSets. It provides declarative updates (rolling updates, rollbacks) to Pods and ensures the desired number of Pods are always running.
- **Service:** An abstraction that defines a logical set of Pods and a policy by which to access them (ClusterIP, NodePort, LoadBalancer). It provides a stable IP address and DNS name, acting as a load balancer for the underlying Pods.

---

### 14. 🌐 DSA / Programming: How do you reverse a string without using built-in functions?
**Answer:**
```java
public class StringReversal {
    public static String reverse(String input) {
        if (input == null || input.isEmpty()) return input;
        
        char[] characters = input.toCharArray();
        int left = 0;
        int right = characters.length - 1;
        
        while (left < right) {
            // Swap characters
            char temp = characters[left];
            characters[left] = characters[right];
            characters[right] = temp;
            left++;
            right--;
        }
        return new String(characters);
    }
}
```
**Time Complexity:** O(n) | **Space Complexity:** O(n) (due to char array)

---

### 15. 🌐 DSA: Two Sum Problem. Write an optimized solution.
**Answer:**
- **Problem:** Given an array of integers and a target sum, return the indices of the two numbers that add up to the target.
- **Optimized Solution:** Use a HashMap to store the difference needed to reach the target.

```java
import java.util.HashMap;
import java.util.Map;

public class TwoSum {
    public int[] findTwoSum(int[] nums, int target) {
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
}
```
**Time Complexity:** O(n) | **Space Complexity:** O(n)

---

### 16. 🏢 Explain Garbage Collection tuning. Difference between G1 and ZGC?
**Answer:**
- **Garbage Collection (GC)** automatically reclaims memory by deleting unreferenced objects from the Heap.

| Collector | Characteristics | Use Case |
| --- | --- | --- |
| **G1 GC (Garbage-First)** | Default in Java 9+. Divides heap into regions. Predictable pause times. | General purpose applications with moderate pause-time requirements. |
| **ZGC (Z Garbage Collector)** | Available in Java 11+. Performs expensive work concurrently. Sub-millisecond pause times regardless of heap size. | Applications requiring ultra-low latency and large heaps (e.g., Financial trading platforms). |

---

### 17. 🏢 System Design: How do you implement Centralized Logging in a Microservices ecosystem?
**Answer:**
- In microservices, logs are distributed across many instances (Infrastructure). Centralized logging is critical for debugging.
- **Standard Stack:** ELK (Elasticsearch, Logstash, Kibana) or EFK (Fluentd instead of Logstash).
- **Implementation:**
  1. Microservices generate logs (preferably in JSON format) and output to `stdout`.
  2. A daemonset agent like **FluentBit** or **Filebeat** collects these logs from Kubernetes nodes.
  3. Logs are sent to **Elasticsearch** (a highly scalable search engine) for indexing and storage.
  4. Developers query and visualize logs using **Kibana**.
- **Crucial Feature:** Generate a unique **Trace ID** at the API Gateway and pass it via HTTP headers (using Spring Cloud Sleuth / Micrometer Tracing) to correlate logs for a single request across all microservices.

---

### 18. 🏢 What is the Circuit Breaker pattern? How does Resilience4j work?
**Answer:**
- **Purpose:** Prevents an application from repeatedly trying to execute an operation that's likely to fail, saving resources and allowing the failing service to recover.
- **States:**
  - **CLOSED:** Normal operation. Requests flow freely.
  - **OPEN:** Failure threshold reached. Requests are blocked immediately, and a fallback method is executed.
  - **HALF-OPEN:** After a timeout, allows a limited number of test requests to see if the downstream service has recovered.

```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;

@Service
public class InventoryService {

    @CircuitBreaker(name = "inventoryService", fallbackMethod = "fallbackInventory")
    public Inventory checkInventory(String productId) {
        // HTTP call to downstream service
        return restTemplate.getForObject("http://inventory-api/check/" + productId, Inventory.class);
    }

    public Inventory fallbackInventory(String productId, Throwable t) {
        // Fallback response when Circuit is OPEN
        return new Inventory(productId, 0, "DOWN");
    }
}
```

---

### 19. 🌐 DSA / Programming: How do you merge two sorted arrays into a single sorted array in-place?
**Answer:**
- **Problem:** Given two sorted integer arrays `nums1` and `nums2`, merge `nums2` into `nums1` as one sorted array. Assume that `nums1` has a size equal to $m + n$ such that it has enough space to hold additional elements from `nums2`.
- **Optimized Solution:** Start from the end of both arrays and place the larger element at the end of `nums1`. This avoids shifting elements and uses $O(1)$ extra space.

```java
public class MergeSortedArrays {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m - 1; // Last element in nums1's initial valid part
        int j = n - 1; // Last element in nums2
        int k = m + n - 1; // Last position in nums1

        // Iterate backwards and place the largest element at the end
        while (j >= 0) {
            if (i >= 0 && nums1[i] > nums2[j]) {
                nums1[k] = nums1[i];
                i--;
            } else {
                nums1[k] = nums2[j];
                j--;
            }
            k--;
        }
    }
}
```
**Time Complexity:** O(m + n) | **Space Complexity:** O(1)

---

### 20. 🌐 DSA / Programming: Write an optimized program to determine if a string of parentheses is valid.
**Answer:**
- **Problem:** Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid (brackets must be closed in the correct order).
- **Optimized Solution:** Use a Stack (or an ArrayDeque for better performance) to keep track of opening brackets and match them with incoming closing brackets.

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class ValidParentheses {
    public boolean isValid(String s) {
        // ArrayDeque is faster than Stack in Java since it doesn't synchronize methods
        Deque<Character> stack = new ArrayDeque<>();
        
        for (char c : s.toCharArray()) {
            if (c == '(' || c == '{' || c == '[') {
                stack.push(c);
            } else {
                if (stack.isEmpty()) return false;
                char top = stack.pop();
                if ((c == ')' && top != '(') ||
                    (c == '}' && top != '{') ||
                    (c == ']' && top != '[')) {
                    return false;
                }
            }
        }
        // If stack is empty, all open brackets were properly closed
        return stack.isEmpty();
    }
}
```
**Time Complexity:** O(n) | **Space Complexity:** O(n)
