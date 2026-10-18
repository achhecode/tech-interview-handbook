---
title: IoC and Dependency Injection
parent: Spring Framework
nav_order: 1
---

# IoC and Dependency Injection
{: .no_toc }

Inversion of Control and the ways Spring injects dependencies.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

# Dependency Injection in Java and Spring Boot

## 1. What Is Dependency Injection?

**Dependency Injection (DI)** is a design technique in which an object receives its dependencies from an external source instead of creating them itself.

A dependency is an object that another class needs to perform its work.

For example, if `OrderService` requires `PaymentService`, then `PaymentService` is a dependency of `OrderService`.

Without DI, the service creates its own dependency. With DI, the dependency is supplied from outside.

### Without Dependency Injection

```java
class PaymentService {
    public void pay() {
        System.out.println("Payment processed");
    }
}

class OrderService {
    private final PaymentService paymentService =
            new PaymentService();

    public void placeOrder() {
        paymentService.pay();
    }
}
```

**Problem:** `OrderService` is tightly coupled to the concrete `PaymentService` implementation. Replacing it with a mock or another implementation requires changing how `OrderService` creates its dependency.

### With Dependency Injection

```java
class PaymentService {
    public void pay() {
        System.out.println("Payment processed");
    }
}

class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {
        paymentService.pay();
    }
}

public class Main {
    public static void main(String[] args) {
        PaymentService paymentService = new PaymentService();
        OrderService orderService = new OrderService(paymentService);

        orderService.placeOrder();
    }
}
```

**Output:**

```text
Payment processed
```

Here, `OrderService` does not create `PaymentService`. It receives it through its constructor.

**Key idea:** DI separates the responsibility of creating dependencies from the responsibility of using them.

---

## 2. What Is the Difference Between Dependency Injection and Inversion of Control?

These concepts are related, but they are not identical.

| Concept | Meaning |
|---|---|
| Inversion of Control (IoC) | A general principle in which control over object creation or program flow is delegated to another component or framework |
| Dependency Injection (DI) | A technique for implementing IoC by supplying dependencies from outside an object |
| IoC container | A framework component that creates, configures, and manages objects and their dependencies |

In Spring Boot, the Spring IoC container manages Spring beans and injects their dependencies.

For example, when Spring creates an `OrderService` bean, it can supply a `PaymentService` bean through the constructor.

---

## 3. What Are the Types of Dependency Injection?

The commonly discussed types are:

1. Constructor injection
2. Setter injection
3. Field injection
4. Method injection

### 3.1 Constructor Injection

Dependencies are provided through the class constructor.

