---
title: Factory and Abstract Factory
parent: Design Patterns and LLD
nav_order: 4
---

# Factory and Abstract Factory
{: .no_toc }

Creational design patterns that encapsulate object creation and reduce coupling between clients and concrete implementations.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

1. TOC
{:toc}
</details>

---

## 1. What is the Factory Design Pattern?

{: .label .label-green }

**Answer**

The Factory Method is a creational design pattern that defines an interface for creating an object while allowing subclasses to decide which concrete class to instantiate.

In everyday Java development, the term *Simple Factory* is also commonly used for a class that selects and creates an object based on an input parameter. A Simple Factory is a useful idiom, but it is not the same as the formal Factory Method pattern.

### Why do we need a Factory?

Suppose an application supports Email and SMS notifications.

Without a factory, the client creates concrete implementations directly:

```java
Notification notification;

if (type.equals("email")) {
    notification = new EmailNotification();
} else {
    notification = new SmsNotification();
}

notification.send("Hello");
```

If new notification types are added, this creation logic can spread across multiple classes.

A factory encapsulates the creation logic so that clients can work with a common interface.

### Key components

- **Product:** The common interface implemented by the objects being created.
- **Concrete Products:** The actual implementations.
- **Factory:** Encapsulates object creation. In the formal Factory Method pattern, subclasses override a creation method.
- **Client:** Uses the product interface without depending on its concrete implementation.

---

## 2. How do you implement a Simple Factory in Java?

{: .label .label-green }

**Answer**

A Simple Factory centralizes object creation in one class or method. The client supplies a choice, and the factory returns the corresponding implementation.

### Complete Java example

```java
// Product interface
interface Notification {
    void send(String message);
}

// Concrete Product 1
class EmailNotification implements Notification {

    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

// Concrete Product 2
class SmsNotification implements Notification {

    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

// Simple Factory
class NotificationFactory {

    public static Notification create(String type) {
        if ("email".equalsIgnoreCase(type)) {
            return new EmailNotification();
        }

        if ("sms".equalsIgnoreCase(type)) {
            return new SmsNotification();
        }

        throw new IllegalArgumentException(
                "Unsupported notification type: " + type
        );
    }
}

// Client
public class Main {

    public static void main(String[] args) {

        Notification email =
                NotificationFactory.create("email");

        Notification sms =
                NotificationFactory.create("sms");

        email.send("Welcome!");
        sms.send("Your OTP is 123456");
    }
}
```

### Output

```text
Email: Welcome!
SMS: Your OTP is 123456
```

### How it works

1. The client calls `NotificationFactory.create("email")`.
2. The factory selects the appropriate implementation.
3. The factory returns an object typed as `Notification`.
4. The client calls `send()` without needing to know the concrete class.

**Important:** This example is a Simple Factory. It centralizes object creation, but adding a new product still requires modifying the factory's selection logic.

---

## 3. What is the Factory Method Design Pattern?

{: .label .label-green }

**Answer**

The Factory Method is a creational pattern that defines a method for creating a product but lets subclasses determine which concrete product to return.

Unlike a Simple Factory, the formal Factory Method pattern uses inheritance and polymorphism to delegate the creation decision to a subclass.

### Complete Java example

```java
// Product
interface Notification {
    void send(String message);
}

// Concrete Products
class EmailNotification implements Notification {

    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class SmsNotification implements Notification {

    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

// Creator
abstract class NotificationCreator {

    // Factory Method
    public abstract Notification createNotification();

    // Business logic uses the product interface
    public void notifyUser(String message) {
        Notification notification = createNotification();
        notification.send(message);
    }
}

// Concrete Creator 1
class EmailNotificationCreator extends NotificationCreator {

    @Override
    public Notification createNotification() {
        return new EmailNotification();
    }
}

// Concrete Creator 2
class SmsNotificationCreator extends NotificationCreator {

    @Override
    public Notification createNotification() {
        return new SmsNotification();
    }
}

// Client
public class Main {

    public static void main(String[] args) {

        NotificationCreator creator =
                new EmailNotificationCreator();

        creator.notifyUser("Welcome to our application!");

        creator = new SmsNotificationCreator();
        creator.notifyUser("Your verification code is 123456");
    }
}
```

### Output

```text
Email: Welcome to our application!
SMS: Your verification code is 123456
```

### How it works

1. The client selects a concrete creator.
2. The creator implements `createNotification()`.
3. `notifyUser()` calls the factory method to obtain a product.
4. The business logic uses the `Notification` interface.
5. Changing the creator changes the product created without changing the notification workflow.

