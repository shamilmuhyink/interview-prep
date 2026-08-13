# Module 8: Design Patterns

> **Scope:** Creational, Structural, Behavioral Patterns, Anti-patterns, Real-world use cases, Integration with Spring/Java
> **Questions:** 20 | **Critical:** 5 | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

---

### Q1. 🔴 🌐 Explain the Singleton design pattern. How do you make a Singleton class thread-safe in Java?

**The Singleton pattern restricts the instantiation of a class to one single instance, typically implemented using a private constructor, a static variable, and a public static getter method; in Java, the most robust thread-safe implementations are the Bill Pugh (static inner class) approach or an Enum.**

**Core Concept:**
- Ensure only one instance of a class exists across the JVM (per classloader).
- Provide a global point of access to it.

**Internal Mechanics & Thread Safety:**
1. **Double-Checked Locking:** Checks if the instance is null, synchronizes on the class, and checks again. Requires the `volatile` keyword to prevent instruction reordering.
2. **Bill Pugh Singleton:** Uses a static inner helper class. The inner class is not loaded until `getInstance()` is called, making it thread-safe without explicit synchronization.
3. **Enum Singleton:** The absolute best way in Java. Provides implicit serialization support and prevents reflection attacks.

**💻 Production-quality code snippet:**
```java
// 1. Bill Pugh Singleton (Best for Lazy Initialization)
public class DatabaseConnection {
    private DatabaseConnection() {} // private constructor

    private static class SingletonHelper {
        private static final DatabaseConnection INSTANCE = new DatabaseConnection();
    }

    public static DatabaseConnection getInstance() {
        return SingletonHelper.INSTANCE;
    }
}

// 2. Enum Singleton (Best for overall safety)
public enum CacheManager {
    INSTANCE;
    
    public void put(String key, Object value) { /*...*/ }
    public Object get(String key) { /*...*/ }
}
```

**⚠️ Edge cases & common pitfalls:**
- **Reflection:** Can bypass private constructors (except in Enum).
- **Serialization:** Deserializing a Singleton creates a new instance unless `readResolve()` is implemented (Enum handles this automatically).
- **Multiple Classloaders:** In web containers, different classloaders might load multiple instances.

---

### Q2. 🔴 🌐 What is the Factory Design Pattern? Differentiate between Factory Method and Abstract Factory.

**The Factory pattern provides an interface for creating objects without specifying their exact classes, while Abstract Factory provides an interface for creating families of related or dependent objects without specifying their concrete classes.**

**Core Concept:**
- Promotes loose coupling by eliminating the need to bind application-specific classes into the code.
- OCP (Open/Closed Principle) compliant.

**Mechanics:**
- **Factory Method:** Uses inheritance. The creation logic is delegated to subclasses (e.g., `createProduct()` method).
- **Abstract Factory:** Uses object composition. It's a factory of factories.

**💻 Code Snippet (Factory Method):**
```java
public interface Notification { void notifyUser(); }
public class SMSNotification implements Notification { public void notifyUser() { /*...*/ } }
public class EmailNotification implements Notification { public void notifyUser() { /*...*/ } }

public class NotificationFactory {
    public static Notification createNotification(String channel) {
        return switch (channel) {
            case "SMS" -> new SMSNotification();
            case "EMAIL" -> new EmailNotification();
            default -> throw new IllegalArgumentException("Unknown channel");
        };
    }
}
```

**⚠️ Edge cases & common pitfalls:**
- Overusing factories can lead to unnecessary complexity and "class explosion".
- Switch statements in simple factories violate OCP if constantly updated (better to use reflection or a map of suppliers).

---

### Q3. 🔴 🏢 How does the Observer Pattern work? Give a real-world example in Java/Spring.

**The Observer pattern defines a one-to-many dependency between objects so that when one object (the subject) changes state, all its dependents (observers) are notified and updated automatically.**

**Core Concept:**
- Publisher-Subscriber model (push/pull mechanisms).
- Decouples the object that triggers the event from the objects that act on it.

**Real-World Examples:**
- Java: `java.util.Observable` (deprecated in Java 9 in favor of `java.beans.PropertyChangeListener` or Flow API).
- Spring: `ApplicationEventPublisher` and `@EventListener`.

**💻 Code snippet (Spring Application Events):**
```java
// 1. Event
public class OrderCreatedEvent extends ApplicationEvent {
    private final String orderId;
    public OrderCreatedEvent(Object source, String orderId) { super(source); this.orderId = orderId; }
}

// 2. Publisher
@Service
public class OrderService {
    private final ApplicationEventPublisher publisher;
    
    public void createOrder() {
        // order logic...
        publisher.publishEvent(new OrderCreatedEvent(this, "ORD-123"));
    }
}

// 3. Observer (Listener)
@Component
public class EmailNotificationListener {
    @EventListener
    public void handleOrderCreatedEvent(OrderCreatedEvent event) {
        System.out.println("Sending email for order: " + event.getOrderId());
    }
}
```