```java
class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**Advantages:**
- Makes mandatory dependencies explicit.
- Ensures required dependencies are provided when the object is constructed.
- Supports immutable fields using `final`.
- Makes unit testing straightforward.
- Helps prevent partially initialized objects.

**Best suited for:** Mandatory dependencies.

### 3.2 Setter Injection

Dependencies are provided through setter methods after the object is created.

```java
class OrderService {
    private PaymentService paymentService;

    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Usage:

```java
OrderService orderService = new OrderService();
orderService.setPaymentService(new PaymentService());
```

**Advantages:**
- Supports optional dependencies.
- Allows a dependency to be replaced after construction.
- Useful when configuration must be supplied after object creation.

**Considerations:** The object may be temporarily incomplete, and changing a dependency at runtime can introduce state-management issues.

**Best suited for:** Optional dependencies that can reasonably be configured after construction.

### 3.3 Field Injection

Dependencies are injected directly into fields, commonly using Spring's `@Autowired`.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

**Advantages:**
- Requires little boilerplate.
- Is concise to write.

**Disadvantages:**
- Dependencies are less visible in the constructor or public API.
- Plain unit tests cannot easily supply dependencies without reflection or Spring test support.
- Fields cannot be declared `final` for normal field injection.
- It is harder to construct a valid object outside the Spring container.

**Best practice:** Generally avoid field injection in production service classes when constructor injection is suitable.

### 3.4 Method Injection

Dependencies are supplied as method parameters when a method is called.

```java
class OrderService {

    public void placeOrder(PaymentService paymentService) {
        paymentService.pay();
    }
}
```

Usage:

```java
OrderService orderService = new OrderService();
orderService.placeOrder(new PaymentService());
```

This example demonstrates passing a dependency to a method. It is not the same as Spring's specialized *lookup method injection*, which lets the container override a method to retrieve a bean.

**Best suited for:** Operations that need a collaborator only for a particular method call, or for passing changing inputs and collaborators explicitly.

---

## 4. Which Type of Dependency Injection Do You Prefer?

**Constructor injection is generally preferred for mandatory dependencies in Spring Boot applications.**

Reasons:

1. Dependencies are clearly declared.
2. Required dependencies must be supplied at construction time.
3. Fields can be `final`.
4. Unit tests can pass mocks or fake implementations directly.
5. The class is less dependent on the Spring container.
6. Missing dependencies are easier to identify.

Setter injection is appropriate for optional dependencies in designs that genuinely need post-construction configuration.

Field injection may be concise, but it makes dependencies less explicit and testing more cumbersome.

### Comparison Table

| Feature | Constructor | Setter | Field |
|---|---|---|---|
| Dependencies explicit | Yes | Through setters | No |
| Supports `final` dependency fields | Yes | Generally no | No |
| Mandatory dependencies | Excellent fit | Possible, but less enforced | Possible through framework |
| Optional dependencies | Possible, but less convenient | Good fit | Possible |
| Plain unit testing | Easy | Easy | More difficult |
| Preferred for Spring services | Yes | For suitable optional dependencies | Generally avoid |

---

## 5. Dependency Injection in Spring Boot

Spring Boot uses the Spring IoC container to create and manage beans.

Common annotations include:

| Annotation | Purpose |
|---|---|
| `@Component` | Marks a class as a Spring-managed component |
| `@Service` | Marks a service-layer component |
| `@Repository` | Marks a persistence-layer component and enables exception translation for supported repository infrastructure |
| `@Controller` | Marks an MVC controller |
| `@RestController` | Marks a REST controller |
| `@Autowired` | Requests dependency injection |
| `@Configuration` | Declares configuration classes |
| `@Bean` | Declares a bean from a configuration method |

### Example: Constructor Injection in Spring Boot

**PaymentService.java**

```java
import org.springframework.stereotype.Service;

@Service
public class PaymentService {

    public void pay() {
        System.out.println("Payment processed");
    }
}
```

**OrderService.java**

```java
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {
        System.out.println("Order placed");
        paymentService.pay();
    }
}
```

**OrderController.java**

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping("/orders")
    public String placeOrder() {
        orderService.placeOrder();
        return "Order processed successfully";
    }
}
```

Spring discovers the annotated components through component scanning, creates their beans, and supplies the dependencies through the constructors.

In Spring, if a class has only one constructor, `@Autowired` is not required on that constructor.

---

## 6. Dependency Injection Using Interfaces

Injecting an interface rather than a concrete implementation can reduce coupling further.

Suppose an application supports multiple payment providers.

**PaymentProcessor.java**

```java
public interface PaymentProcessor {
    void pay();
}
```

**CardPaymentProcessor.java**

```java
import org.springframework.stereotype.Component;

@Component
public class CardPaymentProcessor implements PaymentProcessor {

