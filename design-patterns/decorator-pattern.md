---
title: Decorator
parent: Design Patterns and LLD
nav_order: 7
---

# Decorator
{: .no_toc }

Adds new behavior to an object dynamically without modifying its original class.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

1. TOC
{:toc}
</details>

---

## 1. What is the Decorator Design Pattern?

{: .label .label-green }

**Answer**

The Decorator is a structural design pattern that adds additional behavior to an individual object by wrapping it inside another object that implements the same interface.

Instead of modifying the original class or creating multiple subclasses for every possible combination of behavior, we can combine decorators at runtime.

### Real-world example

Consider a coffee shop where a customer can order a basic coffee and add milk, sugar, or whipped cream.

- Basic coffee costs ₹100.
- Milk adds ₹20.
- Whipped cream adds ₹30.

Instead of creating separate classes such as `CoffeeWithMilk`, `CoffeeWithCream`, and `CoffeeWithMilkAndCream`, we can wrap the coffee object with different decorators.

### Key components

- **Component:** Defines the common interface.
- **Concrete Component:** The original object being decorated.
- **Decorator:** Holds a reference to another component and implements the same interface.
- **Concrete Decorator:** Adds specific behavior before or after delegating to the wrapped component.

---

## 2. How do you implement the Decorator Pattern in Java?

{: .label .label-green }

**Answer**

Let's implement a coffee ordering system in which customers can add milk and whipped cream to their coffee.

### Complete Java example

```java
// Component
interface Coffee {
    String getDescription();
    double getCost();
}

// Concrete Component
class SimpleCoffee implements Coffee {

    @Override
    public String getDescription() {
        return "Simple Coffee";
    }

    @Override
    public double getCost() {
        return 100;
    }
}

// Base Decorator
abstract class CoffeeDecorator implements Coffee {

    protected final Coffee coffee;

    protected CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}

// Concrete Decorator: Milk
class MilkDecorator extends CoffeeDecorator {

    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 20;
    }
}

// Concrete Decorator: Whipped Cream
class CreamDecorator extends CoffeeDecorator {

    public CreamDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Cream";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 30;
    }
}

// Client
public class Main {

    public static void main(String[] args) {

        Coffee coffee = new SimpleCoffee();

        coffee = new MilkDecorator(coffee);
        coffee = new CreamDecorator(coffee);

        System.out.println(coffee.getDescription());
        System.out.println(coffee.getCost());
    }
}
```

### Output

```text
Simple Coffee, Milk, Cream
150.0
```

### How it works

1. `SimpleCoffee` provides the basic coffee functionality.
2. `MilkDecorator` wraps the coffee and adds ₹20.
3. `CreamDecorator` wraps the milk-decorated coffee and adds ₹30.
4. Each decorator delegates to the wrapped object and adds its own behavior.
5. The client uses the `Coffee` interface throughout the process.

The final cost is:

\[
100 + 20 + 30 = 150
\]

**Important:** Every decorator implements the same `Coffee` interface, so decorators can be combined in different orders without changing the original coffee class.

---

## 3. Why do we need the Decorator Pattern?

{: .label .label-green }

**Answer**

The Decorator Pattern is useful when we need to add optional or combinable behavior to objects without modifying their existing classes.

### Problem without Decorator

Suppose we create a coffee shop with these classes:

```java
class SimpleCoffee { }
class CoffeeWithMilk { }
class CoffeeWithCream { }
class CoffeeWithMilkAndCream { }
class CoffeeWithMilkAndSugar { }
class CoffeeWithMilkCreamAndSugar { }
```

As the number of optional features grows, the number of combinations can grow rapidly.

With \(n\) independent optional features, there can be up to \(2^n\) combinations.

### Solution with Decorator

Use one decorator per feature:

```java
Coffee coffee = new SimpleCoffee();

coffee = new MilkDecorator(coffee);
coffee = new CreamDecorator(coffee);
```

You can add or omit decorators as needed.

### Benefits

- Avoids creating a separate subclass for every combination.
- Adds behavior at runtime.
- Keeps the original class unchanged.
- Allows decorators to be combined in different orders.
- Supports the Open/Closed Principle when new behaviors can be introduced through new decorators.

---

## 4. How does the Decorator Pattern work internally?

{: .label .label-blue }

**Answer**

A decorator implements the same interface as the object it wraps and stores a reference to that object.

When the client calls a method, the decorator can:

1. Perform additional work before delegation.
2. Call the wrapped object's method.
3. Modify the result or perform additional work afterward.

### Example: Logging decorator

```java
interface MessageService {
    String send(String message);
}

class BasicMessageService implements MessageService {

    @Override
    public String send(String message) {
        return "Sent: " + message;
    }
}

class LoggingDecorator implements MessageService {

    private final MessageService service;

    public LoggingDecorator(MessageService service) {
        this.service = service;
    }

    @Override
    public String send(String message) {
        System.out.println("Sending message...");
        String result = service.send(message);
        System.out.println("Message processed.");
        return result;
    }
}
```

### Usage

```java
MessageService service = new LoggingDecorator(
        new BasicMessageService()
);

System.out.println(service.send("Hello"));
```

