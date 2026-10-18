---
title: Spring MVC and REST
parent: Spring Framework
nav_order: 5
---

# Spring MVC and REST
{: .no_toc }

Request flow, filters and interceptors.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What Is the Spring MVC Flow?

Spring MVC is a web framework based on the **Model–View–Controller (MVC)** design pattern. It separates request handling, application data, and presentation logic.

The central component is the **`DispatcherServlet`**, which acts as the front controller. It coordinates request mapping, controller execution, exception handling, and response generation.

### 1. Spring MVC Request Flow

<text color="secondary" size="sm">Traditional Spring MVC flow with a server-rendered view</text>

```text
         Client / Browser
                |
                v
         HTTP Request
                |
                v
          Filter Chain
                |
                v
        DispatcherServlet
                |
                v
          HandlerMapping
                |
                v
         HandlerAdapter
                |
                v
            Controller
                |
                v
           Service Layer
                |
                v
          Repository / DB
                |
                v
        Controller returns
        Model + View Name
                |
                v
           ViewResolver
                |
                v
             View
          (JSP / Thymeleaf)
                |
                v
        Rendered HTTP Response
                |
                v
          Client / Browser
```

This diagram shows the traditional server-rendered MVC path. REST APIs follow a slightly different response path, explained below.

### 2. Step-by-Step Explanation

**Step 1: Client sends an HTTP request**

The client sends a request to the application, for example:

```http
GET /users/10
```

**Step 2: Filters process the request**

Servlet filters can perform preprocessing, such as request logging, correlation-ID handling, and security-chain processing, before the request reaches the `DispatcherServlet`.

**Step 3: `DispatcherServlet` receives the request**

The `DispatcherServlet` is the front controller of Spring MVC. It coordinates the request lifecycle rather than containing the business logic itself.

**Step 4: `HandlerMapping` identifies the handler**

Spring uses handler mappings to find the controller method that matches the request.

For example:

```java
@GetMapping("/users/{id}")
public String getUser(@PathVariable Long id, Model model) {
    // ...
}
```

The request `GET /users/10` matches this handler method.

**Step 5: `HandlerAdapter` invokes the handler**

A `HandlerAdapter` invokes the selected handler. For annotated controllers, Spring commonly uses `RequestMappingHandlerAdapter` to invoke methods annotated with `@RequestMapping`, `@GetMapping`, and related annotations.

**Step 6: Controller calls the service layer**

The controller handles HTTP-specific concerns and delegates business logic to a service.

```java
@Controller
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/users/{id}")
    public String getUser(
            @PathVariable Long id,
            Model model) {

        User user = userService.getUserById(id);
        model.addAttribute("user", user);

        return "user-details";
    }
}
```

**Step 7: Service and repository retrieve data**

The service performs business logic and typically calls a repository to retrieve or update data in a database.

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User getUserById(Long id) {
        return userRepository.findById(id)
                .orElseThrow(() ->
                        new UserNotFoundException(
                                "User not found: " + id));
    }
}
```

**Step 8: Controller returns a logical view name and model**

The controller adds data to the `Model` and returns `"user-details"`. This is a logical view name, not necessarily a physical filename.

**Step 9: `ViewResolver` resolves the view**

The `ViewResolver` maps the logical name to the appropriate view implementation or template. For example, a Thymeleaf configuration might resolve it to a template such as `templates/user-details.html`.

**Step 10: View renders the response**

The view renders HTML using the model data. Spring MVC then sends the resulting HTTP response to the client.

### 3. How Does the Flow Change in a REST API?

In Spring Boot REST APIs, controllers commonly use `@RestController` and return Java objects instead of logical view names.

```java
@RestController
@RequestMapping("/api/users")
public class UserRestController {

    private final UserService userService;

    public UserRestController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUserById(id);
    }
}
```

The simplified flow becomes:

```text
Client Request
      |
      v
DispatcherServlet
      |
      v
HandlerMapping
      |
      v
HandlerAdapter
      |
      v
@RestController
      |
      v
Service -> Repository -> Database
      |
      v
Java Object Returned
      |
      v
HttpMessageConverter
   (e.g. Jackson)
      |
      v
JSON HTTP Response
      |
      v