    @Override
    public void pay() {
        System.out.println("Card payment processed");
    }
}
```

**OrderService.java**

```java
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    private final PaymentProcessor paymentProcessor;

    public OrderService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }

    public void placeOrder() {
        paymentProcessor.pay();
    }
}
```

`OrderService` depends on the `PaymentProcessor` abstraction instead of `CardPaymentProcessor`.

This makes it easier to replace the implementation with another provider or a test double.

**Important:** If Spring finds multiple beans implementing `PaymentProcessor`, it needs an unambiguous choice. You can use `@Primary`, `@Qualifier`, or another explicit bean-selection strategy.

For example:

```java
public OrderService(
        @org.springframework.beans.factory.annotation.Qualifier("cardPaymentProcessor")
        PaymentProcessor paymentProcessor) {
    this.paymentProcessor = paymentProcessor;
}
```

The qualifier value must match the actual bean name.

---

## 7. What Are the Benefits of Dependency Injection?

### 7.1 Loose Coupling

Classes depend on abstractions or supplied collaborators instead of creating concrete dependencies themselves.

### 7.2 Easier Unit Testing

A real dependency can be replaced with a mock or fake implementation.

```java
class FakePaymentProcessor implements PaymentProcessor {
    @Override
    public void pay() {
        System.out.println("Fake payment");
    }
}
```

A test can supply this implementation to `OrderService` without connecting to a real payment provider.

### 7.3 Better Maintainability

Changing a dependency implementation does not necessarily require changing the class that consumes it.

### 7.4 Reusability

The same service can be used with different compatible dependencies in different environments.

### 7.5 Centralized Object Management

A framework such as Spring can manage object creation, dependency wiring, and bean lifecycle callbacks.

### 7.6 Easier Extension

Additional implementations can be introduced behind an interface with fewer changes to existing consumer classes.

**Note:** DI does not automatically guarantee good architecture or thread safety. Classes must still have well-designed responsibilities and safe state management.

---

## 8. What Happens If a Dependency Is Missing?

With constructor injection, if a required Spring bean cannot be found or Spring cannot resolve which bean to inject, application context initialization typically fails.

For example, if `OrderService` requires a `PaymentProcessor` but no eligible bean exists, Spring reports an error such as `NoSuchBeanDefinitionException`, often wrapped in a bean-creation exception.

This is useful because a required dependency problem is detected during application startup instead of appearing later when the method is called.

If multiple beans match and no selection rule resolves the ambiguity, Spring reports a dependency-resolution error.

---

## 9. Common Dependency Injection Interview Questions

### Q1. What is dependency injection?

Dependency injection is a technique in which an object receives its dependencies from an external source rather than creating them itself.

### Q2. What is the difference between DI and IoC?

IoC is the broader principle of delegating control to another component or framework. DI is one way to implement IoC by supplying dependencies externally.

### Q3. Which type of DI is preferred in Spring Boot?

Constructor injection is generally preferred for mandatory dependencies because it makes dependencies explicit, supports `final` fields, and simplifies unit testing.

### Q4. Why is field injection discouraged?

Field injection hides dependencies, makes plain unit testing harder, and prevents the injected fields from being declared `final`.

### Q5. Can Spring inject dependencies without `@Autowired`?

Yes. For a class with a single constructor, Spring can use that constructor without `@Autowired`.

### Q6. What happens when two beans implement the same interface?

Spring needs a clear choice. Use `@Qualifier`, `@Primary`, or another explicit bean configuration to resolve the ambiguity.

### Q7. Does DI mean the application must use Spring?

No. DI can be implemented manually using constructors, setters, or method parameters. Spring simply automates dependency resolution and object management.

### Q8. How does DI improve testing?

Tests can supply mock, stub, or fake dependencies instead of invoking real databases, external APIs, or other infrastructure.

### Q9. What is circular dependency?

A circular dependency occurs when two or more components depend on one another, directly or indirectly. Constructor-injected circular dependencies are generally rejected by Spring because the required objects cannot be constructed in sequence. The design often needs refactoring.

### Q10. Does `@Autowired` create an object?

Not by itself. It marks an injection point. Spring's container resolves the dependency and supplies the corresponding bean.

---

## 10. Quick Revision

- **DI:** Supply dependencies externally.
- **IoC:** Delegate control to a framework or another component.
- **Constructor injection:** Best default for mandatory dependencies.
- **Setter injection:** Useful for optional dependencies.
- **Field injection:** Concise but less explicit and harder to test.
- **Method injection:** Supply a collaborator to a specific method call.
- **Spring IoC container:** Creates, configures, and manages beans.
- **`@Qualifier`:** Selects a particular bean when multiple candidates exist.
- **`@Primary`:** Marks a preferred bean among multiple candidates.
- **Main benefit:** Loose coupling, testability, and easier maintenance.

## One-Line Interview Answer

**Dependency Injection is a design technique in which a class receives its dependencies from outside rather than creating them itself; in Spring Boot, constructor injection is generally preferred for mandatory dependencies because it improves loose coupling, testability, and maintainability.**


---



---
title: "Spring IoC Container Internals: BeanFactory vs ApplicationContext and Bean Lifecycle"
parent: "Core Java"
nav_order: 71
---

# Spring IoC Container Internals

The Spring IoC container is responsible for creating, configuring, injecting dependencies into, and managing the lifecycle of Spring beans.

For Java and Spring Boot interviews, understand three things:

1. How the IoC container creates and wires beans.
2. The difference between `BeanFactory` and `ApplicationContext`.
3. How application contexts and bean definitions relate to each other.

## 1. What Is Inversion of Control (IoC)?

**Inversion of Control (IoC)** is a principle in which control over object creation and dependency management is delegated from application code to a container or framework.

Without IoC, a class commonly creates its own dependencies.

```java
class OrderService {
    private final PaymentService paymentService =
            new PaymentService();
}
```

Here, `OrderService` controls the creation of `PaymentService`.

With IoC and dependency injection, Spring creates and supplies the dependency.

```java
@Service
class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring manages the creation and wiring of these objects. `OrderService` focuses on business logic instead of dependency construction.

