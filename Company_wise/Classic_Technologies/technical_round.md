# Classic Technologies And Business Solutions (CTEBS) Technical Round Questions

> **Interview Format Note:** The interviews typically consist of technical rounds focusing on Java, Spring Boot, Angular, and Database interactions (MSSQL/Oracle). There is a strong emphasis on Core Java fundamentals, live project walkthroughs, and problem-solving puzzles.
> **Skill Set:** Core Java, Spring Boot, Angular, SQL (Joins/Stored Procedures), Hibernate/JPA.
> **Sources:** AmbitionBox, Web Search
> **Note:** Questions are ranked by interview frequency (descending) based on aggregated data.

---

### Q1. [🏢] Tell me about yourself, your skills, and walk me through your recent project architecture.
**Answer:**
- **Project Explanation:** Be prepared to explain the complete architecture of your most recent project, specifically focusing on how the Java/Spring Boot backend connects to the database and the Angular frontend.
- **Skills Highlight:** Emphasize core competencies like Object-Oriented Programming, Spring Boot REST APIs, and database query optimization.
- **Problem Solving:** Mention any complex challenges you faced (e.g., performance bottlenecks) and how you resolved them.

---

### Q2. [🏢] How do you create a REST Controller and handle CRUD operations in Spring Boot?
**Answer:**
- **REST Controller:** Uses the `@RestController` annotation which combines `@Controller` and `@ResponseBody`. It handles incoming HTTP requests.
- **CRUD Mapping:** Uses standard HTTP methods mapped via annotations (`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`).
- **Data Binding:** `@RequestBody` is used to map the JSON payload to a Java object, while `@PathVariable` or `@RequestParam` map URL parameters.

```java
import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;

@RestController
@RequestMapping("/api/employees")
public class EmployeeController {

    private final EmployeeService service;

    public EmployeeController(EmployeeService service) {
        this.service = service;
    }

    @PostMapping
    public ResponseEntity<Employee> createEmployee(@RequestBody Employee employee) {
        return ResponseEntity.ok(service.save(employee));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Employee> getEmployee(@PathVariable Long id) {
        return ResponseEntity.ok(service.findById(id));
    }
}
```

---

### Q3. [🏢] Core Java: Explain the Collections framework. What is the difference between `==` and `.equals()`?
**Answer:**
- **Collections:** A framework that provides an architecture to store and manipulate a group of objects. Key interfaces include `List`, `Set`, and `Map`.
- **Difference:**
  - `==` is an operator that compares object references (memory addresses) to check if they point to the exact same object.
  - `.equals()` is a method (overridden from `Object` class) used to compare the actual content or state of two objects.

| Feature | `==` Operator | `.equals()` Method |
| --- | --- | --- |
| **Type** | Operator | Method |
| **Comparison** | Reference/Memory Address | Object Value/State |
| **Overridable** | No | Yes |

---

### Q4. [🏢] Write a SQL query to join two tables and explain what Stored Procedures are.
**Answer:**
- **Stored Procedure:** A prepared SQL code that you can save so the code can be reused over and over again. It is compiled and stored in the database, reducing network traffic and improving performance.
- **Joins:** Used to combine rows from two or more tables based on a related column between them (INNER, LEFT, RIGHT, FULL).

```sql
-- Inner Join Example
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- Stored Procedure Example (Oracle/MSSQL style)
CREATE PROCEDURE GetEmployeesByDept (@DeptId INT)
AS
BEGIN
    SELECT * FROM employees WHERE department_id = @DeptId;
END;
```

---

### Q5. [🏢] How do you integrate a Spring Boot backend with an Angular frontend?
**Answer:**
- **CORS Configuration:** The Spring Boot backend must enable Cross-Origin Resource Sharing (CORS) to accept requests from the Angular application (often running on a different port like `localhost:4200`).
- **Angular Services:** Create an Angular service using `HttpClient` to make asynchronous HTTP requests (GET, POST) to the Spring Boot REST endpoints.
- **RxJS Observables:** The `HttpClient` methods return Observables, which the Angular components subscribe to in order to receive the JSON data asynchronously and bind it to the UI.

---

### Q6. [🏢] How does authentication work in a Spring Boot application?
**Answer:**
- **Definition:** Authentication is the process of verifying who a user is (identity).
- **Mechanism (Spring Security):** When a user logs in, Spring Security's `AuthenticationManager` uses an `AuthenticationProvider` (typically using `UserDetailsService`) to fetch the user record from the database. 
- **Password Hashing:** It compares the hashed password provided by the user (using `BCryptPasswordEncoder`) with the stored hash.
- **JWT (Stateless):** In modern architectures, upon successful authentication, the server generates a JSON Web Token (JWT) and sends it to the client. The client includes this token in the `Authorization` header for subsequent requests, which the server validates using a custom filter (e.g., `JwtRequestFilter`).