### Output

```text
Sending message...
Message processed.
Sent: Hello
```

The decorator adds logging without changing `BasicMessageService`.

---

## 5. What is the difference between Decorator and Inheritance?

{: .label .label-yellow }

**Answer**

| Decorator | Inheritance |
|---|---|
| Adds behavior by wrapping an object. | Adds or overrides behavior through a subclass. |
| Behavior can be combined at runtime. | Behavior combinations often require additional subclasses. |
| Uses composition. | Uses an `extends` relationship for class inheritance. |
| Can decorate a particular object instance. | Defines behavior at the class level. |
| Useful for optional, combinable features. | Useful for stable specialization and shared behavior. |

### Decorator example

```java
Coffee coffee = new CreamDecorator(
        new MilkDecorator(
                new SimpleCoffee()
        )
);
```

### Inheritance example

```java
class MilkCoffee extends SimpleCoffee {
    // Adds milk-related behavior
}

class CreamCoffee extends SimpleCoffee {
    // Adds cream-related behavior
}
```

If both milk and cream are needed, inheritance may require another class. Decorators allow the two behaviors to be combined without creating a dedicated combination class.

**Remember:** Decorator favors composition over creating subclasses for every feature combination.

---

## 6. What is the difference between Decorator and Adapter?

{: .label .label-yellow }

**Answer**

| Decorator | Adapter |
|---|---|
| Adds or modifies behavior. | Converts one interface into another. |
| Usually preserves the wrapped object's interface. | Exposes the interface expected by the client. |
| Focuses on extending functionality. | Focuses on compatibility. |
| Example: Adding logging to a service. | Example: Adapting a legacy payment gateway. |

**Remember:** Decorator enhances behavior; Adapter makes incompatible interfaces work together.

---

## 7. What is the difference between Decorator and Proxy?

{: .label .label-yellow }

**Answer**

Both patterns wrap an object and commonly implement the same interface, but their primary purposes differ.

| Decorator | Proxy |
|---|---|
| Adds responsibilities or behavior. | Controls access to an object. |
| Often supports combining multiple wrappers. | Often handles access checks, lazy initialization, or remote communication. |
| Focuses on extending functionality. | Focuses on controlling how an object is accessed. |
| Example: Adding compression and encryption to a data stream. | Example: A proxy that checks authorization before calling a service. |

**Remember:** Decorator adds behavior; Proxy controls access. Their structures can look very similar.

---

## 8. What are the advantages and disadvantages of the Decorator Pattern?

{: .label .label-blue }

**Answer**

### Advantages

- **Extensibility:** New behavior can be added without modifying existing classes.
- **Flexibility:** Decorators can be combined at runtime.
- **Single responsibility:** Each decorator can focus on one additional behavior.
- **Reduced subclassing:** Avoids creating classes for every combination of optional features.
- **Open/Closed Principle:** Existing components can be extended through new decorators.

### Disadvantages

- **More classes:** Each behavior may require its own decorator.
- **Debugging complexity:** Multiple wrappers can make execution flow harder to follow.
- **Order dependency:** Some decorators produce different results depending on their order.
- **Object identity:** The wrapped object and the outer decorator are different references.
- **Interface limitations:** A decorator implementing an interface does not automatically expose extra methods declared only on the concrete wrapped class.

---

## 9. Where is the Decorator Pattern used in real-world Java applications?

{: .label .label-green }

**Answer**

The Decorator Pattern is used extensively in Java's I/O APIs and in applications that need to combine optional processing steps.

### 9.1 Java I/O streams

Java provides a classic example of the Decorator Pattern.

```java
import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

public class Main {

    public static void main(String[] args) throws IOException {

        try (InputStream input = new BufferedInputStream(
                new FileInputStream("data.txt"))) {

            int value = input.read();

            System.out.println(value);
        }
    }
}
```

**How it works**

- `FileInputStream` reads bytes from a file.
- `BufferedInputStream` wraps the file stream.
- The buffered stream adds buffering to reduce frequent underlying I/O operations.
- Both are used through the `InputStream` abstraction.

Additional stream decorators, such as `DataInputStream`, can provide other capabilities.

### 9.2 Other common use cases

- **Logging:** Add logging around service calls.
- **Caching:** Cache results before delegating to an underlying service.
- **Compression:** Compress or decompress data streams.
- **Encryption:** Add encryption or decryption to data processing.
- **Authorization:** Wrap operations with permission checks, although access control is also commonly implemented using interceptors or proxies.
- **Metrics:** Measure execution time and count calls.

---

## 10. How is the Decorator Pattern used in Spring Boot?

{: .label .label-blue }

**Answer**

In Spring Boot, the same concept can be used to add logging, metrics, caching, or other behavior around an application-owned interface.

### Example: Adding logging to a service

```java
public interface EmployeeService {
    String getEmployeeName(long id);
}
```

### Concrete component

```java
import org.springframework.stereotype.Service;

@Service("basicEmployeeService")
public class BasicEmployeeService implements EmployeeService {

    @Override
    public String getEmployeeName(long id) {
        return "Danish";
    }
}
```

### Decorator