**Key distinction:** IoC is the broader principle; dependency injection is a technique Spring uses to implement it.

## 2. How Does IoC Work Inside the Spring Container?

The Spring container uses bean definitions, configuration metadata, and dependency-resolution rules to determine which objects to create and how to connect them.

<box gap={1} padding={3} border radius="lg">
  <box background="surface-secondary" padding={3} radius="md" align="center">
    **1. Application configuration**

    <text size="sm" textAlign="center">`@Component`, `@Service`, `@Configuration`, `@Bean`, properties</text>
  </box>
  <box align="center">
    <icon name="arrow-down" size="lg" />
  </box>
  <box background="surface" border padding={3} radius="md" align="center">
    **2. Bean definitions**

    <text size="sm" textAlign="center">Spring discovers component classes and registers bean metadata</text>
  </box>
  <box align="center">
    <icon name="arrow-down" size="lg" />
  </box>
  <box background="surface" border padding={3} radius="md" align="center">
    **3. Bean creation and dependency resolution**

    <text size="sm" textAlign="center">Spring instantiates objects and resolves constructor or other dependencies</text>
  </box>
  <box align="center">
    <icon name="arrow-down" size="lg" />
  </box>
  <box background="surface" border padding={3} radius="md" align="center">
    **4. Bean initialization**

    <text size="sm" textAlign="center">Dependency injection, aware callbacks, bean post-processors, initialization callbacks</text>
  </box>
  <box align="center">
    <icon name="arrow-down" size="lg" />
  </box>
  <box background="surface-secondary" padding={3} radius="md" align="center">
    **5. Ready-to-use beans**

    <text size="sm" textAlign="center">Beans are available for injection and application use</text>
  </box>
</box>

This is a simplified lifecycle. The exact steps depend on bean scope, configuration, post-processors, proxies, and other container features.

### Step 1: Configuration and Component Scanning

Spring discovers component classes and configuration metadata.

```java
@Service
public class PaymentService {
    public void pay() {
        System.out.println("Payment processed");
    }
}
```