Client
```

Here, Spring typically uses an `HttpMessageConverter` to serialize the Java object into JSON. A `ViewResolver` is generally **not involved** in this REST response path.

Example JSON response:

```json
{
  "id": 10,
  "name": "Danish"
}
```

### 4. Important Components of Spring MVC

| Component | Responsibility |
|---|---|
| `DispatcherServlet` | Front controller; coordinates the MVC request lifecycle |
| `HandlerMapping` | Finds the handler for the incoming request |
| `HandlerAdapter` | Invokes the selected handler |
| `Controller` | Handles HTTP requests and delegates application work |
| `Model` | Carries data for server-rendered views |
| `ViewResolver` | Resolves logical view names to views |
| `View` | Renders a response, commonly HTML |
| `HttpMessageConverter` | Converts request and response bodies, such as JSON, to and from Java objects |
| `HandlerExceptionResolver` | Resolves exceptions raised during Spring MVC request processing |

### 5. Difference Between `@Controller` and `@RestController`

| Feature | `@Controller` | `@RestController` |
|---|---|---|
| Typical use | MVC applications that render views | REST APIs |
| Return value | Often a view name | Often an object serialized into JSON or XML |
| Response-body behavior | Use `@ResponseBody` when returning the body directly | Included by default through `@ResponseBody` |
| `ViewResolver` | Often used for view names | Usually not used for object responses |

### 6. Interview Answer

"Spring MVC follows the Front Controller pattern, where `DispatcherServlet` is the central entry point for HTTP requests. It uses `HandlerMapping` to identify the appropriate controller and `HandlerAdapter` to invoke it. The controller delegates business logic to the service layer, which may interact with repositories and databases.

For traditional MVC applications, the controller returns a logical view name and model data. `ViewResolver` identifies the view, which renders the response. In REST APIs, the controller usually returns an object, and an `HttpMessageConverter`, such as Jackson, serializes it into JSON. Therefore, REST responses generally do not require a `ViewResolver`."

### Quick Revision

- **Front Controller:** `DispatcherServlet`
- **Finds the handler:** `HandlerMapping`
- **Invokes the handler:** `HandlerAdapter`
- **Business logic:** Service layer
- **Database access:** Repository layer
- **Resolves server-rendered views:** `ViewResolver`
- **Serializes REST responses:** `HttpMessageConverter`
- **Traditional MVC response:** Model + View
- **REST API response:** Java object converted to JSON or another representation


---


## What Design Patterns Are Used in the Spring Framework?

The Spring Framework uses several design patterns to manage object creation, dependency injection, database access, cross-cutting concerns, and communication between components.

For interviews, remember that **IoC is a principle, not a classic GoF design pattern**, and Spring's bean scope of `singleton` is not exactly the same as the traditional Singleton design pattern.

### 1. Important Design Patterns in Spring

| Design pattern | Where Spring uses it | Purpose |
|---|---|---|
| Inversion of Control (IoC) / Dependency Injection | Spring IoC container | Delegates object creation and dependency management to the container |
| Singleton | Default bean scope | Shares one bean instance per bean definition per container |
| Factory | `BeanFactory`, `ApplicationContext` | Creates and manages beans |
| Template Method | `JdbcTemplate`, `RestTemplate` | Reuses common workflow while allowing specific operations to vary |
| Proxy | Spring AOP, Spring Security | Adds behavior around method calls |
| Observer | Application events | Notifies listeners when events are published |
| Adapter | `HandlerAdapter` in Spring MVC | Allows different handler types to work through a common interface |
| Front Controller | `DispatcherServlet` | Centralizes request processing |
| Strategy | `PlatformTransactionManager`, resource selection | Encapsulates interchangeable implementations of behavior |
| Decorator | Wrappers and response/request decorators | Adds behavior while preserving an interface |
| Facade | `ApplicationContext`, higher-level APIs | Provides a convenient interface to underlying functionality |

The exact pattern used depends on the Spring module and implementation. Some Spring features combine several patterns.

### 2. Inversion of Control (IoC) and Dependency Injection

**Concept:** Instead of application classes creating and managing their dependencies, the Spring container manages beans and supplies their dependencies.

Without dependency injection:

```java
public class OrderService {

