# ☕ Java Developer Roadmap — Complete Beginner to Advanced

> **Source:** [roadmap.sh/java](https://roadmap.sh/java)
> **Target:** Complete Beginners to Advanced Developers
> **Java Version:** 21+ (LTS) | **Last Updated:** September 2026
> **Structure:** Progressive learning path — each section builds on the previous one

## 🧭 How to Use This Guide

> 💡 **Beginner Tip:** Don't try to learn everything at once! 
> 
> - **If you are a complete beginner:** Start at Phase 0 and work your way through Phase 1 and 2. **Practice writing code** for every concept you read about. Do not move on to Phase 3 until you are comfortable building small applications on your own.
> - **If you are preparing for interviews:** Pay attention to the 🔴/🟡/🟢 tags which indicate how frequently a topic is asked by major companies.

## 0. 🚀 Phase 0: Getting Started

Before writing any code, you need to set up your environment.

1. **Install the JDK (Java Development Kit):** This contains the tools needed to write and run Java programs. (Recommended: Download Eclipse Temurin JDK 21+).
2. **Choose an IDE (Integrated Development Environment):** An IDE is a smart text editor for writing code. (Recommended: IntelliJ IDEA Community Edition or VS Code with the Java Extension Pack).
3. **Write Your First Program:**
   ```java
   public class HelloWorld {
       public static void main(String[] args) {
           System.out.println("Hello, World!");
       }
   }
   ```

---

## 📋 Table of Contents

| # | Section | Level | Topics |
|---|---------|-------|--------|
| 1 | [Fundamentals](#1--fundamentals) | 🟢 Beginner | Syntax, Data Types, Variables, Operators, Control Flow |
| 2 | [Program Structure](#2--program-structure) | 🟢 Beginner | Classes, Objects, Methods, Access Modifiers, Packages |
| 3 | [Object-Oriented Programming](#3--object-oriented-programming-oop) | 🟢 Beginner | 4 Pillars, Interfaces, Abstract Classes, Enums, Records |
| 4 | [Strings & Math](#4--strings--math-operations) | 🟢 Beginner | String Pool, StringBuilder, Math API |
| 5 | [Exception Handling](#5--exception-handling) | 🟡 Intermediate | Checked/Unchecked, Custom Exceptions, Try-with-Resources |
| 6 | [Data Structures & Collections](#6--data-structures--collections-framework) | 🟡 Intermediate | List, Set, Map, Queue, Deque, Iterator |
| 7 | [Generics](#7--generics) | 🟡 Intermediate | Type Parameters, Bounded Types, Wildcards |
| 8 | [Functional Programming](#8--functional-programming) | 🟡 Intermediate | Lambdas, Functional Interfaces, Stream API, Optionals |
| 9 | [I/O Operations](#9--io-operations) | 🟡 Intermediate | File I/O, Streams, NIO, Serialization |
| 10 | [Concurrency & Multithreading](#10--concurrency--multithreading) | 🔴 Advanced | Threads, Executors, Virtual Threads, Locks, JMM |
| 11 | [JVM Internals](#11--jvm-internals) | 🔴 Advanced | Memory Model, GC, ClassLoaders, JIT |
| 12 | [Modern Java Features (8–21+)](#12--modern-java-features-821) | 🔴 Advanced | Records, Sealed Classes, Pattern Matching, Virtual Threads |
| 13 | [Date & Time API](#13--date--time-api) | 🟡 Intermediate | java.time, Formatting, Zones |
| 14 | [Networking & Regex](#14--networking--regular-expressions) | 🟡 Intermediate | Sockets, HTTP Client, Pattern Matching |
| 15 | [Reflection & Annotations](#15--reflection--annotations) | 🔴 Advanced | Dynamic Class Inspection, Custom Annotations |
| 16 | [Build Tools](#16--build-tools--dependency-management) | 🟡 Intermediate | Maven, Gradle |
| 17 | [Databases & JDBC](#17--databases--jdbc) | 🟡 Intermediate | JDBC, Connection Pooling, Transactions |
| 18 | [ORM (JPA & Hibernate)](#18--orm-jpa--hibernate) | 🔴 Advanced | Entity Mapping, Caching, N+1 Problem |
| 19 | [Spring Framework & Spring Boot](#19--spring-framework--spring-boot) | 🔴 Advanced | IoC/DI, REST, Security, Actuator |
| 20 | [Testing](#20--testing) | 🟡 Intermediate | JUnit 5, Mockito, Integration Testing |
| 21 | [Logging](#21--logging) | 🟢 Beginner | SLF4J, Logback, Log4j2 |
| 22 | [Design Patterns](#22--design-patterns) | 🔴 Advanced | Creational, Structural, Behavioral |
| 23 | [Microservices Architecture](#23--microservices-architecture) | 🔴 Advanced | Service Discovery, API Gateway, Resilience |
| 24 | [DevOps Essentials](#24--devops-essentials) | 🔴 Advanced | Docker, Kubernetes, CI/CD, AWS |

---

## 1. 🟢 Fundamentals

> **Goal:** Understand how Java works, write your first programs, and master basic syntax.

### 1.1 What is Java?

Java is a popular programming language used for everything from mobile apps to large enterprise backend systems. 

> 💡 **Beginner Tip:** Java is "Write Once, Run Anywhere". When you write Java code, a tool called a compiler turns it into "bytecode". This bytecode is then run by the **Java Virtual Machine (JVM)**. Because the JVM exists for Windows, Mac, and Linux, your single piece of code can run anywhere!

```
┌──────────────┐    ┌──────────┐    ┌──────────┐    ┌──────────────┐
│  .java file  │───▶│  javac   │───▶│ .class   │───▶│     JVM      │
│ (source code)│    │(compiler)│    │(bytecode)│    │(runs on any  │
│              │    │          │    │          │    │  platform)   │
└──────────────┘    └──────────┘    └──────────┘    └──────────────┘
```

### 1.2 Data Types

Java has **two categories** of data types:

| Category | Types | Examples | Size |
|----------|-------|---------|------|
| **Primitive** | `byte` | `-128` to `127` | 1 byte |
| | `short` | `-32,768` to `32,767` | 2 bytes |
| | `int` | `-2^31` to `2^31 - 1` | 4 bytes |
| | `long` | `-2^63` to `2^63 - 1` | 8 bytes |
| | `float` | `3.14f` | 4 bytes |
| | `double` | `3.14159` | 8 bytes |
| | `char` | `'A'`, `'\u0041'` | 2 bytes |
| | `boolean` | `true`, `false` | ~1 bit |
| **Reference** | Objects, Arrays, Strings, Interfaces | `String name = "Java"` | Varies |

```java
// Primitive types
int age = 25;
double salary = 85000.50;
char grade = 'A';
boolean isJavaDeveloper = true;
long population = 8_000_000_000L;  // Underscores for readability (Java 7+)

// Reference types
String name = "Java Developer";
int[] scores = {90, 85, 92, 88};
```

### 1.3 Variables & Scopes

```java
public class VariableScopes {
    // Instance variable — belongs to each object
    private String instanceVar = "I belong to the object";
    
    // Static/Class variable — shared across ALL objects
    private static int classVar = 0;
    
    public void method() {
        // Local variable — exists only inside this method
        int localVar = 42;
        
        // Block-scoped variable
        if (localVar > 0) {
            String blockVar = "I exist only in this if-block";
        }
        // blockVar is NOT accessible here
    }
}
```

> ⚠️ **Interview Pitfall:** Local variables must be initialized before use. Instance/class variables get default values (`0`, `null`, `false`).

### 1.4 Operators

| Category | Operators | Example |
|----------|-----------|---------|
| Arithmetic | `+`, `-`, `*`, `/`, `%` | `10 % 3` → `1` |
| Relational | `==`, `!=`, `<`, `>`, `<=`, `>=` | `5 > 3` → `true` |
| Logical | `&&`, `\|\|`, `!` | `true && false` → `false` |
| Bitwise | `&`, `\|`, `^`, `~`, `<<`, `>>`, `>>>` | `5 & 3` → `1` |
| Assignment | `=`, `+=`, `-=`, `*=`, `/=` | `x += 5` |
| Ternary | `? :` | `age >= 18 ? "Adult" : "Minor"` |
| `instanceof` | `instanceof` | `obj instanceof String` |

### 1.5 Control Flow

```java
// If-else
if (score >= 90) {
    grade = 'A';
} else if (score >= 80) {
    grade = 'B';
} else {
    grade = 'C';
}

// Enhanced switch expression (Java 14+)
String result = switch (grade) {
    case 'A' -> "Excellent";
    case 'B' -> "Good";
    case 'C' -> "Average";
    default -> "Unknown";
};

// For loop
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}

// Enhanced for-each loop
for (String item : items) {
    System.out.println(item);
}

// While loop
while (condition) {
    // body
}

// Do-while loop (executes at least once)
do {
    // body
} while (condition);
```

### 1.6 Type Casting

```java
// Widening (implicit) — no data loss
int num = 100;
long bigNum = num;        // int → long (automatic)
double decimal = bigNum;  // long → double (automatic)

// Narrowing (explicit) — potential data loss
double pi = 3.14159;
int truncated = (int) pi;  // 3 (decimal part lost!)

// String conversions
String str = String.valueOf(42);        // int → String
int parsed = Integer.parseInt("42");    // String → int
double parsedD = Double.parseDouble("3.14"); // String → double
```

---

## 2. 🟢 Program Structure

> **Goal:** Understand how Java programs are organized — classes, objects, methods, and packages.

### 2.1 Lifecycle of a Java Program

```
┌─────────────────────────────────────────────────────────────────────────┐
│  1. WRITE (.java)  →  2. COMPILE (javac)  →  3. LOAD (ClassLoader)    │
│  →  4. VERIFY (Bytecode Verifier)  →  5. EXECUTE (JVM / JIT Compiler) │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Classes & Objects

```java
// Class — the blueprint
public class Employee {
    // Fields (attributes)
    private String name;
    private int age;
    private double salary;
    
    // Constructor
    public Employee(String name, int age, double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
    
    // Method (behavior)
    public double calculateAnnualSalary() {
        return salary * 12;
    }
    
    // Getter
    public String getName() {
        return name;
    }
    
    // Setter with validation
    public void setSalary(double salary) {
        if (salary < 0) throw new IllegalArgumentException("Salary cannot be negative");
        this.salary = salary;
    }
    
    @Override
    public String toString() {
        return "Employee{name='%s', age=%d, salary=%.2f}".formatted(name, age, salary);
    }
}

// Object — an instance of the class
Employee emp = new Employee("Shamil", 28, 95000.0);
System.out.println(emp.calculateAnnualSalary()); // 1140000.0
```

### 2.3 Access Modifiers

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| *(default/package-private)* | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

### 2.4 Static Keyword

```java
public class MathUtils {
    // Static field — shared by ALL instances
    private static int instanceCount = 0;
    
    // Static method — called on the CLASS, not an object
    public static int add(int a, int b) {
        return a + b;
    }
    
    // Static block — runs once when the class is first loaded
    static {
        System.out.println("MathUtils class loaded!");
    }
    
    // Static inner class
    public static class Constants {
        public static final double PI = 3.14159265358979;
    }
}

// Usage — no object needed
int sum = MathUtils.add(5, 3);
double pi = MathUtils.Constants.PI;
```

### 2.5 Nested Classes

```java
public class OuterClass {
    private String outerField = "Outer";
    
    // 1. Non-static inner class (has reference to outer)
    class InnerClass {
        void display() {
            System.out.println(outerField); // Can access outer's private fields
        }
    }
    
    // 2. Static nested class (no reference to outer)
    static class StaticNestedClass {
        void display() {
            // Cannot access outerField directly
            System.out.println("Static nested class");
        }
    }
    
    // 3. Local class (inside a method)
    void method() {
        class LocalClass {
            void display() { System.out.println("Local class"); }
        }
        new LocalClass().display();
    }
    
    // 4. Anonymous class
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("Anonymous class running");
        }
    };
}
```

### 2.6 Packages

```java
// Declaring a package
package com.company.project.service;

// Importing classes
import java.util.List;
import java.util.ArrayList;
import java.util.stream.*;  // Wildcard import (imports all classes in the package)

// Static import — access static members directly
import static java.lang.Math.PI;
import static java.lang.Math.sqrt;

double circumference = 2 * PI * radius;  // No Math.PI needed
```

---

## 3. 🟢 Object-Oriented Programming (OOP)

> **Goal:** Understand how to organize code into real-world concepts (objects). Master the 4 pillars of OOP.
> 
> 💡 **Beginner Tip:** Think of OOP as building with Lego blocks. Instead of writing one long script, you create small, reusable "objects" (like a `User` or a `ShoppingCart`) that interact with each other. This is how almost all modern software is built.
> *(🔴 Note for interviews: This is the #1 most frequently asked topic).*

### 3.1 The Four Pillars

```
┌───────────────────────────────────────────────────────────────────┐
│                    Object-Oriented Programming                    │
├────────────────┬───────────────┬───────────────┬─────────────────┤
│  ENCAPSULATION │  INHERITANCE  │ POLYMORPHISM  │  ABSTRACTION    │
│  Data hiding   │  Code reuse   │ Many forms    │  Hide details   │
│  via access    │  via extends  │ via overload/ │  via abstract   │
│  modifiers     │  / implements │ override      │  class/interface│
└────────────────┴───────────────┴───────────────┴─────────────────┘
```

### 3.2 Encapsulation

```java
public class BankAccount {
    // Private fields — data is hidden
    private String accountNumber;
    private double balance;
    
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }
    
    // Controlled access through methods
    public void deposit(double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Deposit must be positive");
        this.balance += amount;
    }
    
    public void withdraw(double amount) {
        if (amount > balance) throw new InsufficientFundsException("Insufficient funds");
        this.balance -= amount;
    }
    
    // Read-only access to balance
    public double getBalance() {
        return balance;
    }
}
```

### 3.3 Inheritance

```java
// Parent class
public class Animal {
    protected String name;
    protected int age;
    
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public void makeSound() {
        System.out.println("Some generic sound");
    }
}

// Child class — inherits from Animal
public class Dog extends Animal {
    private String breed;
    
    public Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }
    
    @Override  // Method overriding — runtime polymorphism
    public void makeSound() {
        System.out.println(name + " says: Woof!");
    }
    
    // Dog-specific method
    public void fetch() {
        System.out.println(name + " is fetching the ball!");
    }
}
```

> ⚠️ **Interview Pitfall:** Java does NOT support multiple inheritance of classes (only interfaces). This avoids the **Diamond Problem**.

### 3.4 Polymorphism

```java
// Compile-time polymorphism (Method Overloading)
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c) { return a + b + c; }
}

// Runtime polymorphism (Method Overriding)
Animal animal = new Dog("Buddy", 3, "Labrador");
animal.makeSound();  // Output: "Buddy says: Woof!" — resolved at RUNTIME
```

📊 **Overloading vs Overriding:**

| Feature | Overloading | Overriding |
|---------|------------|------------|
| When resolved | Compile-time | Runtime |
| Methods | Same name, different params | Same name, same params |
| Classes | Same class | Parent-Child |
| Return type | Can differ | Must be same or covariant |
| Access modifier | Can differ | Cannot be more restrictive |
| `static` methods | Can be overloaded | Cannot be overridden (hidden) |

### 3.5 Abstraction

```java
// Abstract class — partial abstraction
public abstract class Shape {
    protected String color;
    
    public Shape(String color) {
        this.color = color;
    }
    
    // Abstract method — must be implemented by subclasses
    public abstract double area();
    public abstract double perimeter();
    
    // Concrete method — shared implementation
    public void displayInfo() {
        System.out.printf("Shape: %s, Color: %s, Area: %.2f%n",
            getClass().getSimpleName(), color, area());
    }
}

// Interface — complete abstraction (Java 8+ can have default methods)
public interface Drawable {
    void draw();  // abstract by default
    
    default void resize(double factor) {  // Default method (Java 8+)
        System.out.println("Resizing by factor: " + factor);
    }
    
    static Drawable createCircle() {  // Static method in interface
        return () -> System.out.println("Drawing circle");
    }
}

// Concrete class implementing both
public class Circle extends Shape implements Drawable {
    private double radius;
    
    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }
    
    @Override
    public double area() { return Math.PI * radius * radius; }
    
    @Override
    public double perimeter() { return 2 * Math.PI * radius; }
    
    @Override
    public void draw() { System.out.println("Drawing circle with radius " + radius); }
}
```

📊 **Abstract Class vs Interface:**

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Instantiation | ❌ Cannot | ❌ Cannot |
| Constructors | ✅ Yes | ❌ No |
| State (fields) | ✅ Instance variables | Only `static final` constants |
| Method types | Abstract + Concrete | Abstract + `default` + `static` + `private` (9+) |
| Inheritance | Single (`extends`) | Multiple (`implements`) |
| When to use | IS-A relationship with shared state | Capability/contract definition |

### 3.6 Enums

```java
public enum OrderStatus {
    PENDING("Order placed"),
    PROCESSING("Being prepared"),
    SHIPPED("On the way"),
    DELIVERED("Delivered successfully"),
    CANCELLED("Order cancelled");
    
    private final String description;
    
    OrderStatus(String description) {
        this.description = description;
    }
    
    public String getDescription() {
        return description;
    }
    
    // Enums can have methods
    public boolean isFinalState() {
        return this == DELIVERED || this == CANCELLED;
    }
}

// Usage
OrderStatus status = OrderStatus.SHIPPED;
System.out.println(status.getDescription());  // "On the way"
System.out.println(status.isFinalState());    // false
```

### 3.7 Records (Java 16+)

```java
// Records — immutable data carriers (replaces boilerplate POJOs)
public record Employee(String name, int age, double salary) {
    // Compact constructor for validation
    public Employee {
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
        if (salary < 0) throw new IllegalArgumentException("Salary cannot be negative");
    }
    
    // Custom method
    public double annualSalary() {
        return salary * 12;
    }
}

// Usage — auto-generates constructor, getters, equals(), hashCode(), toString()
Employee emp = new Employee("Shamil", 28, 95000);
System.out.println(emp.name());     // "Shamil" (accessor, not getName())
System.out.println(emp);            // Employee[name=Shamil, age=28, salary=95000.0]
```

### 3.8 Method Chaining (Fluent API)

```java
public class QueryBuilder {
    private String table;
    private String whereClause;
    private String orderBy;
    private int limit;
    
    public QueryBuilder from(String table) {
        this.table = table;
        return this;  // Return 'this' to enable chaining
    }
    
    public QueryBuilder where(String condition) {
        this.whereClause = condition;
        return this;
    }
    
    public QueryBuilder orderBy(String column) {
        this.orderBy = column;
        return this;
    }
    
    public QueryBuilder limit(int limit) {
        this.limit = limit;
        return this;
    }
    
    public String build() {
        return "SELECT * FROM %s WHERE %s ORDER BY %s LIMIT %d"
            .formatted(table, whereClause, orderBy, limit);
    }
}

// Usage
String query = new QueryBuilder()
    .from("employees")
    .where("salary > 50000")
    .orderBy("name")
    .limit(10)
    .build();
```

---

## 4. 🟢 Strings & Math Operations

### 4.1 String Internals

> 🔴 **CRITICAL — String vs StringBuilder vs StringBuffer asked by 4/9 companies**

```
┌─────────────────────────────────────────────────────────────────┐
│                        String Pool (Heap)                       │
│  ┌─────────┐  ┌─────────┐  ┌──────────┐                       │
│  │ "Hello" │  │ "World" │  │ "Java"   │                       │
│  └─────────┘  └─────────┘  └──────────┘                       │
│                                                                 │
│  String s1 = "Hello";  // Points to pool                       │
│  String s2 = "Hello";  // Points to SAME object in pool        │
│  String s3 = new String("Hello"); // NEW object on heap        │
│                                                                 │
│  s1 == s2     → true  (same reference in pool)                 │
│  s1 == s3     → false (different objects)                      │
│  s1.equals(s3)→ true  (same content)                           │
└─────────────────────────────────────────────────────────────────┘
```

📊 **String vs StringBuilder vs StringBuffer:**

| Feature | String | StringBuilder | StringBuffer |
|---------|--------|--------------|--------------|
| Mutability | **Immutable** | **Mutable** | **Mutable** |
| Thread-safe | ✅ (immutable) | ❌ Not thread-safe | ✅ Synchronized |
| Performance | Slow for concatenation | **Fastest** | Slower than StringBuilder |
| When to use | Constants, few modifications | Single-threaded string building | Multi-threaded string building |

```java
// ❌ Bad — creates many intermediate String objects
String result = "";
for (int i = 0; i < 10000; i++) {
    result += i;  // Creates a NEW String each iteration!
}

// ✅ Good — mutable, efficient
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

### 4.2 Important String Methods

```java
String s = "Hello, World!";

s.length();              // 13
s.charAt(0);             // 'H'
s.substring(0, 5);       // "Hello"
s.indexOf("World");      // 7
s.contains("World");     // true
s.startsWith("Hello");   // true
s.toUpperCase();         // "HELLO, WORLD!"
s.trim();                // Removes leading/trailing whitespace
s.strip();               // Better than trim() — handles Unicode whitespace (Java 11+)
s.replace("World", "Java"); // "Hello, Java!"
s.split(", ");           // ["Hello", "World!"]
s.isBlank();             // false (Java 11+)
s.repeat(3);             // "Hello, World!Hello, World!Hello, World!" (Java 11+)
s.chars()                // IntStream of characters (Java 8+)

// Text blocks (Java 15+)
String json = """
    {
        "name": "Shamil",
        "role": "Java Developer"
    }
    """;

// String formatting
String formatted = "Hello, %s! You are %d years old.".formatted("Shamil", 28);
```

---

## 5. 🟡 Exception Handling

> 🔴 **CRITICAL — Checked vs Unchecked asked by 4/9 companies**

### 5.1 Exception Hierarchy

```
                    java.lang.Throwable
                    /                 \
           java.lang.Error      java.lang.Exception
           (Don't catch!)        /                \
          /       \       RuntimeException    Checked Exceptions
    OutOfMemory  StackOverflow  (Unchecked)    (Must handle)
                              /     |    \        /      \
                   NullPointer  ClassCast  ArrayIndex  IOException  SQLException
                   Exception    Exception  OutOfBounds
```

📊 **Checked vs Unchecked Exceptions:**

| Feature | Checked Exception | Unchecked Exception |
|---------|-------------------|---------------------|
| Extends | `Exception` | `RuntimeException` |
| Compile-time check | ✅ Must handle or declare | ❌ No requirement |
| Examples | `IOException`, `SQLException` | `NullPointerException`, `ClassCastException` |
| When to use | Recoverable conditions | Programming errors |

### 5.2 Exception Handling Patterns

```java
// Basic try-catch-finally
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Cannot divide by zero: " + e.getMessage());
} finally {
    System.out.println("Always executes — cleanup here");
}

// Multi-catch (Java 7+)
try {
    // risky code
} catch (IOException | SQLException e) {
    logger.error("I/O or DB error: {}", e.getMessage());
}

// Try-with-resources (Java 7+) — auto-closes resources
try (var reader = new BufferedReader(new FileReader("data.txt"));
     var writer = new BufferedWriter(new FileWriter("output.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        writer.write(line.toUpperCase());
        writer.newLine();
    }
} // reader and writer are automatically closed here

// Custom exception
public class InsufficientFundsException extends RuntimeException {
    private final double deficit;
    
    public InsufficientFundsException(String message, double deficit) {
        super(message);
        this.deficit = deficit;
    }
    
    public double getDeficit() { return deficit; }
}
```

> ⚠️ **Interview Pitfall:** `throw` vs `throws`
> - `throw` — used to **actually throw** an exception object: `throw new RuntimeException("error")`
> - `throws` — used in method signature to **declare** checked exceptions: `void read() throws IOException`

> ⚠️ **Interview Pitfall:** `final` vs `finally` vs `finalize`
> - `final` — keyword to make variables constant, methods un-overridable, classes un-inheritable
> - `finally` — block that always executes after try-catch (cleanup code)
> - `finalize` — **deprecated** method called by GC before object destruction (don't use!)

---

## 6. 🟡 Data Structures & Collections Framework

> 🔴 **CRITICAL — HashMap internals asked by 9/9 companies. Collections is the #1 topic.**

### 6.1 Collections Hierarchy

```
                         Iterable
                            │
                        Collection
                     /      |       \
                  List     Set      Queue
                /   |      / \       |  \
          ArrayList  LinkedList  HashSet  TreeSet  PriorityQueue  Deque
                                    |                              |
                             LinkedHashSet                    ArrayDeque
                             
                          Map (separate hierarchy)
                        /    |         \
                  HashMap  TreeMap   LinkedHashMap
                     |
              ConcurrentHashMap
```

### 6.2 List Implementations

📊 **ArrayList vs LinkedList:**

| Feature | ArrayList | LinkedList |
|---------|-----------|------------|
| Underlying structure | Dynamic array | Doubly linked list |
| Random access `get(i)` | **O(1)** ✅ | O(n) ❌ |
| Add/Remove at end | **O(1) amortized** | **O(1)** |
| Add/Remove at middle | O(n) — shifting | **O(1)** — pointer change |
| Memory | Less (contiguous) | More (node + pointers) |
| Best for | Read-heavy, random access | Insert/delete-heavy |

```java
// ArrayList — most commonly used
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.add(1, "Charlie");  // Insert at index 1
names.remove("Bob");
names.get(0);  // O(1) access

// LinkedList — good for frequent insertions/deletions
List<String> linkedNames = new LinkedList<>();
linkedNames.addFirst("First");
linkedNames.addLast("Last");

// Immutable List (Java 9+)
List<String> immutable = List.of("A", "B", "C");  // Cannot modify!
```

### 6.3 Set Implementations

```java
// HashSet — no duplicates, no order guarantee, O(1) add/remove/contains
Set<String> hashSet = new HashSet<>();
hashSet.add("Java");
hashSet.add("Python");
hashSet.add("Java");  // Ignored — duplicate!
System.out.println(hashSet.size());  // 2

// LinkedHashSet — maintains insertion order
Set<String> linkedSet = new LinkedHashSet<>();

// TreeSet — sorted order (natural ordering or Comparator)
Set<String> treeSet = new TreeSet<>();
treeSet.add("Banana");
treeSet.add("Apple");
treeSet.add("Cherry");
System.out.println(treeSet);  // [Apple, Banana, Cherry]
```

### 6.4 Map Implementations — HashMap Internals

> 🔴 **#1 Most Asked Topic — 9/9 companies ask about HashMap internals**

```
HashMap Internal Structure (Java 8+):
┌─────────────────────────────────────────────────────────┐
│  Array of Buckets (Node<K,V>[])                         │
│  Index = hash(key) & (capacity - 1)                     │
│                                                         │
│  [0] → null                                             │
│  [1] → Node(K1,V1) → Node(K2,V2) → null  (LinkedList)  │
│  [2] → null                                             │
│  [3] → TreeNode(K3,V3)                   (Red-Black     │
│         /          \                      Tree when      │
│    TreeNode(K4,V4)  TreeNode(K5,V5)       bucket size    │
│                                           >= 8)          │
│  [4] → Node(K6,V6) → null                              │
│  ...                                                    │
│  [15] → null                                            │
│                                                         │
│  Default capacity: 16 | Load factor: 0.75               │
│  Threshold = capacity × load factor = 12                │
│  When size > threshold → resize (double capacity)       │
│  Treeification threshold: bucket size >= 8              │
│  Untreeification threshold: bucket size <= 6            │
└─────────────────────────────────────────────────────────┘
```

```java
// HashMap — O(1) average for put/get/remove
Map<String, Integer> map = new HashMap<>();
map.put("Java", 1995);
map.put("Python", 1991);
map.getOrDefault("Go", 0);  // Returns 0 if key not found
map.putIfAbsent("Java", 2000);  // Won't overwrite existing
map.computeIfAbsent("Kotlin", k -> 2011);
map.merge("Java", 1, Integer::sum);  // Merge values

// Iterating
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " → " + entry.getValue());
}
map.forEach((key, value) -> System.out.println(key + ": " + value));
```

📊 **HashMap vs ConcurrentHashMap:**

| Feature | HashMap | ConcurrentHashMap |
|---------|---------|-------------------|
| Thread-safe | ❌ No | ✅ Yes |
| Null keys | ✅ One null key | ❌ No null keys |
| Null values | ✅ Multiple | ❌ No null values |
| Locking | None | Segment/bucket-level locking |
| Performance (single-thread) | Faster | Slightly slower |
| Performance (multi-thread) | Unsafe/crashes | **Much better** than `Collections.synchronizedMap()` |
| Iterators | Fail-fast | Weakly consistent |

### 6.5 Queue & Deque

```java
// PriorityQueue — elements ordered by natural ordering or Comparator
Queue<Integer> pq = new PriorityQueue<>();  // Min-heap by default
pq.offer(30);
pq.offer(10);
pq.offer(20);
pq.poll();  // 10 (smallest first)

// Max-heap
Queue<Integer> maxPq = new PriorityQueue<>(Comparator.reverseOrder());

// ArrayDeque — double-ended queue (faster than Stack and LinkedList for stack/queue ops)
Deque<String> deque = new ArrayDeque<>();
deque.push("First");    // Stack: push to front
deque.push("Second");
deque.pop();            // Stack: pop from front → "Second"
deque.offerLast("End"); // Queue: add to end
deque.pollFirst();      // Queue: remove from front
```

### 6.6 Comparable vs Comparator

```java
// Comparable — natural ordering (implemented in the class itself)
public class Employee implements Comparable<Employee> {
    private String name;
    private double salary;
    
    @Override
    public int compareTo(Employee other) {
        return Double.compare(this.salary, other.salary);
    }
}
Collections.sort(employees);  // Uses compareTo()

// Comparator — external ordering strategy
Comparator<Employee> byName = Comparator.comparing(Employee::getName);
Comparator<Employee> bySalaryDesc = Comparator.comparing(Employee::getSalary).reversed();
Comparator<Employee> byNameThenSalary = byName.thenComparing(bySalaryDesc);

employees.sort(byNameThenSalary);
```

---

## 7. 🟡 Generics

> **Goal:** Write type-safe, reusable code that works with any object type.

### 7.1 Generic Classes & Methods

```java
// Generic class
public class Pair<K, V> {
    private K key;
    private V value;
    
    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    
    public K getKey() { return key; }
    public V getValue() { return value; }
}

Pair<String, Integer> pair = new Pair<>("Age", 28);

// Generic method
public static <T extends Comparable<T>> T findMax(List<T> list) {
    return list.stream().max(Comparator.naturalOrder()).orElseThrow();
}
```

### 7.2 Bounded Types & Wildcards

```java
// Upper bounded — accepts Number or any subclass
public static double sum(List<? extends Number> numbers) {
    return numbers.stream().mapToDouble(Number::doubleValue).sum();
}

// Lower bounded — accepts Integer or any superclass
public static void addIntegers(List<? super Integer> list) {
    list.add(1);
    list.add(2);
}

// PECS Rule: Producer Extends, Consumer Super
// ? extends T → READ from it (produces values)
// ? super T   → WRITE to it (consumes values)
```

### 7.3 Type Erasure

```java
// At compile time:
List<String> strings = new ArrayList<>();
List<Integer> integers = new ArrayList<>();

// After type erasure (at runtime), both become:
// List strings = new ArrayList();
// List integers = new ArrayList();

// This is why you can't do:
// if (obj instanceof List<String>) { }  // ❌ Compile error
// new T();                              // ❌ Compile error
```

---

## 8. 🟡 Functional Programming

> 🔴 **CRITICAL — Java 8 Streams API asked by 8/9 companies**

### 8.1 Functional Interfaces

```java
// A functional interface has exactly ONE abstract method
@FunctionalInterface
public interface Transformer<T, R> {
    R transform(T input);
}

// Built-in functional interfaces (java.util.function)
```

| Interface | Method | Input → Output | Example |
|-----------|--------|---------------|---------|
| `Predicate<T>` | `test(T)` | `T → boolean` | `s -> s.isEmpty()` |
| `Function<T,R>` | `apply(T)` | `T → R` | `s -> s.length()` |
| `Consumer<T>` | `accept(T)` | `T → void` | `s -> System.out.println(s)` |
| `Supplier<T>` | `get()` | `() → T` | `() -> new ArrayList<>()` |
| `UnaryOperator<T>` | `apply(T)` | `T → T` | `s -> s.toUpperCase()` |
| `BinaryOperator<T>` | `apply(T,T)` | `(T,T) → T` | `(a, b) -> a + b` |
| `BiFunction<T,U,R>` | `apply(T,U)` | `(T,U) → R` | `(s, i) -> s.repeat(i)` |

### 8.2 Lambda Expressions

```java
// Before Java 8 — anonymous class
Comparator<String> comp = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
};

// Java 8+ — lambda expression
Comparator<String> comp = (a, b) -> a.length() - b.length();

// Method reference (even more concise)
Comparator<String> comp = Comparator.comparingInt(String::length);
```

### 8.3 Stream API

> **Streams are lazy pipelines** — intermediate operations are not executed until a terminal operation is called.

```java
List<Employee> employees = getEmployees();

// ✅ Stream pipeline: Source → Intermediate ops → Terminal op
List<String> highEarnerNames = employees.stream()
    .filter(e -> e.getSalary() > 80000)          // Intermediate: filter
    .sorted(Comparator.comparing(Employee::getName)) // Intermediate: sort
    .map(Employee::getName)                       // Intermediate: transform
    .distinct()                                   // Intermediate: remove duplicates
    .limit(10)                                    // Intermediate: take first 10
    .toList();                                    // Terminal: collect to List (Java 16+)

// Aggregation operations
double avgSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);

// Grouping
Map<String, List<Employee>> byDepartment = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Partitioning
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 100000));

// Reducing
int totalAge = employees.stream()
    .map(Employee::getAge)
    .reduce(0, Integer::sum);

// Chained collectors — average salary by department
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));
```

📊 **`map()` vs `flatMap()`:**

| Feature | `map()` | `flatMap()` |
|---------|---------|-------------|
| Transforms | `T → R` (1:1) | `T → Stream<R>` (1:many, then flattens) |
| Use case | Transform each element | Flatten nested collections |
| Example | `["Hello"] → [5]` (length) | `[["a","b"],["c"]] → ["a","b","c"]` |

```java
// map — transforms each element
List<String> names = employees.stream()
    .map(Employee::getName)
    .toList();

// flatMap — flattens nested structures
List<List<String>> nested = List.of(List.of("a", "b"), List.of("c", "d"));
List<String> flat = nested.stream()
    .flatMap(Collection::stream)
    .toList();  // ["a", "b", "c", "d"]
```

### 8.4 Optional

```java
// Avoid NullPointerException with Optional
Optional<Employee> found = employees.stream()
    .filter(e -> e.getName().equals("Shamil"))
    .findFirst();

// Safe operations
String name = found
    .map(Employee::getName)
    .orElse("Unknown");

// Throw if not found
Employee emp = found
    .orElseThrow(() -> new EmployeeNotFoundException("Not found"));

// Chaining
String city = Optional.ofNullable(employee)
    .map(Employee::getAddress)
    .map(Address::getCity)
    .orElse("Unknown City");
```

---

## 9. 🟡 I/O Operations

### 9.1 File I/O with NIO.2

```java
// Modern file operations (java.nio.file)
Path path = Path.of("data", "employees.txt");

// Write to file
Files.writeString(path, "Hello, Java!", StandardCharsets.UTF_8);
Files.write(path, List.of("Line 1", "Line 2"));

// Read from file
String content = Files.readString(path);
List<String> lines = Files.readAllLines(path);

// Stream file lines (memory-efficient for large files)
try (Stream<String> lineStream = Files.lines(path)) {
    long count = lineStream
        .filter(line -> line.contains("Java"))
        .count();
}

// Walk directory tree
try (Stream<Path> paths = Files.walk(Path.of("src"))) {
    paths.filter(p -> p.toString().endsWith(".java"))
         .forEach(System.out::println);
}

// Copy, Move, Delete
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
Files.deleteIfExists(path);
```

### 9.2 Serialization

```java
// Mark class as serializable
public class Employee implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;
    
    private String name;
    private transient String password;  // 'transient' — NOT serialized
    
    // ...
}

// Serialize (write object to file)
try (var oos = new ObjectOutputStream(new FileOutputStream("emp.ser"))) {
    oos.writeObject(employee);
}

// Deserialize (read object from file)
try (var ois = new ObjectInputStream(new FileInputStream("emp.ser"))) {
    Employee emp = (Employee) ois.readObject();
}
```

---

## 10. 🔴 Concurrency & Multithreading

> 🛑 **Prerequisite Check:** Before starting this section, ensure you are comfortable with basic classes, interfaces, and lambda expressions.
> 
> 💡 **Beginner Tip:** "Concurrency" just means doing multiple things at the same time. Imagine a restaurant with one chef (single-threaded) vs multiple chefs (multi-threaded). Multi-threading makes apps faster but is harder to coordinate!

### 10.1 Creating Threads

```java
// Method 1: Extend Thread
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread: " + Thread.currentThread().getName());
    }
}

// Method 2: Implement Runnable (preferred — allows extending another class)
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable: " + Thread.currentThread().getName());
    }
}

// Method 3: Lambda (Java 8+)
Thread thread = new Thread(() -> System.out.println("Lambda thread"));
thread.start();  // start(), NOT run()!
```

📊 **Runnable vs Callable:**

| Feature | Runnable | Callable |
|---------|----------|----------|
| Method | `run()` | `call()` |
| Return value | `void` | `V` (generic return type) |
| Exceptions | Cannot throw checked exceptions | Can throw checked exceptions |
| Used with | `Thread`, `ExecutorService` | `ExecutorService` only |
| Result | None | `Future<V>` |

### 10.2 Thread Synchronization

```java
// synchronized keyword — mutual exclusion
public class Counter {
    private int count = 0;
    
    // Synchronized method
    public synchronized void increment() {
        count++;  // Without sync, this is NOT atomic!
    }
    
    // Synchronized block (more granular)
    public void incrementV2() {
        synchronized (this) {
            count++;
        }
    }
}
```

📊 **`volatile` vs `synchronized`:**

| Feature | `volatile` | `synchronized` |
|---------|-----------|----------------|
| Visibility | ✅ Ensures visibility across threads | ✅ Ensures visibility |
| Atomicity | ❌ No compound atomicity | ✅ Yes |
| Locking | ❌ No lock | ✅ Acquires monitor lock |
| Use case | Simple flags/state visibility | Critical sections, compound operations |
| Performance | Faster (no locking) | Slower (lock overhead) |

### 10.3 `wait()` vs `sleep()`

| Feature | `wait()` | `sleep()` |
|---------|----------|-----------|
| Belongs to | `Object` | `Thread` |
| Lock release | ✅ Releases lock | ❌ Keeps lock |
| Wake up | `notify()` / `notifyAll()` | After duration |
| Must be in `synchronized` | ✅ Yes | ❌ No |

### 10.4 ExecutorService & Thread Pools

```java
// Fixed thread pool
ExecutorService executor = Executors.newFixedThreadPool(4);

// Submit tasks
Future<String> future = executor.submit(() -> {
    Thread.sleep(1000);
    return "Task completed!";
});

// Get result (blocks until complete)
String result = future.get();  // or future.get(5, TimeUnit.SECONDS)

// Shutdown properly
executor.shutdown();
executor.awaitTermination(10, TimeUnit.SECONDS);
```

### 10.5 CompletableFuture (Java 8+)

```java
// Asynchronous, non-blocking programming
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> fetchDataFromAPI())          // Run async
    .thenApply(data -> parseData(data))             // Transform result
    .thenApply(parsed -> enrichData(parsed))        // Chain another transform
    .exceptionally(ex -> {                          // Handle errors
        logger.error("Failed: {}", ex.getMessage());
        return "fallback-data";
    });

// Combine multiple futures
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> fetchUser());
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> fetchOrders());

CompletableFuture<String> combined = future1.thenCombine(future2,
    (user, orders) -> user + " has " + orders);

// Wait for ALL futures
CompletableFuture.allOf(future1, future2, future3).join();

// Wait for ANY future
CompletableFuture.anyOf(future1, future2).thenAccept(System.out::println);
```

### 10.6 Virtual Threads (Java 21+)

```java
// Virtual threads — lightweight, managed by JVM (not OS)
// Can create MILLIONS of virtual threads (vs ~few thousand platform threads)

// Method 1: Thread.ofVirtual()
Thread vThread = Thread.ofVirtual().start(() -> {
    System.out.println("Running on virtual thread: " + Thread.currentThread());
});

// Method 2: ExecutorService with virtual threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    // Submit 100,000 tasks — each gets its own virtual thread!
    IntStream.range(0, 100_000).forEach(i ->
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            return i;
        })
    );
}

// Method 3: Structured Concurrency (Preview in Java 21)
// Ensures parent task waits for all subtasks
```

📊 **Platform Threads vs Virtual Threads:**

| Feature | Platform Thread | Virtual Thread |
|---------|----------------|----------------|
| Managed by | OS | JVM |
| Cost | Heavy (~1MB stack) | Lightweight (~few KB) |
| Max count | ~few thousand | **Millions** |
| Blocking | Blocks OS thread | JVM unmounts, reuses carrier |
| Best for | CPU-intensive tasks | I/O-bound tasks |

---

## 11. 🔴 JVM Internals

> 🛑 **Prerequisite Check:** This is an advanced topic. Make sure you understand how objects and variables work in Java before diving deep into memory management.

### 11.1 JVM Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                         JVM Architecture                         │
├─────────────────┬────────────────────────┬───────────────────────┤
│  Class Loader   │   Runtime Data Areas   │   Execution Engine    │
│  Subsystem      │                        │                       │
│                 │  ┌──────────────────┐   │  ┌────────────────┐  │
│  Bootstrap ───▶ │  │   Method Area    │   │  │  Interpreter   │  │
│  Extension ───▶ │  │ (Class metadata) │   │  │                │  │
│  Application──▶ │  ├──────────────────┤   │  ├────────────────┤  │
│                 │  │      Heap        │   │  │  JIT Compiler  │  │
│  Loading ──▶    │  │ (Objects, GC)    │   │  │  (Hot code)    │  │
│  Linking ──▶    │  ├──────────────────┤   │  ├────────────────┤  │
│  Initialization │  │   Stack (per     │   │  │  Garbage       │  │
│                 │  │    thread)       │   │  │  Collector     │  │
│                 │  ├──────────────────┤   │  │                │  │
│                 │  │   PC Register    │   │  └────────────────┘  │
│                 │  ├──────────────────┤   │                       │
│                 │  │ Native Method    │   │                       │
│                 │  │   Stack          │   │                       │
│                 │  └──────────────────┘   │                       │
└─────────────────┴────────────────────────┴───────────────────────┘
```

### 11.2 Memory Model (Heap vs Stack)

| Feature | Stack | Heap |
|---------|-------|------|
| Stores | Local variables, method calls, primitives | Objects, instance variables |
| Lifetime | Method scope | Until GC collects |
| Thread access | Per-thread (private) | Shared across threads |
| Speed | Faster | Slower |
| Size | Small (configurable: `-Xss`) | Large (configurable: `-Xmx`) |
| Error | `StackOverflowError` | `OutOfMemoryError` |

### 11.3 Garbage Collection

```
Heap Memory Layout:
┌────────────────────────────────────────────────────────┐
│  Young Generation (Minor GC — frequent, fast)          │
│  ┌─────────────┬──────────────┬──────────────┐        │
│  │    Eden     │  Survivor S0 │  Survivor S1 │        │
│  │ (new objs)  │              │              │        │
│  └─────────────┴──────────────┴──────────────┘        │
├────────────────────────────────────────────────────────┤
│  Old Generation (Major GC / Full GC — infrequent, slow)│
│  ┌────────────────────────────────────────────┐        │
│  │  Long-lived objects (survived many GCs)    │        │
│  └────────────────────────────────────────────┘        │
├────────────────────────────────────────────────────────┤
│  Metaspace (off-heap, replaces PermGen in Java 8+)     │
│  Class metadata, method definitions                    │
└────────────────────────────────────────────────────────┘
```

📊 **GC Algorithms:**

| GC | Best For | Pause Type | Flag |
|----|----------|-----------|------|
| **G1** (default) | General purpose, balanced | Short pauses | `-XX:+UseG1GC` |
| **ZGC** | Ultra-low latency (<1ms pauses) | Sub-millisecond | `-XX:+UseZGC` |
| **Shenandoah** | Low latency, concurrent | Sub-millisecond | `-XX:+UseShenandoahGC` |
| **Parallel GC** | Throughput-focused | Longer pauses OK | `-XX:+UseParallelGC` |
| **Serial GC** | Small heaps, single CPU | Stop-the-world | `-XX:+UseSerialGC` |

```bash
# Common JVM tuning flags
java -Xms512m -Xmx2g -XX:+UseG1GC -XX:MaxGCPauseMillis=200 \
     -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heap.hprof \
     -jar application.jar
```

---

## 12. 🔴 Modern Java Features (8–21+)

### 12.1 Feature Timeline

| Version | Key Features | Year |
|---------|-------------|------|
| Java 8 | Lambdas, Streams, Optional, `java.time`, Default Methods | 2014 |
| Java 9 | Modules (JPMS), JShell, `List.of()`, `Stream.ofNullable()` | 2017 |
| Java 10 | `var` (local variable type inference) | 2018 |
| Java 11 (LTS) | `String` methods, `HttpClient`, `var` in lambdas | 2018 |
| Java 14 | `switch` expressions, `NullPointerException` messages | 2020 |
| Java 15 | Text blocks `"""` | 2020 |
| Java 16 | Records, `instanceof` pattern matching | 2021 |
| Java 17 (LTS) | Sealed classes, enhanced `switch` | 2021 |
| Java 21 (LTS) | Virtual threads, sequenced collections, pattern matching | 2023 |

### 12.2 Key Modern Features

```java
// var — local type inference (Java 10+)
var names = new ArrayList<String>();  // Inferred as ArrayList<String>
var stream = names.stream();          // Inferred as Stream<String>

// Records (Java 16+) — see Section 3.7

// Sealed Classes (Java 17+) — restrict which classes can extend
public sealed class Shape permits Circle, Rectangle, Triangle {
    // Only Circle, Rectangle, Triangle can extend Shape
}
public final class Circle extends Shape { }
public non-sealed class Rectangle extends Shape { }  // Open for further extension

// Pattern Matching for instanceof (Java 16+)
if (obj instanceof String s && s.length() > 5) {
    System.out.println(s.toUpperCase());  // 's' is auto-cast!
}

// Pattern Matching for switch (Java 21+)
String describe(Object obj) {
    return switch (obj) {
        case Integer i when i > 0 -> "Positive integer: " + i;
        case Integer i            -> "Non-positive integer: " + i;
        case String s             -> "String of length " + s.length();
        case null                 -> "null value";
        default                   -> "Unknown: " + obj;
    };
}

// Sequenced Collections (Java 21+)
SequencedCollection<String> list = new ArrayList<>();
list.addFirst("First");
list.addLast("Last");
list.getFirst();
list.reversed();  // Reversed view
```

---

## 13. 🟡 Date & Time API

```java
// Modern java.time API (Java 8+) — ALWAYS use this, never java.util.Date

// Current date/time
LocalDate today = LocalDate.now();                    // 2026-09-06
LocalTime now = LocalTime.now();                      // 22:30:15.123
LocalDateTime dateTime = LocalDateTime.now();         // 2026-09-06T22:30:15.123
ZonedDateTime zoned = ZonedDateTime.now(ZoneId.of("Asia/Kolkata"));
Instant instant = Instant.now();                      // UTC timestamp

// Creating specific dates
LocalDate birthday = LocalDate.of(1998, Month.MARCH, 15);
LocalDate parsed = LocalDate.parse("2026-09-06");

// Operations (immutable — returns new instances)
LocalDate nextWeek = today.plusWeeks(1);
LocalDate lastMonth = today.minusMonths(1);
Period age = Period.between(birthday, today);  // "28 years, 5 months, 22 days"
Duration duration = Duration.ofHours(8);       // PT8H

// Formatting
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd-MMM-yyyy HH:mm");
String formatted = dateTime.format(fmt);  // "06-Sep-2026 22:30"

// Parsing
LocalDateTime parsed = LocalDateTime.parse("06-Sep-2026 22:30", fmt);
```

---

## 14. 🟡 Networking & Regular Expressions

### 14.1 HTTP Client (Java 11+)

```java
HttpClient client = HttpClient.newBuilder()
    .connectTimeout(Duration.ofSeconds(10))
    .build();

// GET request
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Accept", "application/json")
    .GET()
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.statusCode());  // 200
System.out.println(response.body());        // JSON string

// POST request
HttpRequest postReq = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString("""
        {"name": "Shamil", "role": "Developer"}
        """))
    .build();

// Async request
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println)
    .join();
```

### 14.2 Regular Expressions

```java
// Basic pattern matching
Pattern pattern = Pattern.compile("\\b[A-Z][a-z]+\\b");
Matcher matcher = pattern.matcher("Hello World Java Developer");
while (matcher.find()) {
    System.out.println(matcher.group());  // Hello, World, Java, Developer
}

// Common patterns
String email = "test@example.com";
boolean valid = email.matches("^[\\w.-]+@[\\w.-]+\\.[a-zA-Z]{2,}$");

// Replace
String cleaned = "Hello   World".replaceAll("\\s+", " ");  // "Hello World"

// Split
String[] parts = "one,two,,three".split(",", -1);  // ["one", "two", "", "three"]
```

---

## 15. 🔴 Reflection & Annotations

### 15.1 Reflection API

```java
// Inspect class at runtime
Class<?> clazz = Employee.class;  // or Class.forName("com.example.Employee")

// Get all methods
Method[] methods = clazz.getDeclaredMethods();
for (Method m : methods) {
    System.out.printf("%s %s(%s)%n",
        m.getReturnType().getSimpleName(),
        m.getName(),
        Arrays.stream(m.getParameterTypes())
              .map(Class::getSimpleName)
              .collect(Collectors.joining(", "))
    );
}

// Invoke method dynamically
Object employee = clazz.getDeclaredConstructor(String.class, int.class, double.class)
    .newInstance("Shamil", 28, 95000);
Method getName = clazz.getMethod("getName");
String name = (String) getName.invoke(employee);

// Access private field
Field salaryField = clazz.getDeclaredField("salary");
salaryField.setAccessible(true);  // Bypass access control
double salary = (double) salaryField.get(employee);
```

### 15.2 Custom Annotations

```java
// Define a custom annotation
@Retention(RetentionPolicy.RUNTIME)  // Available at runtime
@Target(ElementType.METHOD)          // Can only be applied to methods
public @interface RateLimit {
    int maxRequests() default 100;
    int periodSeconds() default 60;
    String message() default "Rate limit exceeded";
}

// Use it
public class UserController {
    @RateLimit(maxRequests = 10, periodSeconds = 30)
    public ResponseEntity<List<User>> getUsers() {
        return ResponseEntity.ok(userService.findAll());
    }
}

// Process it via reflection (at runtime)
Method method = UserController.class.getMethod("getUsers");
if (method.isAnnotationPresent(RateLimit.class)) {
    RateLimit rl = method.getAnnotation(RateLimit.class);
    System.out.println("Max requests: " + rl.maxRequests());  // 10
}
```

---

## 16. 🟡 Build Tools & Dependency Management

### 16.1 Maven (Industry Standard)

```xml
<!-- pom.xml — Maven project descriptor -->
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <properties>
        <java.version>21</java.version>
        <spring-boot.version>3.3.0</spring-boot.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring-boot.version}</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

```bash
# Maven lifecycle
mvn clean           # Delete target/
mvn compile          # Compile source code
mvn test             # Run unit tests
mvn package          # Create JAR/WAR
mvn install          # Install to local repo (~/.m2)
mvn deploy           # Deploy to remote repo
mvn spring-boot:run  # Run Spring Boot app
```

### 16.2 Gradle

```groovy
// build.gradle (Groovy DSL)
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.0'
}

java {
    sourceCompatibility = JavaVersion.VERSION_21
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

```bash
# Gradle commands
gradle build        # Build project
gradle test          # Run tests
gradle bootRun       # Run Spring Boot app
gradle dependencies  # Show dependency tree
```

---

## 17. 🟡 Databases & JDBC

### 17.1 JDBC Basics

```java
// Modern JDBC with try-with-resources
String url = "jdbc:postgresql://localhost:5432/mydb";
String sql = "SELECT id, name, salary FROM employees WHERE department = ?";

try (Connection conn = DriverManager.getConnection(url, "user", "pass");
     PreparedStatement ps = conn.prepareStatement(sql)) {
    
    ps.setString(1, "Engineering");  // Parameterized query (prevents SQL injection!)
    
    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            int id = rs.getInt("id");
            String name = rs.getString("name");
            double salary = rs.getDouble("salary");
            System.out.printf("ID: %d, Name: %s, Salary: %.2f%n", id, name, salary);
        }
    }
}

// Transaction management
Connection conn = dataSource.getConnection();
try {
    conn.setAutoCommit(false);  // Start transaction
    
    // Execute multiple statements...
    updateAccount(conn, fromAccount, -amount);
    updateAccount(conn, toAccount, +amount);
    
    conn.commit();  // All or nothing
} catch (SQLException e) {
    conn.rollback();  // Undo all changes
    throw e;
} finally {
    conn.setAutoCommit(true);
    conn.close();
}
```

### 17.2 Connection Pooling (HikariCP)

```yaml
# application.yml (Spring Boot + HikariCP)
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: user
    password: secret
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
```

---

## 18. 🔴 ORM (JPA & Hibernate)

> **Goal:** Map Java objects to database tables. Handle relationships, caching, and performance.

### 18.1 Entity Mapping

```java
@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(nullable = false)
    private Double salary;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
    
    @OneToMany(mappedBy = "employee", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Address> addresses = new ArrayList<>();
    
    @Enumerated(EnumType.STRING)
    private EmployeeStatus status;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}
```

### 18.2 The N+1 Problem

```
❌ N+1 Problem:
Query 1: SELECT * FROM departments;                    (1 query)
Query 2: SELECT * FROM employees WHERE dept_id = 1;    (N queries,
Query 3: SELECT * FROM employees WHERE dept_id = 2;     one per
Query 4: SELECT * FROM employees WHERE dept_id = 3;     department)
...                                                     = N+1 total!
```

```java
// ❌ Causes N+1 — fetches employees lazily, one query per department
List<Department> departments = departmentRepository.findAll();
departments.forEach(d -> d.getEmployees().size());  // N additional queries!

// ✅ Fix 1: JOIN FETCH (JPQL)
@Query("SELECT d FROM Department d JOIN FETCH d.employees")
List<Department> findAllWithEmployees();

// ✅ Fix 2: @EntityGraph
@EntityGraph(attributePaths = {"employees"})
List<Department> findAll();

// ✅ Fix 3: Batch fetching
@BatchSize(size = 25)  // Fetch employees in batches of 25
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

📊 **FetchType.LAZY vs FetchType.EAGER:**

| Feature | LAZY | EAGER |
|---------|------|-------|
| Loading | On-demand (when accessed) | Immediately with parent |
| Performance | Better (fewer queries upfront) | Can cause performance issues |
| Default for | `@OneToMany`, `@ManyToMany` | `@ManyToOne`, `@OneToOne` |
| Risk | `LazyInitializationException` | Excessive data loading |

### 18.3 Caching

```
Hibernate Caching Architecture:
┌──────────────────────────────────────────────────┐
│  1st Level Cache (Session/EntityManager level)    │
│  ✅ Always ON | Per-transaction | Automatic       │
├──────────────────────────────────────────────────┤
│  2nd Level Cache (SessionFactory level)           │
│  ❌ OFF by default | Shared across sessions       │
│  Providers: EhCache, Hazelcast, Infinispan        │
├──────────────────────────────────────────────────┤
│  Query Cache (caches query results)               │
│  ❌ OFF by default | Use with caution             │
└──────────────────────────────────────────────────┘
```

---

## 19. 🔴 Spring Framework & Spring Boot

> 🛑 **Prerequisite Check:** You must understand OOP, Interfaces, and basic HTTP/Databases before learning Spring. 
> 
> 💡 **Beginner Tip:** Spring Boot is a "framework" — a pre-built structure that makes creating web apps much faster. It does a lot of the heavy lifting for you!

### 19.1 IoC / Dependency Injection

```java
// Constructor injection (RECOMMENDED — makes dependencies explicit, enables testing)
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final NotificationService notificationService;
    
    // @Autowired is optional with single constructor (Spring 4.3+)
    public OrderService(OrderRepository orderRepository,
                        PaymentService paymentService,
                        NotificationService notificationService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
        this.notificationService = notificationService;
    }
    
    @Transactional
    public Order createOrder(OrderRequest request) {
        Order order = orderRepository.save(new Order(request));
        paymentService.processPayment(order);
        notificationService.sendConfirmation(order);
        return order;
    }
}
```

📊 **`@Component` vs `@Service` vs `@Repository` vs `@Controller`:**

| Annotation | Layer | Special Behavior |
|-----------|-------|-----------------|
| `@Component` | Generic | Base stereotype, no special behavior |
| `@Service` | Business Logic | Semantic marker for service layer |
| `@Repository` | Data Access | Exception translation (DB exceptions → Spring `DataAccessException`) |
| `@Controller` | Web/MVC | Returns views (HTML) |
| `@RestController` | Web/REST | `@Controller` + `@ResponseBody` (returns JSON/XML) |

### 19.2 Spring Boot Auto-Configuration

```java
// Spring Boot starter — includes auto-configuration for common setups
@SpringBootApplication  // = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

```
Auto-configuration flow:
1. Spring Boot scans META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
2. Each auto-config class has @Conditional annotations
3. Beans are created ONLY if conditions are met:
   - @ConditionalOnClass — class is on classpath
   - @ConditionalOnMissingBean — no user-defined bean exists
   - @ConditionalOnProperty — property is set
```

### 19.3 RESTful API Development

```java
@RestController
@RequestMapping("/api/v1/employees")
public class EmployeeController {
    
    private final EmployeeService employeeService;
    
    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }
    
    @GetMapping
    public ResponseEntity<List<EmployeeDTO>> getAllEmployees(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        return ResponseEntity.ok(employeeService.findAll(page, size));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<EmployeeDTO> getById(@PathVariable Long id) {
        return ResponseEntity.ok(employeeService.findById(id));
    }
    
    @PostMapping
    public ResponseEntity<EmployeeDTO> create(@Valid @RequestBody CreateEmployeeRequest request) {
        EmployeeDTO created = employeeService.create(request);
        URI location = URI.create("/api/v1/employees/" + created.id());
        return ResponseEntity.created(location).body(created);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<EmployeeDTO> update(@PathVariable Long id,
                                               @Valid @RequestBody UpdateEmployeeRequest request) {
        return ResponseEntity.ok(employeeService.update(id, request));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        employeeService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### 19.4 Global Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                FieldError::getDefaultMessage,
                (a, b) -> a
            ));
        ErrorResponse error = new ErrorResponse(400, "Validation failed", errors);
        return ResponseEntity.badRequest().body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse(500, "Internal server error", LocalDateTime.now());
        return ResponseEntity.status(500).body(error);
    }
}

record ErrorResponse(int status, String message, Object details) {}
```

### 19.5 Spring Security + JWT

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 19.6 Spring Bean Lifecycle

```
Bean Lifecycle:
┌──────────────────────────────────────────────────────────────────┐
│ 1. Instantiation (Constructor)                                   │
│ 2. Dependency Injection (Setter/Field injection)                 │
│ 3. @PostConstruct / InitializingBean.afterPropertiesSet()        │
│ 4. Custom init-method                                            │
│ 5. ──── Bean is ready to use ────                                │
│ 6. @PreDestroy / DisposableBean.destroy()                        │
│ 7. Custom destroy-method                                         │
└──────────────────────────────────────────────────────────────────┘
```

### 19.7 Spring Boot Actuator

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  endpoint:
    health:
      show-details: always
```

```
Available endpoints:
/actuator/health       — Application health check
/actuator/info         — Application information
/actuator/metrics      — Application metrics
/actuator/env          — Environment properties
/actuator/loggers      — View/change log levels at runtime
/actuator/prometheus   — Prometheus metrics (for Grafana)
```

---

## 20. 🟡 Testing

### 20.1 JUnit 5

```java
@ExtendWith(MockitoExtension.class)
class EmployeeServiceTest {
    
    @Mock
    private EmployeeRepository employeeRepository;
    
    @InjectMocks
    private EmployeeService employeeService;
    
    @Test
    @DisplayName("Should return employee when valid ID is provided")
    void shouldReturnEmployee_WhenValidId() {
        // Arrange
        Employee expected = new Employee("Shamil", 28, 95000);
        when(employeeRepository.findById(1L)).thenReturn(Optional.of(expected));
        
        // Act
        Employee result = employeeService.findById(1L);
        
        // Assert
        assertNotNull(result);
        assertEquals("Shamil", result.getName());
        assertEquals(95000, result.getSalary());
        verify(employeeRepository, times(1)).findById(1L);
    }
    
    @Test
    @DisplayName("Should throw exception when employee not found")
    void shouldThrowException_WhenEmployeeNotFound() {
        when(employeeRepository.findById(999L)).thenReturn(Optional.empty());
        
        assertThrows(ResourceNotFoundException.class,
            () -> employeeService.findById(999L));
    }
    
    @ParameterizedTest
    @ValueSource(doubles = {-100, -1, 0})
    @DisplayName("Should reject invalid salaries")
    void shouldRejectInvalidSalaries(double salary) {
        assertThrows(IllegalArgumentException.class,
            () -> new Employee("Test", 25, salary));
    }
}
```

### 20.2 Integration Testing

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class EmployeeControllerIT {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Test
    void shouldCreateAndRetrieveEmployee() {
        // Create
        var request = new CreateEmployeeRequest("Shamil", 28, 95000);
        ResponseEntity<EmployeeDTO> createResponse = restTemplate
            .postForEntity("/api/v1/employees", request, EmployeeDTO.class);
        assertEquals(HttpStatus.CREATED, createResponse.getStatusCode());
        
        // Retrieve
        Long id = createResponse.getBody().id();
        ResponseEntity<EmployeeDTO> getResponse = restTemplate
            .getForEntity("/api/v1/employees/" + id, EmployeeDTO.class);
        assertEquals("Shamil", getResponse.getBody().name());
    }
}
```

---

## 21. 🟢 Logging

```java
// SLF4J + Logback (Spring Boot default)
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class OrderService {
    private static final Logger log = LoggerFactory.getLogger(OrderService.class);
    // Or use Lombok: @Slf4j at class level
    
    public Order createOrder(OrderRequest request) {
        log.info("Creating order for customer: {}", request.customerId());
        log.debug("Order details: {}", request);
        
        try {
            Order order = processOrder(request);
            log.info("Order created successfully: orderId={}", order.getId());
            return order;
        } catch (PaymentException e) {
            log.error("Payment failed for customer {}: {}", 
                request.customerId(), e.getMessage(), e);  // Logs full stack trace
            throw e;
        }
    }
}
```

```yaml
# application.yml — logging configuration
logging:
  level:
    root: INFO
    com.example.service: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/application.log
```

---

## 22. 🔴 Design Patterns

> 🔴 **Singleton and Factory asked by 4/9 companies**

### 22.1 Creational Patterns

```java
// Singleton (Thread-safe, Bill Pugh approach)
public class DatabaseConnection {
    private DatabaseConnection() {}
    
    private static class Holder {
        private static final DatabaseConnection INSTANCE = new DatabaseConnection();
    }
    
    public static DatabaseConnection getInstance() {
        return Holder.INSTANCE;
    }
}

// Builder Pattern
public class HttpRequest {
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    private final String body;
    
    private HttpRequest(Builder builder) {
        this.url = builder.url;
        this.method = builder.method;
        this.headers = builder.headers;
        this.body = builder.body;
    }
    
    public static class Builder {
        private String url;
        private String method = "GET";
        private Map<String, String> headers = new HashMap<>();
        private String body;
        
        public Builder url(String url) { this.url = url; return this; }
        public Builder method(String method) { this.method = method; return this; }
        public Builder header(String key, String value) { headers.put(key, value); return this; }
        public Builder body(String body) { this.body = body; return this; }
        
        public HttpRequest build() {
            if (url == null) throw new IllegalStateException("URL is required");
            return new HttpRequest(this);
        }
    }
}

// Factory Pattern
public interface NotificationFactory {
    Notification create(String type);
}

public class NotificationFactoryImpl implements NotificationFactory {
    @Override
    public Notification create(String type) {
        return switch (type.toLowerCase()) {
            case "email" -> new EmailNotification();
            case "sms" -> new SmsNotification();
            case "push" -> new PushNotification();
            default -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}
```

### 22.2 Structural Patterns

```java
// Adapter Pattern — makes incompatible interfaces work together
public class PaymentAdapter implements PaymentGateway {
    private final LegacyPaymentSystem legacySystem;
    
    public PaymentAdapter(LegacyPaymentSystem legacySystem) {
        this.legacySystem = legacySystem;
    }
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        // Adapt new interface to legacy system
        LegacyPaymentData data = convertToLegacyFormat(request);
        int status = legacySystem.makePayment(data);
        return convertFromLegacyResult(status);
    }
}
```

### 22.3 Behavioral Patterns

```java
// Strategy Pattern — select algorithm at runtime
@FunctionalInterface
public interface PricingStrategy {
    double calculatePrice(double basePrice);
}

public class PricingService {
    private final Map<String, PricingStrategy> strategies = Map.of(
        "REGULAR", price -> price,
        "PREMIUM", price -> price * 0.9,  // 10% discount
        "VIP", price -> price * 0.8       // 20% discount
    );
    
    public double calculateFinalPrice(double basePrice, String customerType) {
        return strategies.getOrDefault(customerType, p -> p)
                         .calculatePrice(basePrice);
    }
}

// Observer Pattern — Spring's ApplicationEvent
public record OrderCreatedEvent(Long orderId, String customerEmail) {}

@Component
public class OrderNotificationListener {
    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        sendEmail(event.customerEmail(), "Your order #" + event.orderId() + " is confirmed!");
    }
}
```

---

## 23. 🔴 Microservices Architecture

> 🔴 **CRITICAL — Microservices vs Monolithic asked by 9/9 companies**

### 23.1 Microservices vs Monolithic

| Feature | Monolithic | Microservices |
|---------|-----------|---------------|
| Deployment | Single unit | Independent services |
| Scaling | Scale entire app | Scale individual services |
| Technology | Single tech stack | Polyglot (mix languages) |
| Team structure | Single large team | Small, autonomous teams |
| Data | Single database | Database per service |
| Communication | In-process calls | Network calls (HTTP/gRPC/messaging) |
| Complexity | Simple to start | Complex infrastructure |
| Failure | Single point of failure | Partial failures, need resilience |

### 23.2 Key Patterns

```
Microservices Architecture:
┌────────┐     ┌──────────────┐     ┌─────────────────┐
│ Client │────▶│  API Gateway │────▶│ Service Discovery│
│        │     │ (Spring Cloud│     │ (Eureka / K8s    │
│        │     │  Gateway)    │     │  DNS)            │
└────────┘     └──────┬───────┘     └─────────────────┘
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐
    │ User     │ │ Order    │ │ Payment  │
    │ Service  │ │ Service  │ │ Service  │
    │          │ │          │ │          │
    │ [UserDB] │ │ [OrderDB]│ │ [PayDB]  │
    └──────────┘ └──────────┘ └──────────┘
          │           │           │
          └───────────┼───────────┘
                      ▼
              ┌──────────────┐
              │  Message     │
              │  Broker      │
              │  (Kafka/     │
              │   RabbitMQ)  │
              └──────────────┘
```

### 23.3 Circuit Breaker (Resilience4j)

```java
@Service
public class OrderService {
    
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService", fallbackMethod = "paymentFallback")
    @TimeLimiter(name = "paymentService")
    public CompletableFuture<PaymentResponse> processPayment(PaymentRequest request) {
        return CompletableFuture.supplyAsync(
            () -> paymentClient.charge(request)
        );
    }
    
    // Fallback method — called when circuit opens or retries exhausted
    private CompletableFuture<PaymentResponse> paymentFallback(PaymentRequest request, Throwable ex) {
        log.warn("Payment service unavailable, queuing for retry: {}", ex.getMessage());
        return CompletableFuture.completedFuture(
            new PaymentResponse("QUEUED", "Payment will be processed later")
        );
    }
}
```

```yaml
# Resilience4j configuration
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 30s
        permitted-number-of-calls-in-half-open-state: 3
  retry:
    instances:
      paymentService:
        max-attempts: 3
        wait-duration: 1s
        exponential-backoff-multiplier: 2
```

### 23.4 Saga Pattern (Distributed Transactions)

```
Choreography-based Saga:
┌──────────┐  OrderCreated  ┌──────────┐  PaymentDone  ┌──────────┐
│  Order   │ ──────────────▶│ Payment  │───────────────▶│ Inventory│
│  Service │                │ Service  │                │ Service  │
└──────────┘                └──────────┘                └──────────┘
     ▲                           │                           │
     │        PaymentFailed      │     InventoryFailed       │
     └───────────────────────────┘◀──────────────────────────┘
           (Compensating transactions — rollback)
```

---

## 24. 🔴 DevOps Essentials

### 24.1 Docker

```dockerfile
# Multi-stage Dockerfile for Spring Boot (production-optimized)
# Stage 1: Build
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN apk add --no-cache maven && mvn clean package -DskipTests

# Stage 2: Runtime (smaller image)
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

# Non-root user for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 8080
HEALTHCHECK --interval=30s CMD wget -qO- http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-XX:+UseG1GC", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

```yaml
# docker-compose.yml
version: '3.9'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/mydb
    depends_on:
      db:
        condition: service_healthy
    
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

### 24.2 Kubernetes Basics

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: java-app
  template:
    metadata:
      labels:
        app: java-app
    spec:
      containers:
        - name: java-app
          image: myregistry/java-app:1.0.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "512Mi"
              cpu: "500m"
            limits:
              memory: "1Gi"
              cpu: "1000m"
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 30
---
apiVersion: v1
kind: Service
metadata:
  name: java-app-service
spec:
  type: ClusterIP
  selector:
    app: java-app
  ports:
    - port: 80
      targetPort: 8080
```

### 24.3 CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/ci-cd.yml
name: Java CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: maven
      
      - name: Build & Test
        run: mvn clean verify
      
      - name: Build Docker Image
        if: github.ref == 'refs/heads/main'
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Push to Registry
        if: github.ref == 'refs/heads/main'
        run: |
          docker tag myapp:${{ github.sha }} myregistry/myapp:latest
          docker push myregistry/myapp:latest
```

### 24.4 AWS Core Services

| Service | Purpose | Java Integration |
|---------|---------|-----------------|
| **EC2** | Virtual machines | Deploy JAR/Docker |
| **S3** | Object storage | `aws-sdk-java-v2` for file uploads |
| **RDS** | Managed databases | PostgreSQL/MySQL for Spring Boot |
| **ECS/EKS** | Container orchestration | Deploy Docker/K8s |
| **API Gateway** | API management | Front for Lambda/ECS |
| **Lambda** | Serverless functions | Spring Cloud Function |
| **SQS/SNS** | Messaging | Event-driven microservices |
| **CloudWatch** | Monitoring & Logging | Spring Boot Actuator metrics |

---

## 🗺️ Learning Path Summary

```
PHASE 1 — Foundation (Weeks 1-4)
├── Sections 1-4: Fundamentals, Program Structure, OOP, Strings
└── 🎯 Build: Console-based Student Management System

PHASE 2 — Core Java (Weeks 5-8)
├── Sections 5-9: Exceptions, Collections, Generics, Streams, I/O
└── 🎯 Build: File-based Employee CRUD Application

PHASE 3 — Advanced Java (Weeks 9-12)
├── Sections 10-15: Concurrency, JVM, Modern Features, Reflection
└── 🎯 Build: Multi-threaded Web Scraper / Chat Server

PHASE 4 — Enterprise Java (Weeks 13-18)
├── Sections 16-21: Maven, JDBC, JPA, Spring Boot, Testing, Logging
└── 🎯 Build: Full REST API with Spring Boot + PostgreSQL + JWT Auth

PHASE 5 — Architecture & DevOps (Weeks 19-24)
├── Sections 22-24: Design Patterns, Microservices, DevOps
└── 🎯 Build: Microservices E-commerce System (Docker + K8s + CI/CD)
```

---

## 📚 Recommended Resources

| Resource | Type | Best For |
|----------|------|----------|
| [roadmap.sh/java](https://roadmap.sh/java) | Interactive Roadmap | Visual progress tracking |
| *Effective Java* — Joshua Bloch | Book | Best practices & idioms |
| *Java Concurrency in Practice* — Brian Goetz | Book | Multithreading mastery |
| [Baeldung](https://www.baeldung.com/) | Tutorials | Spring Boot & Java how-tos |
| [Java Brains (YouTube)](https://www.youtube.com/@Java.Brains) | Video | Spring Boot deep dives |
| [LeetCode](https://leetcode.com/) | Practice | DSA with Java |
| [Spring.io Guides](https://spring.io/guides) | Official Docs | Spring Boot getting started |

---

> *This roadmap is part of the `Java-interview-prep` repository.*
> *Based on [roadmap.sh/java](https://roadmap.sh/java) — adapted and expanded for Senior Full Stack Java Developer preparation.*
> *Total Topics: 24 sections | 100+ sub-topics | Beginner → Advanced*