The `@Service` annotation makes this class a candidate for component scanning when its package is included in the scan.

### Step 2: Bean Definition Registration

Spring registers metadata describing how to create and configure the bean. A bean definition can contain information such as the bean's class, scope, constructor arguments, and properties.

A bean definition is **metadata about a bean**, not the bean instance itself.

### Step 3: Bean Instantiation and Dependency Injection

When Spring creates `OrderService`, it resolves the required `PaymentService` dependency and supplies it through the constructor.

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {
        paymentService.pay();
    }
}
```

Spring generally creates singleton beans eagerly during application-context initialization. Other scopes and lazy initialization can change when creation occurs.

### Step 4: Bean Initialization

Spring can perform additional processing after instantiation and dependency population, including:

- `Aware` callbacks where applicable.
- `BeanPostProcessor` processing.
- `@PostConstruct` initialization callbacks.
- `InitializingBean.afterPropertiesSet()`.
- Custom initialization methods.

The exact order depends on the lifecycle callback and the type of processing involved.

### Step 5: Bean Use and Destruction

Once initialized, a bean can be injected into other beans and used by the application.

For managed singleton beans, Spring can invoke destruction callbacks when the application context closes. Prototype-scoped beans have different destruction-management behavior.

---

## 3. What Are BeanFactory and ApplicationContext?

Both `BeanFactory` and `ApplicationContext` provide access to Spring-managed beans.

`BeanFactory` is the basic IoC container interface. `ApplicationContext` extends `BeanFactory` and adds more application-oriented capabilities.

### BeanFactory

`BeanFactory` provides fundamental dependency-injection and bean-management functionality.

Typical responsibilities include:
- Retrieving beans.
- Managing bean definitions and dependencies.
- Creating beans when required by the configured factory behavior.

### ApplicationContext

`ApplicationContext` provides the core `BeanFactory` functionality plus additional features, including:

- Application event publication and listener support.
- Internationalization through message resolution.
- Integration with Spring's resource-loading abstractions.
- Automatic registration and use of many container post-processors.
- Integration with additional application and web infrastructure.

### Comparison Table

| Feature | `BeanFactory` | `ApplicationContext` |
|---|---|---|
| Basic dependency injection | Yes | Yes |
| Bean retrieval | Yes | Yes |
| Application event publishing | Not provided by the basic interface | Yes |
| Internationalization support | Not provided by the basic interface | Yes |
| Resource-loading integration | Basic container functionality | Additional resource abstraction |
| Typical default singleton initialization | Can be lazy depending on usage/configuration | Usually eager for non-lazy singleton beans |
| Common usage | Basic or specialized container needs | Most Spring and Spring Boot applications |

**Interview answer:** `ApplicationContext` is generally preferred for regular Spring applications because it provides additional infrastructure beyond the basic `BeanFactory` capabilities.

Do not say that `BeanFactory` cannot support advanced behavior in any configuration. Spring's capabilities also depend on the factory implementation and registered extensions.

---

## 4. How Does ApplicationContext Work in Spring Boot?

Spring Boot commonly creates an application context when an application starts.

```java
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

At a high level, Spring Boot:

1. Creates the appropriate application context for the application type.
2. Loads configuration and processes component scanning.
3. Registers bean definitions and configuration-derived beans.
4. Refreshes the context, creating and initializing applicable beans.
5. Makes the initialized context available to the running application.

For a typical Spring Boot web application, the context is commonly a web-specific application context. The exact implementation depends on the application type and configuration.

### How Does One Bean Get Another Bean?

Consider this dependency chain:

```text
OrderController
      |
      v
OrderService
      |
      v
PaymentService
```

Spring resolves each constructor dependency and supplies the appropriate bean.

For example:

```java
@RestController
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
}
```

The controller does not need to call `new OrderService()`. The container supplies the dependency.

---

## 5. What Is a Bean Context? Is It Different from ApplicationContext?