**Remember:** Simple Factory uses centralized selection logic; Factory Method delegates creation to subclasses.

---

## 4. What is the Abstract Factory Design Pattern?

{: .label .label-green }

**Answer**

The Abstract Factory is a creational design pattern that provides an interface for creating families of related objects without specifying their concrete classes.

It is useful when multiple objects must work together as part of the same product family.

### Real-world example: UI components

Suppose an application supports Windows and Mac themes.

Each theme needs a matching set of components:

- A button.
- A checkbox.

A Windows theme should create Windows-style buttons and checkboxes. A Mac theme should create Mac-style buttons and checkboxes.

Creating these objects independently could accidentally mix components from different themes. Abstract Factory provides a way to create a consistent family.

### Key components

- **Abstract Factory:** Declares methods for creating each product type.
- **Concrete Factories:** Create a matching family of products.
- **Abstract Products:** Define interfaces for each product type.
- **Concrete Products:** Implement the product interfaces.
- **Client:** Uses the abstract factory and product interfaces.

### Complete Java example

```java
// Abstract Product A
interface Button {
    void render();
}

// Abstract Product B
interface Checkbox {
    void render();
}

// Concrete Products: Windows
class WindowsButton implements Button {

    @Override
    public void render() {
        System.out.println("Rendering Windows button");
    }
}

class WindowsCheckbox implements Checkbox {

    @Override
    public void render() {
        System.out.println("Rendering Windows checkbox");
    }
}

// Concrete Products: Mac
class MacButton implements Button {

    @Override
    public void render() {
        System.out.println("Rendering Mac button");
    }
}

class MacCheckbox implements Checkbox {

    @Override
    public void render() {
        System.out.println("Rendering Mac checkbox");
    }
}

// Abstract Factory
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete Factory: Windows
class WindowsFactory implements GUIFactory {

    @Override
    public Button createButton() {
        return new WindowsButton();
    }

    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

// Concrete Factory: Mac
class MacFactory implements GUIFactory {

    @Override
    public Button createButton() {
        return new MacButton();
    }

    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Client
class Application {

    private final Button button;
    private final Checkbox checkbox;

    public Application(GUIFactory factory) {
        this.button = factory.createButton();
        this.checkbox = factory.createCheckbox();
    }

    public void renderUI() {
        button.render();
        checkbox.render();
    }
}

// Main
public class Main {

    public static void main(String[] args) {

        GUIFactory factory = new WindowsFactory();

        Application app = new Application(factory);
        app.renderUI();
    }
}
```

### Output

```text
Rendering Windows button
Rendering Windows checkbox
```

To use the Mac family, change the factory:

```java
GUIFactory factory = new MacFactory();

Application app = new Application(factory);
app.renderUI();
```

The application code remains unchanged. The factory determines which family of components is created.

**Important:** Abstract Factory makes it easier to switch between product families. Adding a completely new product type, however, generally requires changing the abstract factory interface and its concrete implementations.

---

## 5. What is the difference between Simple Factory, Factory Method, and Abstract Factory?

{: .label .label-yellow }

**Answer**

| Feature | Simple Factory | Factory Method | Abstract Factory |
|---|---|---|---|
| Main purpose | Centralize object creation. | Delegate product creation to subclasses. | Create families of related products. |
| Typical structure | One factory class or method. | Creator hierarchy with an overridable creation method. | Factory interface with multiple creation methods. |
| Uses inheritance for creation? | Not necessarily. | Yes, typically. | Not necessarily for clients; concrete factories implement an interface. |
| Number of product types | Often one product hierarchy. | Usually one product hierarchy. | Multiple related product hierarchies. |
| Example | Select Email or SMS notification. | Subclasses choose a notification implementation. | Create matching buttons and checkboxes for a theme. |

### Quick example

**Simple Factory**

```java
NotificationFactory.create("email");
```

**Factory Method**

```java
NotificationCreator creator =
        new EmailNotificationCreator();

creator.createNotification();
```

**Abstract Factory**

```java
GUIFactory factory = new WindowsFactory();

Button button = factory.createButton();
Checkbox checkbox = factory.createCheckbox();
```

**Interview tip:** If the requirement is to create one product based on a choice, a Simple Factory may be enough. If subclasses decide which product to create, consider Factory Method. If several related products must be created consistently, consider Abstract Factory.

---

## 6. What is the difference between Factory Method and Abstract Factory?

{: .label .label-yellow }

**Answer**

Both patterns encapsulate object creation, but they differ in the scope of what they create and how creation is organized.