```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component("loggingEmployeeService")
public class LoggingEmployeeService implements EmployeeService {

    private final EmployeeService delegate;

    public LoggingEmployeeService(
            @Qualifier("basicEmployeeService")
            EmployeeService delegate) {
        this.delegate = delegate;
    }

    @Override
    public String getEmployeeName(long id) {
        System.out.println("Fetching employee: " + id);

        String name = delegate.getEmployeeName(id);

        System.out.println("Employee fetched successfully");

        return name;
    }
}
```

### Client service

```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

@Service
public class EmployeeReportService {

    private final EmployeeService employeeService;

    public EmployeeReportService(
            @Qualifier("loggingEmployeeService")
            EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    public String generateReport(long id) {
        return employeeService.getEmployeeName(id);
    }
}
```

**How it works**

1. Spring creates the basic service bean.
2. Spring injects that bean into the logging decorator.
3. The report service receives the decorated implementation.
4. Calls pass through the logging decorator before reaching the basic service.

**Important:** The qualifiers avoid ambiguity because both beans implement `EmployeeService`. This example illustrates explicit decoration through dependency injection; Spring AOP can provide similar cross-cutting behavior through proxies, but AOP is not identical to the classic Decorator Pattern.

---

## 11. How does the Decorator Pattern support SOLID principles?

{: .label .label-blue }

**Answer**

- **Single Responsibility Principle (SRP):** Each decorator can handle one additional responsibility, such as logging or compression.
- **Open/Closed Principle (OCP):** New decorators can extend behavior without modifying the original component.
- **Dependency Inversion Principle (DIP):** Both the decorator and the concrete component can depend on the same abstraction.

### Example

```java
Coffee coffee = new CreamDecorator(
        new MilkDecorator(
                new SimpleCoffee()
        )
);
```

Each class implements `Coffee`, allowing decorators to be combined through the abstraction rather than depending on specific concrete implementations.

**Note:** The pattern can support SOLID principles, but correct design still depends on the details of the implementation.

---

## 12. Can decorators be chained, and does their order matter?

{: .label .label-yellow }

**Answer**

Yes. Decorators can wrap one another, forming a chain of behavior.

```java
Coffee coffee = new CreamDecorator(
        new MilkDecorator(
                new SimpleCoffee()
        )
);
```

The call to `getCost()` travels through the outer decorator to the inner decorator and finally to the original coffee.

For this coffee example, the cost is additive, so reversing the milk and cream decorators produces the same total.

However, order can matter when decorators transform data or results.

For example, consider a system that compresses and encrypts data:

- Compressing and then encrypting produces encrypted compressed data.
- Encrypting and then compressing generally produces a different result and may not compress effectively.

**Remember:** Chaining is a major benefit of Decorator, but choose the order based on the behavior being added.

---

## 13. What are the SOLID and design considerations when using Decorator?

{: .label .label-blue }

**Answer**

Use decorators when additional behavior is optional, independently understandable, and composable.

### Best practices

- Program to an interface rather than a concrete implementation.
- Keep each decorator focused on one responsibility.
- Ensure each decorator preserves the expected contract of the component.
- Document any ordering requirements.
- Avoid excessive layers that make debugging difficult.
- Handle exceptions and resource cleanup correctly when wrapping I/O or service calls.

**Important:** A decorator should generally be substitutable for the component it wraps. A decorator that unexpectedly changes core behavior can violate the Liskov Substitution Principle.

---

## 14. Quick interview revision

{: .label .label-green }

| Question | Short answer |
|---|---|
| What is Decorator? | A structural pattern that adds behavior to an object by wrapping it. |
| What are its main components? | Component, concrete component, decorator, and concrete decorator. |
| Does it use inheritance or composition? | Primarily composition, with a shared interface. |
| Can decorators be added at runtime? | Yes. |
| Does it modify the original class? | No, not necessarily; it wraps the object. |
| Decorator vs Adapter? | Decorator adds behavior; Adapter converts interfaces. |
| Decorator vs Proxy? | Decorator extends behavior; Proxy controls access. |
| Decorator vs inheritance? | Decorator combines behavior through wrappers; inheritance defines subclass behavior. |
| Is Decorator a structural pattern? | Yes. |
| What is a real Java example? | `BufferedInputStream` wrapping `FileInputStream`. |
| Does decorator order matter? | Sometimes, depending on how behaviors interact. |
| Can decorators be chained? | Yes, one decorator can wrap another. |
| Is Spring AOP the same as Decorator? | No. Spring AOP commonly uses proxies to apply cross-cutting behavior. |

---

## 15. Final recommendation

{: .label .label-green }

Remember this simple rule:

**Use the Decorator Pattern when you need to add optional behavior to an object without modifying its original class or creating a subclass for every combination of features.**

For Java and Spring Boot projects:

1. Define a common interface for the component.
2. Create a decorator that implements the same interface.
3. Hold a reference to the wrapped component.
4. Delegate existing behavior and add the required functionality.
5. Chain decorators only when the resulting execution order is clear.

The key interview distinction is: **Decorator adds behavior, Adapter changes the interface, and Proxy controls access.**