The phrase **"bean context" is not a standard Spring container interface equivalent to `ApplicationContext`**. It is important to clarify what someone means by it.

A Spring bean is an object managed by the container. The application context is the container that manages bean definitions, dependency wiring, and lifecycle behavior.

In applications with multiple modules, Spring can also use parent-child application contexts.

### Parent and Child Application Contexts

A parent-child arrangement has two distinct application contexts:

```text
Parent ApplicationContext
    |
    |-- Shared infrastructure beans
    |-- Common service beans
    |
    +---- Child ApplicationContext
              |
              |-- Module-specific beans
              |-- Web/controller beans
```

A child context can generally resolve beans from its parent. The parent does not automatically resolve beans defined only in the child.

This can be useful in certain web or modular applications, but **a separate child context is not automatically created for every bean or every package**.

### Important Differences

| Term | Meaning |
|---|---|
| Bean | An object managed by Spring |
| Bean definition | Metadata used to create and configure a bean |
| BeanFactory | Basic Spring container interface |
| ApplicationContext | Extended container interface with application-level capabilities |
| Parent context | An application context whose beans may be visible to a child context |
| Child context | An application context that can resolve beans from its parent as well as its own definitions |

---

## 6. What Happens If Spring Finds Multiple Beans of the Same Type?

Suppose two beans implement the same interface:

```java
public interface PaymentProcessor {
    void pay();
}
```

```java
@Component
class CardPaymentProcessor implements PaymentProcessor {
    public void pay() {
        System.out.println("Card payment");
    }
}
```

```java
@Component
class UpiPaymentProcessor implements PaymentProcessor {
    public void pay() {
        System.out.println("UPI payment");
    }
}
```

If a service requires `PaymentProcessor`, Spring may find two candidates and report an ambiguity unless a selection rule resolves it.

Use `@Qualifier` to select a particular bean:

```java
@Service
class OrderService {

    private final PaymentProcessor paymentProcessor;

    OrderService(
            @Qualifier("upiPaymentProcessor")
            PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
}
```

Alternatively, `@Primary` can mark a preferred candidate when no more specific selection overrides it.

This is an example of the container resolving dependencies from registered metadata rather than application code manually constructing every collaborator.

---

## 7. Common Interview Questions

### Q1. What is the Spring IoC container?

It is the Spring component responsible for instantiating, configuring, wiring, and managing the lifecycle of beans.

### Q2. What happens internally when Spring Boot starts?

Spring Boot prepares an application context, processes configuration and bean definitions, resolves dependencies, initializes applicable beans, and makes the context available to the application.

### Q3. What is the difference between a bean and a bean definition?

A bean is the managed object. A bean definition is the metadata that describes how Spring should create and configure it.

### Q4. What is the difference between BeanFactory and ApplicationContext?

`BeanFactory` provides fundamental bean management. `ApplicationContext` extends it with application-level capabilities such as events, message resolution, and resource loading.

### Q5. Does Spring create all beans at startup?

Not necessarily. Non-lazy singleton beans are generally created eagerly during context initialization. Lazy beans, prototype beans, and other scopes can follow different creation behavior.

### Q6. What is dependency resolution?

Dependency resolution is the process of finding a suitable bean for an injection point, considering type, qualifiers, primary status, and other configuration.

### Q7. What is the role of BeanPostProcessor?

A `BeanPostProcessor` can process bean instances before and after initialization callbacks. Spring uses post-processors for features such as annotation processing and proxy creation.

### Q8. Does Spring always create a proxy for every bean?

No. Proxies are created when required by features such as certain AOP advice, transactions, or method security. Many beans are ordinary objects without proxies.

### Q9. Can an application have multiple ApplicationContexts?

Yes. Parent-child contexts or multiple independent contexts can exist, depending on the application architecture.

### Q10. Why is constructor injection useful with the IoC container?

It makes required dependencies explicit and allows Spring to supply them during construction. It also makes unit testing easier and supports `final` dependency fields.