| Factory Method | Abstract Factory |
|---|---|
| Focuses on creating a product through an overridable method. | Focuses on creating a family of related products. |
| Commonly uses inheritance and overriding. | Commonly uses composition by passing a factory object to the client. |
| Typically has one main factory method for a product type. | Usually has multiple factory methods for related product types. |
| Example: A subclass creates an Email or SMS notification. | Example: A theme factory creates matching buttons and checkboxes. |

**Remember:** Factory Method delegates a creation decision; Abstract Factory groups related creation operations behind one interface.

---

## 7. What are the advantages and disadvantages of Factory patterns?

{: .label .label-blue }

**Answer**

### Advantages

- **Loose coupling:** Clients depend on product interfaces rather than concrete classes.
- **Centralized creation:** Object construction is kept out of business logic.
- **Extensibility:** New implementations can often be introduced without changing client code.
- **Consistency:** Abstract Factory helps ensure that related objects come from the same family.
- **Testability:** Interfaces make it easier to inject mock or test implementations.

### Disadvantages

- **More classes:** Concrete products and factories add structure and boilerplate.
- **Added complexity:** A factory can be unnecessary for straightforward object construction.
- **Simple Factory maintenance:** Conditional selection logic must change when new product types are introduced.
- **Abstract Factory rigidity:** Adding a new product type may require updating every factory implementation.
- **Configuration:** The application must still decide which factory or implementation to use.

**Best practice:** Introduce a factory when it simplifies object creation or reduces coupling, rather than using one for every class.

---

## 8. Where are Factory patterns used in real-world applications?

{: .label .label-green }

**Answer**

Factory patterns are useful when the concrete implementation depends on configuration, environment, user selection, or a provider.

### Common use cases

- **Notification systems:** Create Email, SMS, or push notification senders.
- **Payment systems:** Create adapters for different payment providers.
- **Database integrations:** Select provider-specific implementations behind a common abstraction.
- **Cloud applications:** Create storage clients for different providers or environments.
- **UI frameworks:** Create matching controls for different platforms or themes.
- **Document processing:** Create parsers for JSON, XML, CSV, or other formats.
- **Testing:** Supply test doubles or mock implementations instead of production dependencies.

### Example: Document parser factory

```java
interface DocumentParser {
    void parse(String content);
}

class JsonParser implements DocumentParser {

    @Override
    public void parse(String content) {
        System.out.println("Parsing JSON");
    }
}

class XmlParser implements DocumentParser {

    @Override
    public void parse(String content) {
        System.out.println("Parsing XML");
    }
}

class ParserFactory {

    public static DocumentParser create(String type) {
        if ("json".equalsIgnoreCase(type)) {
            return new JsonParser();
        }

        if ("xml".equalsIgnoreCase(type)) {
            return new XmlParser();
        }

        throw new IllegalArgumentException(
                "Unsupported format: " + type
        );
    }
}
```

**Usage**

```java
DocumentParser parser = ParserFactory.create("json");
parser.parse("{\"name\":\"Danish\"}");
```

**Output**

```text
Parsing JSON
```

The client uses `DocumentParser` and does not need to know which concrete parser was created.

---

## 9. How are Factory patterns used in Spring Boot?

{: .label .label-blue }

**Answer**

Spring provides dependency injection and bean management, so you do not always need to write a traditional factory class. However, factory methods are useful when creating objects dynamically or encapsulating configuration-dependent creation logic.

### 9.1 Using `@Bean` as a factory method

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class NotificationConfig {

    @Bean
    public Notification notification() {
        return new EmailNotification();
    }
}
```

Spring calls the `@Bean` method to obtain the bean managed by the container.

A service can inject the abstraction:

```java
import org.springframework.stereotype.Service;

@Service
public class NotificationService {

    private final Notification notification;

    public NotificationService(Notification notification) {
        this.notification = notification;
    }

    public void send(String message) {
        notification.send(message);
    }
}
```

**Note:** A Spring `@Bean` method is a factory method in the general sense, but it does not automatically implement the formal Factory Method design pattern.

### 9.2 Using a factory to select an implementation

When the application must select among multiple implementations at runtime, a dedicated factory can be useful.

```java
import org.springframework.stereotype.Component;
import java.util.Map;

@Component
public class NotificationFactory {

    private final Map<String, Notification> notifications;

    public NotificationFactory(
            Map<String, Notification> notifications) {
        this.notifications = notifications;
    }