**⚠️ Edge cases & common pitfalls:**
- **Memory Leaks (Lapsed Listener Problem):** If observers are not deregistered, the subject holds a strong reference to them, preventing garbage collection.
- **Synchronous Blocking:** By default, Spring `@EventListener` is synchronous. If one listener is slow, it blocks the publisher. (Use `@Async` to solve this).

---

### Q4. 🔴 🏢 Explain the Strategy Pattern. How is it implemented using Java 8 functional interfaces?

**The Strategy pattern enables selecting an algorithm's behavior at runtime by defining a family of algorithms, encapsulating each one, and making them interchangeable without altering the client code.**

**Core Concept:**
- Eliminates large conditional statements (`if-else` or `switch`).
- Implements the Open/Closed Principle.

**💻 Code snippet (Java 8+ style):**
```java
import java.math.BigDecimal;
import java.util.function.Function;

public class PaymentProcessor {
    // Strategy defined as a functional interface
    public void processPayment(BigDecimal amount, Function<BigDecimal, Boolean> paymentStrategy) {
        boolean success = paymentStrategy.apply(amount);
        if (success) {
            System.out.println("Payment successful!");
        }
    }
}

// Client Code
public class Main {
    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();
        
        Function<BigDecimal, Boolean> creditCardStrategy = amt -> { /* CC logic */ return true; };
        Function<BigDecimal, Boolean> upiStrategy = amt -> { /* UPI logic */ return true; };
        
        processor.processPayment(new BigDecimal("100.00"), upiStrategy);
    }
}
```

**⚠️ Edge cases & common pitfalls:**
- If the number of strategies is small and unlikely to change, `if-else` might be simpler.
- Clients must understand how strategies differ to select the appropriate one.

---

### Q5. 🔴 🌐 What is the Decorator Pattern? How is it used in Java I/O?

**The Decorator pattern allows behavior to be added to an individual object, dynamically or statically, without affecting the behavior of other objects from the same class by wrapping the original object in a decorator class.**

**Core Concept:**
- Favors composition over inheritance.
- Provides a flexible alternative to subclassing for extending functionality.

**Real-world Example (Java I/O):**
The `java.io` package heavily uses Decorators. For example, `BufferedReader` decorates a `FileReader` to add buffering capabilities.

**💻 Code snippet:**
```java
// Base Component
interface Coffee { double getCost(); String getDescription(); }

class SimpleCoffee implements Coffee {
    public double getCost() { return 2.0; }
    public String getDescription() { return "Simple Coffee"; }
}

// Decorator
abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;
    public CoffeeDecorator(Coffee coffee) { this.decoratedCoffee = coffee; }
    public double getCost() { return decoratedCoffee.getCost(); }
    public String getDescription() { return decoratedCoffee.getDescription(); }
}

// Concrete Decorator
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) { super(coffee); }
    public double getCost() { return super.getCost() + 0.5; }
    public String getDescription() { return super.getDescription() + ", Milk"; }
}

// Usage
Coffee coffee = new MilkDecorator(new SimpleCoffee());
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
```

**⚠️ Edge cases & common pitfalls:**
- Can result in a system with lots of small objects that look similar, making debugging difficult (tracing through multiple wrappers).
- Order of decoration can matter and cause subtle bugs if not managed properly.

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

*(Placeholder for 6-12)*
6. Explain the Builder pattern and when to use it over constructor chaining.
7. What is the Proxy pattern? How does Spring AOP use it?
8. Explain the Facade pattern. How is it different from an API Gateway?
9. What is the Adapter pattern? Provide an example of its use in Java.
10. Describe the Command pattern and its use cases (e.g., undo/redo, CQRS).
11. What is the Template Method pattern? Contrast it with the Strategy pattern.
12. Explain the Chain of Responsibility pattern. How are Servlet Filters an example of this?

---

## 🟢 GOOD TO KNOW (Questions 13–20)

*(Placeholder for 13-20)*
13. Describe the State design pattern.
14. What is the Mediator pattern? When would you use it?
15. Explain the Flyweight pattern and its relationship with the Java String Pool.
16. What is the Prototype pattern? Explain deep vs shallow copying in Java.
17. Describe the Iterator pattern and how it's implemented in Java Collections.
18. What is the Visitor pattern? Why is it less common in modern Java?
19. What is the Memento pattern?
20. What is an Anti-pattern? Give examples like God Object, Magic Numbers, and Poltergeists.

---

> 💡 **Pro Tip:** In modern Java (especially Spring Boot applications), many structural and creational patterns are handled by the framework (Dependency Injection abstracts away Factories, AOP handles Proxies). Be prepared to discuss how frameworks implement these patterns behind the scenes!