---

## 8. Quick Revision

- **IoC:** Delegates object creation and dependency management to a container.
- **DI:** Supplies an object's dependencies from outside.
- **Bean definition:** Metadata describing a managed bean.
- **BeanFactory:** Basic IoC container interface.
- **ApplicationContext:** Extended container interface for application-level features.
- **Spring Boot startup:** Creates and refreshes an application context.
- **Dependency resolution:** Selects the appropriate bean for an injection point.
- **BeanPostProcessor:** Processes bean instances around initialization.
- **Parent-child contexts:** A child can generally access parent beans; a parent cannot automatically access child-only beans.

## One-Line Interview Answer

**The Spring IoC container implements inversion of control by reading bean configuration, registering bean definitions, creating objects, resolving and injecting dependencies, applying initialization and post-processing, and managing bean lifecycle callbacks; `ApplicationContext` builds on `BeanFactory` with additional application-level features.**


---


---
title: "IoC vs Dependency Injection (DI) in Spring Framework"
parent: "Core Java"
nav_order: 72
---

# What Is the Difference Between IoC and DI?

**Inversion of Control (IoC)** is a design principle in which control over object creation and management is delegated to an external container or framework.

**Dependency Injection (DI)** is a technique used to implement IoC by providing an object's dependencies from outside instead of letting the object create them itself.

In Spring Framework, the Spring IoC container manages beans and uses dependency injection to connect them.

## 1. IoC vs DI: Key Differences

| Feature | Inversion of Control (IoC) | Dependency Injection (DI) |
|---|---|---|
| Definition | A design principle | A technique or implementation pattern |
| Purpose | Transfers control to a framework or container | Supplies dependencies to an object |
| Scope | Broader concept | A way to implement IoC |
| Responsibility | Container controls object creation and lifecycle | Dependencies are provided externally |
| Spring example | Spring IoC container | Constructor injection using Spring |
| Relationship | The overall principle | One way to achieve IoC |

## 2. Example Without IoC and DI

```java
class PaymentService {
    public void pay() {
        System.out.println("Payment processed");
    }
}

class OrderService {
    private final PaymentService paymentService =
            new PaymentService();

    public void placeOrder() {
        paymentService.pay();
    }
}
```

Here, `OrderService` creates its own `PaymentService` dependency. This tightly couples the two classes and makes replacing the dependency more difficult.

## 3. Example Using DI in Spring

```java
import org.springframework.stereotype.Service;

@Service
class PaymentService {
    public void pay() {
        System.out.println("Payment processed");
    }
}

@Service
class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {
        paymentService.pay();
    }
}
```

Spring creates and manages these beans and supplies `PaymentService` to `OrderService` through constructor injection.

- **IoC:** Spring takes responsibility for managing object creation and wiring.
- **DI:** Spring supplies `PaymentService` to the `OrderService` constructor.

## 4. How They Work Together

1. Spring discovers bean definitions through configuration and component scanning.
2. The IoC container creates and manages the required beans.
3. DI resolves and supplies dependencies to those beans.
4. The initialized beans are available for application use.

## 5. Benefits

- **Loose coupling:** Classes do not need to create their own dependencies.
- **Testability:** Dependencies can be replaced with mocks or test implementations.
- **Maintainability:** Implementations can be changed with fewer modifications to consumers.
- **Centralized management:** Spring handles bean creation, configuration, and lifecycle callbacks.

## 6. Common Interview Follow-Up

**Q: Is IoC the same as DI?**

No. IoC is the broader principle of transferring control to a framework or container. DI is a technique for implementing IoC by supplying dependencies externally. Spring uses DI as one of the main ways to implement IoC.

## One-Line Interview Answer

**IoC is the principle of delegating control over object creation and management to a container, whereas DI is the technique of supplying dependencies from outside an object; Spring uses dependency injection to implement inversion of control.**



---