---

### Q7. [🏢] What is the purpose of a Refresh Token?
**Answer:**
- **Problem:** Access tokens (like JWTs) should have a short lifespan (e.g., 15 minutes) for security reasons to mitigate token theft. However, requiring users to log in every 15 minutes is a poor user experience.
- **Solution:** A **Refresh Token** is issued alongside the access token. It has a much longer lifespan (e.g., 7 days or 30 days). 
- **Usage:** When the access token expires, the client sends the refresh token to a dedicated authentication endpoint to obtain a new access token without requiring the user's credentials again.
- **Security:** Refresh tokens are usually stored securely (e.g., in `httpOnly` cookies) and can be revoked by the server if suspicious activity is detected.

---

### Q8. [🏢] How does Dependency Injection work in Angular?
**Answer:**
- **Definition:** Dependency Injection (DI) is a design pattern where a class requests dependencies from external sources rather than creating them itself.
- **Providers & Injectors:** Angular has its own DI framework. You register a provider (typically a Service) in a module (`@NgModule`) or directly on the service using `@Injectable({ providedIn: 'root' })`.
- **Injection:** You inject the dependency into a component or another service by declaring it in the constructor. Angular's Injector looks up the provider and passes the instance to the component.
- **Singleton:** Services provided at the `root` level are singletons; the same instance is shared across the entire application.

---

### Q9. [🏢] How do parent and child components communicate in Angular?
**Answer:**
- **Parent to Child:** Using the `@Input()` decorator. The parent binds data to the child's input property in the template.
  - *Example:* `<app-child [data]="parentData"></app-child>`
- **Child to Parent:** Using the `@Output()` decorator with an `EventEmitter`. The child emits an event, and the parent listens for it.
  - *Example:* `<app-child (dataChange)="onDataChange($event)"></app-child>`
- **Parent to Child (ViewChild):** A parent can use `@ViewChild()` to get a reference to the child component instance and directly call its methods or access properties.
- **Shared Service:** Both components can inject a shared service (often using `BehaviorSubject`) to communicate if they are deeply nested or not directly related.

---

### Q10. [🏢] What is the difference between a shallow copy and a deep copy in Java?
**Answer:**
- **Shallow Copy:** Creates a new object, but inserts references into it to the objects found in the original. The cloned object and original object share references to the same mutable internal objects. Using `Object.clone()` without overriding usually results in a shallow copy.
- **Deep Copy:** Creates a new object and recursively copies all objects it references. The cloned object is completely independent of the original object. 
- **How to achieve Deep Copy:** By manually overriding the `clone()` method to clone nested objects, using serialization (writing to a byte stream and reading back), or using copy constructors.

---

### Q11. [🏢] What is the `UNION` operator in SQL?
**Answer:**
- **Purpose:** The `UNION` operator is used to combine the result sets of two or more `SELECT` statements into a single result set.
- **Rules:** 
  1. Every `SELECT` statement within `UNION` must have the same number of columns.
  2. The columns must have similar data types.
  3. The columns in every `SELECT` statement must be in the same order.
- **UNION vs UNION ALL:** `UNION` removes duplicate rows from the final result set, whereas `UNION ALL` includes all duplicates, making it faster since it skips the distinct sort operation.

---

### Q12. [🏢] How do you achieve authorization in Angular?
**Answer:**
- **Definition:** Authorization determines what an authenticated user is allowed to see or do (e.g., Admin vs User roles).
- **Route Guards:** Use Angular Route Guards (`CanActivate`, `CanMatch`) to prevent unauthorized users from navigating to specific routes based on their role.
- **Structural Directives:** Use `*ngIf` or create a custom structural directive (e.g., `*appHasRole="['ADMIN']"`) to hide or show UI elements (buttons, links) based on the user's permissions.
- **Interceptor:** While primarily for authentication, an HTTP Interceptor handles `403 Forbidden` responses from the backend, redirecting unauthorized API attempts to an error page or login screen.

---

### Q13. [🏢] What is the `...` (spread/rest) operator in JavaScript/TypeScript?
**Answer:**
- **Spread Operator:** Expands an iterable (like an array or object) into individual elements.
  - *Array Copy:* `let newArr = [...oldArr];` (creates a shallow copy).
  - *Object Merge:* `let merged = {...obj1, ...obj2};`.
- **Rest Parameter:** Collects multiple elements and condenses them into a single array element. Used in function parameters.
  - *Function definition:* `function sum(...numbers) { return numbers.reduce((a, b) => a + b); }`
- **Summary:** The spread operator "unpacks" elements, while the rest parameter "packs" elements into an array.