    private final OrderRepository repository =
            new OrderRepository();
}
```

With Spring dependency injection:

```java
@Service
public class OrderService {

    private final OrderRepository repository;

    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}
```

Spring creates and injects the repository bean.

**Interview point:** IoC is the broader principle; Dependency Injection is one way to implement it.

### 3. Singleton Pattern

Spring's default bean scope is `singleton`.

```java
@Service
public class PaymentService {
}
```

Within the same Spring container and bean definition, Spring normally maintains one shared `PaymentService` instance.

You can also specify it explicitly:

```java
@Service
@Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
public class PaymentService {
}
```

**Important distinction:**

- Traditional Singleton pattern: the class itself generally restricts instance creation.
- Spring singleton scope: the container manages the shared instance.
- A singleton bean is not necessarily one instance across the entire JVM. Different containers or bean definitions can have separate instances.

Also, singleton scope does not automatically make a bean thread-safe.

### 4. Factory Pattern

Spring uses factory-based abstractions to create and manage objects.

Examples include:

- `BeanFactory`
- `ApplicationContext`
- `FactoryBean`

Example using `FactoryBean`:

```java
public class PaymentClientFactory
        implements FactoryBean<PaymentClient> {

    @Override
    public PaymentClient getObject() {
        return new PaymentClient();
    }

    @Override
    public Class<?> getObjectType() {
        return PaymentClient.class;
    }
}
```

Spring uses the factory to obtain the `PaymentClient` object.

**Interview point:** `BeanFactory` is a core container interface. `FactoryBean` lets a bean definition supply an object through custom creation logic.

### 5. Template Method Pattern

The Template Method pattern defines a common algorithm structure while allowing selected steps to vary.

Spring provides template classes such as `JdbcTemplate` that handle repetitive infrastructure work.

```java
@Repository
public class UserRepository {

    private final JdbcTemplate jdbcTemplate;

    public UserRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public String findNameById(Long id) {
        return jdbcTemplate.queryForObject(
                "SELECT name FROM users WHERE id = ?",
                String.class,
                id
        );
    }
}
```

`JdbcTemplate` manages much of the repetitive JDBC workflow, such as resource handling and exception translation, while the application supplies the query and parameters.

**Interview point:** `JdbcTemplate` is a classic example of the Template Method pattern in Spring.

### 6. Proxy Pattern

Spring uses proxies to intercept method calls and apply additional behavior.

Common examples include:

- `@Transactional` — transaction management.
- `@Cacheable` — caching.
- `@Async` — asynchronous method execution.
- Spring AOP advice — logging, auditing, and other cross-cutting concerns.

Example:

```java
@Service
public class PaymentService {

    @Transactional
    public void processPayment() {
        // Perform database operations.
    }
}
```

When invoked through the Spring proxy, the method can run within a transaction managed by Spring.

**Important:** Proxy-based behavior typically applies to calls that pass through the proxy. A self-invocation such as one method calling another method on `this` can bypass proxy advice.

**Interview point:** Proxy is the most precise pattern to cite for Spring AOP and declarative transaction management. AOP adds cross-cutting behavior; the proxy is one mechanism Spring uses to apply it.

### 7. Observer Pattern

The Observer pattern allows interested listeners to receive notifications when an event occurs.

Spring supports application events through `ApplicationEventPublisher` and `@EventListener`.

```java
public record UserRegisteredEvent(Long userId) {
}
```

Publish the event:

```java
@Service
public class UserService {

    private final ApplicationEventPublisher publisher;

    public UserService(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void registerUser(Long userId) {
        // Save user to the database.

        publisher.publishEvent(
                new UserRegisteredEvent(userId)
        );
    }
}
```

Listen for the event:

```java
@Component
public class WelcomeEmailListener {