    public Notification getNotification(String type) {
        Notification notification =
                notifications.get(type.toLowerCase());

        if (notification == null) {
            throw new IllegalArgumentException(
                    "Unsupported notification type: " + type
            );
        }

        return notification;
    }
}
```

Register implementations with matching bean names:

```java
import org.springframework.stereotype.Component;

@Component("email")
class EmailNotification implements Notification {

    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

@Component("sms")
class SmsNotification implements Notification {

    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}
```

A service can then request the required implementation:

```java
Notification notification =
        notificationFactory.getNotification("email");

notification.send("Welcome");
```

**Important:** Spring injects beans by their names in this example. Normalize the selection key consistently, validate unsupported types, and consider whether runtime selection is genuinely required. This example is a Simple Factory backed by Spring-managed beans.

---

## 10. How do Factory patterns support SOLID principles?

{: .label .label-blue }

**Answer**

Factory patterns can help implement SOLID principles by separating object creation from business logic.

- **Single Responsibility Principle (SRP):** The factory handles creation, while the client handles business operations.
- **Open/Closed Principle (OCP):** Factory Method and Abstract Factory can allow new implementations to be added without modifying client logic. A Simple Factory with conditionals may require modification when a new product is introduced.
- **Dependency Inversion Principle (DIP):** Clients depend on product interfaces rather than concrete implementations.

### Example

```java
Notification notification =
        notificationFactory.getNotification("email");

notification.send("Welcome");
```

The client works with `Notification`, not directly with `EmailNotification` or `SmsNotification`.

**Remember:** Factories can reduce coupling, but the design must still be structured appropriately to satisfy SOLID principles.

---

## 11. Can Abstract Factory use Dependency Injection?

{: .label .label-green }

**Answer**

Yes. Dependency injection is a natural way to supply an Abstract Factory to a client.

```java
public class Application {

    private final GUIFactory factory;

    public Application(GUIFactory factory) {
        this.factory = factory;
    }

    public void render() {
        Button button = factory.createButton();
        Checkbox checkbox = factory.createCheckbox();

        button.render();
        checkbox.render();
    }
}
```

The application does not construct `WindowsFactory` or `MacFactory` itself. The desired implementation is injected from outside.

For example, Spring can register one concrete factory as a bean and inject it into `Application`.

**Important:** If multiple factories are registered, use configuration, `@Qualifier`, or another explicit selection mechanism to choose the intended family.

---

## 12. What is the difference between Factory and Builder?

{: .label .label-yellow }

**Answer**

| Factory | Builder |
|---|---|
| Encapsulates which object or implementation to create. | Encapsulates how to construct a complex object step by step. |
| Often selects among concrete implementations. | Often configures fields and optional parameters. |
| Example: Create an Email or SMS notification. | Example: Build an HTTP request with URI, headers, and timeout. |

**Remember:** Factory focuses on object selection and creation; Builder focuses on controlled, step-by-step construction.

---

## 13. Quick interview revision

{: .label .label-green }

| Question | Short answer |
|---|---|
| What is a Factory pattern? | A creational approach that encapsulates object creation. |
| What is a Simple Factory? | A class or method that selects and creates a product. |
| What is Factory Method? | A pattern in which subclasses determine which product a creation method returns. |
| What is Abstract Factory? | A pattern that creates families of related products. |
| Factory Method vs Abstract Factory? | One focuses on a product creation method; the other creates a related product family. |
| Is Simple Factory a formal GoF pattern? | No. It is a common idiom, not one of the original GoF patterns. |
| Is Factory a creational pattern? | Yes. Factory Method and Abstract Factory are creational patterns. |
| Does Spring use factory methods? | Yes. For example, methods annotated with `@Bean` create Spring-managed beans. |
| Can Factory improve testability? | Yes, when clients depend on abstractions and implementations can be substituted. |
| What is a disadvantage of Simple Factory? | Conditional selection logic may need modification for each new product type. |
| What is a disadvantage of Abstract Factory? | Adding a new product type can require changing every concrete factory. |
| Factory vs Builder? | Factory selects or creates an implementation; Builder configures an object's construction. |

---

## 14. Final recommendation

{: .label .label-green }

Remember these three rules:

1. **Simple Factory:** Use it when you want to centralize object creation and select an implementation based on a condition.
2. **Factory Method:** Use it when subclasses should decide which product to create.
3. **Abstract Factory:** Use it when you need to create families of related objects that should work together.

For Java and Spring Boot applications, start with dependency injection and simple interfaces. Introduce a dedicated factory when creation or runtime selection logic becomes complex enough to justify it.

The key interview distinction is: **Factory Method delegates the creation decision; Abstract Factory creates a consistent family of related products.**