    @EventListener
    public void handleUserRegistered(
            UserRegisteredEvent event) {

        System.out.println(
                "Send welcome email to user "
                + event.userId()
        );
    }
}
```

By default, Spring application event listeners run synchronously in the publishing thread unless configured otherwise.

**Interview point:** Spring application events are a practical example of the Observer pattern.

### 8. Adapter Pattern

The Adapter pattern makes incompatible interfaces work together through a common abstraction.

Spring MVC uses `HandlerAdapter` to invoke different types of request handlers through a common mechanism.

Simplified flow:

```text
HTTP Request
     |
     v
DispatcherServlet
     |
     v
HandlerMapping
     |
     v
HandlerAdapter
     |
     v
Selected Controller Handler
```

The `DispatcherServlet` does not need to know the internal invocation details of every supported handler type. The appropriate adapter handles those details.

**Interview point:** `HandlerAdapter` is a direct example of the Adapter pattern.

### 9. Front Controller Pattern

Spring MVC uses the Front Controller pattern through `DispatcherServlet`.

Instead of each controller independently managing the entire HTTP request lifecycle, requests enter through a central servlet.

```text
Client
  |
  v
DispatcherServlet
  |
  +--> HandlerMapping
  |
  +--> HandlerAdapter
  |
  +--> Controller
  |
  +--> Exception Resolution
  |
  +--> Response
```

This centralizes request dispatching and integrates common MVC infrastructure.

### 10. Strategy Pattern

The Strategy pattern defines interchangeable implementations of an operation.

For example, Spring's transaction infrastructure supports different transaction-manager implementations through the `PlatformTransactionManager` interface.

```java
public interface PlatformTransactionManager {
    // Transaction-management operations.
}
```

Depending on the application, Spring can use an appropriate implementation for JDBC, JPA, or another supported transaction technology.

**Interview point:** The strategy is selected through configuration or dependency injection, allowing application code to depend on an abstraction rather than a particular implementation.

### 11. Decorator Pattern

The Decorator pattern adds behavior to an object while preserving its interface.

Java I/O wrappers are a familiar example:

```java
InputStream input =
        new BufferedInputStream(
                new FileInputStream("data.txt")
        );
```

`BufferedInputStream` wraps another `InputStream` and adds buffering.

Spring applications can use wrapper-based decorators for requests, responses, and other components. However, **Spring AOP should primarily be identified as proxy-based interception**, not simply described as the Decorator pattern.

### 12. Facade Pattern

The Facade pattern provides a simpler interface to a more complex subsystem.

`ApplicationContext` offers a convenient, high-level interface to Spring's container functionality, including bean lookup, resource access, and event publication.

For example:

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(
                AppConfig.class
        );

PaymentService service =
        context.getBean(PaymentService.class);
```

This illustrates a convenient container API, although `ApplicationContext` is also a container abstraction and should not be treated as a pure textbook Facade implementation in every respect.

### 13. Quick Interview Revision

| Interview question | Best answer |
|---|---|
| Which pattern does Spring MVC use for request handling? | Front Controller — `DispatcherServlet` |
| Which pattern is used by Spring AOP? | Proxy |
| Which pattern is used by `JdbcTemplate`? | Template Method |
| Which pattern is used by `HandlerAdapter`? | Adapter |
| Which pattern is used by Spring application events? | Observer |
| What is the default Spring bean scope? | Singleton |
| What creates and manages Spring beans? | IoC container, exposed through interfaces such as `BeanFactory` |
| Which pattern supports interchangeable transaction implementations? | Strategy |
| Is IoC a GoF design pattern? | No; it is a design principle |

### Sample Interview Answer

"Spring uses multiple design patterns to keep applications loosely coupled and maintainable. The IoC container implements dependency injection to manage beans. Spring's default bean scope is singleton. `BeanFactory` and `FactoryBean` provide factory-based object creation, while `JdbcTemplate` demonstrates the Template Method pattern. Spring AOP and declarative transactions commonly use proxies. Spring MVC uses the Front Controller pattern through `DispatcherServlet` and the Adapter pattern through `HandlerAdapter`. Spring application events demonstrate the Observer pattern, and transaction-manager abstractions illustrate the Strategy pattern."



---


## What Is the Difference Between Spring and Spring Boot?

Spring is a Java application development framework that provides features such as Dependency Injection, Spring MVC, transaction management, and data access.

Spring Boot builds on the Spring Framework to make applications easier and faster to configure, develop, run, and deploy. It provides auto-configuration, starter dependencies, and convenient production-ready features.

**Key interview point:** Spring Boot does not replace Spring. It simplifies the development and configuration of Spring applications.

### 1. Spring vs. Spring Boot

| Feature | Spring Framework | Spring Boot |
|---|---|---|
| Purpose | Provides core application development features | Simplifies building and running Spring applications |
| Configuration | Often requires more explicit configuration | Provides auto-configuration and sensible defaults |
| Dependency management | Dependencies are selected and configured explicitly | Starter dependencies simplify dependency selection |
| Server setup | Traditional deployments may require an external server | Supports embedded servers such as Tomcat |
| Deployment | Commonly deployed as a WAR to an external server, or run through other setups | Commonly packaged as an executable JAR |
| Dependency Injection | Supported | Uses Spring's Dependency Injection |
| Spring MVC | Supported | Integrates Spring MVC easily |
| Production monitoring | Can be configured using available libraries | Spring Boot Actuator provides convenient health and metrics endpoints |
| Microservices development | Can be used to build microservices | Provides convenient defaults and integrates with Spring Cloud |
| Flexibility | Highly configurable | Also highly configurable, with sensible defaults |

**Important:** Spring Framework applications can also run as standalone applications, and Spring Boot applications can be packaged as WAR files. These are common deployment differences, not strict limitations.

### 2. What Does Spring Boot Provide?

#### A. Auto-configuration

Spring Boot configures application components based on the dependencies available on the classpath and the application's configuration.

For example, when the appropriate Spring MVC dependencies are present, Spring Boot configures much of the infrastructure needed for a web application.

```java
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(
                MyApplication.class, args);
    }
}
```

`@SpringBootApplication` combines:

- `@Configuration` — allows the class to provide bean definitions.
- `@EnableAutoConfiguration` — enables Spring Boot auto-configuration.
- `@ComponentScan` — enables component scanning from the application's package.

Auto-configuration is conditional: you can override or replace many defaults with your own configuration.

#### B. Starter Dependencies

Spring Boot starters simplify dependency management by grouping commonly used libraries.

For example, in Maven:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

This starter brings in commonly needed dependencies for Spring MVC web applications and embedded-server support.

Other common starters include:

- `spring-boot-starter-data-jpa` — JPA and database access.
- `spring-boot-starter-security` — Spring Security.
- `spring-boot-starter-test` — testing libraries.
- `spring-boot-starter-actuator` — application monitoring and management.

Use a compatible Spring Boot version and its managed dependency versions rather than selecting arbitrary versions for every library.

#### C. Embedded Server

Spring Boot can run a web application with an embedded server, such as Tomcat, without requiring a separately installed application server.

For example:

```bash
java -jar target/my-application.jar
```

The application can start its embedded server and serve HTTP requests.

#### D. Production-Ready Features

Spring Boot Actuator provides endpoints and integrations for monitoring application health, metrics, and other operational information.

For example, a health endpoint is commonly available at:

```text
/actuator/health
```

Endpoints must be appropriately configured and secured before being exposed in production.

### 3. How Would You Build a Web Application with Spring vs. Spring Boot?

**With Spring Framework alone**, you typically choose and configure the necessary modules, dependencies, servlet infrastructure, and deployment setup yourself. The exact setup depends on the application and the configuration approach.

**With Spring Boot**, you commonly add a starter dependency, configure application properties, and create the application entry point.

Example configuration:

```properties
server.port=8081
spring.application.name=user-service
```

Spring Boot uses these settings to configure the application's name and HTTP server port.

### 4. Why Choose Spring Boot Over Spring Framework Alone?

Choose Spring Boot when you want to:

- Develop REST APIs and microservices quickly.
- Reduce repetitive infrastructure configuration.
- Use embedded servers and executable JAR packaging.
- Integrate database, security, testing, and monitoring libraries conveniently.
- Establish consistent defaults across development teams.
- Build cloud-native applications with Spring Cloud and other ecosystem tools when needed.

### 5. When Might You Use Spring Framework Without Spring Boot?

A Spring application without Boot can make sense when:

- You are maintaining an existing application with established configuration and deployment conventions.
- You need to integrate into an environment that manages application-server deployment centrally.
- Your organization has a specific framework or platform configuration that should remain unchanged.
- You need explicit control over configuration and prefer not to use Boot's conventions.

However, Spring Boot is also highly customizable. You do not need to avoid Boot simply because you require flexibility.

### 6. Is Spring Boot Better for Microservices?

Spring Boot is often the practical choice for microservices because it simplifies independent application startup, dependency management, deployment, and operational monitoring.

For example, an architecture might contain:

```text
API Gateway
     |
     +----> User Service (Spring Boot)
     |
     +----> Order Service (Spring Boot)
     |
     +----> Payment Service (Spring Boot)
```

Each service can be packaged and deployed independently. Spring Boot does not itself provide every microservices capability: service discovery, distributed configuration, messaging, and other distributed-system features may require Spring Cloud or additional infrastructure.

### 7. Interview Answer

"Spring Framework provides the core capabilities for developing Java applications, including Dependency Injection, Spring MVC, and transaction management. Spring Boot builds on Spring and reduces the setup required through auto-configuration, starter dependencies, embedded servers, and production-ready features such as Actuator.

I would generally choose Spring Boot for new REST APIs, microservices, and cloud-native applications because it speeds up development and simplifies deployment. I would use Spring Framework without Boot when an existing platform or deployment architecture requires a more traditional configuration approach. Spring Boot is not a replacement for Spring; it makes Spring easier to use."

### Quick Revision

- **Spring:** Core framework and application development capabilities.
- **Spring Boot:** Simplified configuration, dependency setup, and deployment.
- **Auto-configuration:** Configures components based on dependencies and settings.
- **Starter dependencies:** Simplify dependency selection.
- **Embedded server:** Run web applications without separately deploying to an external server.
- **Actuator:** Health, metrics, and operational endpoints.
- **Recommended default for new Spring REST APIs:** Spring Boot.


---



## 1. What Is Method Overloading and Method Overriding?

Method overloading and method overriding are two important concepts in Java polymorphism.

- **Method overloading** is compile-time polymorphism.
- **Method overriding** is runtime polymorphism.

### A. Method Overloading

Method overloading occurs when multiple methods in the same class have the **same name but different parameter lists**.

The parameters can differ in number, type, or order.

```java
public class Calculator {

    public int add(int a, int b) {
        return a + b;
    }

    public int add(int a, int b, int c) {
        return a + b + c;
    }

    public double add(double a, double b) {
        return a + b;
    }
}
```

Usage:

```java
Calculator calculator = new Calculator();

System.out.println(calculator.add(10, 20));       // 30
System.out.println(calculator.add(10, 20, 30));   // 60
System.out.println(calculator.add(2.5, 3.5));     // 6.0
```

**Important rules:**

- Changing only the return type does not create a valid overload.
- Overloaded methods may have different access modifiers and return types, provided the parameter lists distinguish them.
- The compiler selects the applicable overload based on the arguments and Java's method-resolution rules.

### B. Method Overriding

Method overriding occurs when a subclass provides its own implementation of an inherited instance method with a compatible signature.

```java
class Animal {

    public void sound() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {

    @Override
    public void sound() {
        System.out.println("Dog barks");
    }
}
```

Usage:

```java
Animal animal = new Dog();

animal.sound(); // Dog barks
```

Although the reference type is `Animal`, Java invokes the overridden implementation in `Dog` at runtime.

**Important rules:**

- The overriding method must have a compatible return type.
- It cannot reduce the visibility of the inherited method.
- `@Override` helps the compiler verify that the method actually overrides an inherited method.
- Static methods are hidden, not overridden.
- Private methods are not overridden, and constructors cannot be overridden.

### 2. Difference Between Overloading and Overriding

| Feature | Overloading | Overriding |
|---|---|---|
| Purpose | Provide multiple ways to call a method | Customize inherited behavior |
| Relationship | Can occur within the same class | Requires inheritance or an applicable subtype relationship |
| Parameters | Must differ | Must match the overridden method's signature |
| Polymorphism | Compile-time | Runtime |
| Return type | Can differ when parameter lists distinguish methods | Must be compatible with the inherited method |
| `@Override` | Not used for ordinary overloading | Recommended |
| Example | `getBean(String)` and `getBean(Class)` | A subclass implements an inherited method differently |

## 3. Where Is Method Overloading Used in Spring?

### A. `ApplicationContext.getBean()`

Spring provides overloaded `getBean()` methods to retrieve beans in different ways.

```java
ApplicationContext context = ...;

// Retrieve by bean name
Object bean1 = context.getBean("userService");

// Retrieve by type
UserService bean2 = context.getBean(UserService.class);

// Retrieve by name and expected type
UserService bean3 =
        context.getBean("userService", UserService.class);
```

These methods have the same name but different parameter lists, making them examples of method overloading.

### B. `JdbcTemplate`

`JdbcTemplate` offers overloaded query methods that support different combinations of SQL, parameters, and result-handling callbacks.

For example:

```java
jdbcTemplate.queryForObject(
        "SELECT COUNT(*) FROM users",
        Integer.class
);
```

Another overload allows SQL parameters:

```java
jdbcTemplate.queryForObject(
        "SELECT COUNT(*) FROM users WHERE status = ?",
        Integer.class,
        "ACTIVE"
);
```

The method overloads provide a convenient API for different query requirements.

### C. Dependency Injection

Spring supports constructor injection, setter injection, and field injection. However, **these are not examples of overloading the `@Autowired` annotation**. They are different dependency-injection mechanisms.

Constructors themselves can be overloaded in Java, but Spring's constructor-selection rules determine which constructor is used. For most application code, prefer a single constructor for required dependencies.

## 4. Where Is Method Overriding Used in Spring?

### A. Implementing Spring Interfaces

Spring provides interfaces that applications implement to supply custom behavior.

For example, implementing `CommandLineRunner`:

```java
@Component
public class StartupTask implements CommandLineRunner {

    @Override
    public void run(String... args) {
        System.out.println("Application startup task");
    }
}
```

The `run()` method implements an interface method. Technically, this is **interface implementation**, not class-method overriding, because `CommandLineRunner` is an interface.

### B. Extending Spring Classes

When a subclass extends a Spring class and customizes an inherited method, that is genuine method overriding.

```java
public class CustomService {

    public void execute() {
        System.out.println("Default execution");
    }
}

public class SpecialService extends CustomService {

    @Override
    public void execute() {
        System.out.println("Customized execution");
    }
}
```

Spring framework extension points sometimes use this approach. For example, custom implementations can extend framework base classes and override supported methods.

### C. Spring MVC Controller Methods

A controller method annotated with `@GetMapping` is not automatically an example of method overriding. It is simply a request handler.

For interview answers, use clear examples such as overriding an inherited method or implementing an interface method, and distinguish the two accurately.

## 5. If a Class Must Not Be Extended and Its Instances Must Not Be Created by Other Classes, How Do You Declare It?

This requirement has two parts:

1. Prevent inheritance.
2. Prevent outside code from directly invoking the constructor.

Use a `final` class with a `private` constructor.

```java
public final class UtilityClass {

    private UtilityClass() {
        // Prevent direct construction from outside.
    }

    public static void doSomething() {
        System.out.println("Utility method");
    }
}
```

- `final` prevents other classes from extending `UtilityClass`.
- `private` prevents other classes from calling its constructor.
- Static methods can be called without creating an instance.

Usage:

```java
UtilityClass.doSomething();
```

This works, but the following is not allowed:

```java
UtilityClass obj = new UtilityClass(); // Compilation error
```

Nor can another class extend it:

```java
class Child extends UtilityClass { } // Compilation error
```

### Important distinction

A private constructor does not necessarily mean an instance can never be created. Code inside the class itself can invoke it, and other mechanisms such as reflection may bypass normal access restrictions depending on permissions and runtime conditions.

If your goal is a utility class with no instances, `final` plus a private constructor is the standard approach. If a controlled factory should create instances, keep the constructor private and expose a static factory method instead.

## 6. Interview Quick Revision

- **Overloading:** Same method name, different parameter lists.
- **Overriding:** A subclass replaces inherited instance-method behavior.
- **Compile-time polymorphism:** Overloading.
- **Runtime polymorphism:** Overriding.
- **Spring overloading example:** `ApplicationContext.getBean()`.
- **Spring interface implementation example:** `CommandLineRunner.run()`.
- **Prevent extension:** `final` class.
- **Prevent direct external construction:** `private` constructor.
- **Standard non-instantiable utility class:** `final` class + private constructor + static methods.
