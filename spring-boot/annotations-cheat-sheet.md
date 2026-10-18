---
title: Annotations Cheat Sheet
parent: Spring Boot
nav_order: 2
---

# Annotations Cheat Sheet
{: .no_toc }

Most-used annotations grouped by purpose.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

# Important Spring Boot Annotations

## 1. `@Component`

### What is `@Component`?

`@Component` marks a Java class as a Spring-managed component. Spring discovers the class through component scanning and registers it as a bean in the Spring ApplicationContext.

A Spring-managed bean can have its dependencies injected and its lifecycle managed by the Spring container.

### Example

```java
import org.springframework.stereotype.Component;

@Component
public class EmailValidator {

    public boolean isValid(String email) {
        return email != null && email.contains("@");
    }
}
```

Injecting the component into another class:

```java
@Service
public class UserService {

    private final EmailValidator emailValidator;

    public UserService(EmailValidator emailValidator) {
        this.emailValidator = emailValidator;
    }

    public boolean validateEmail(String email) {
        return emailValidator.isValid(email);
    }
}
```

### Important points

- `@Component` is a general-purpose stereotype annotation.
- It makes a class eligible for detection through component scanning.
- By default, Spring uses singleton scope for a component bean.
- The default bean name is usually the class name with its first letter in lowercase, such as `emailValidator`.
- `@Service`, `@Repository`, and `@Controller` are specialized stereotype annotations built on top of `@Component`.

**Interview answer:** `@Component` tells Spring to discover and manage a class as a bean so that it can participate in dependency injection and other container-managed functionality.

## 2. `@Autowired`

### What is `@Autowired`?

`@Autowired` tells Spring to resolve and inject a dependency into a Spring-managed bean.

Spring normally resolves the dependency by type. If multiple beans match, you may need `@Qualifier`, `@Primary`, or another explicit configuration.

### Example: Constructor injection

```java
@Repository
public class UserRepository {

    public String findUser() {
        return "Danish";
    }
}
```

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    // @Autowired is optional for a single constructor
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public String getUser() {
        return userRepository.findUser();
    }
}
```

Because `UserService` has only one constructor, Spring can use it for dependency injection without an explicit `@Autowired`.

### Other injection types

**Field injection:**

```java
@Autowired
private UserRepository userRepository;
```

**Setter injection:**

```java
@Autowired
public void setUserRepository(UserRepository userRepository) {
    this.userRepository = userRepository;
}
```

Constructor injection is generally preferred because it makes required dependencies explicit, supports immutable fields, and simplifies unit testing.

### What if multiple beans implement the same interface?

Suppose two implementations exist:

```java
public interface NotificationService {
    void send(String message);
}
```

```java
@Service
public class EmailNotificationService
        implements NotificationService {

    public void send(String message) {
        System.out.println("Email: " + message);
    }
}
```

```java
@Service
public class SmsNotificationService
        implements NotificationService {

    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}
```

Injecting `NotificationService` by type alone may be ambiguous. Use `@Qualifier` to select the intended bean:

```java
@Service
public class AlertService {

    private final NotificationService notificationService;

    public AlertService(
            @Qualifier("emailNotificationService")
            NotificationService notificationService) {
        this.notificationService = notificationService;
    }
}
```

Alternatively, `@Primary` can identify a preferred bean when multiple candidates exist.

**Interview answer:** `@Autowired` performs dependency injection by asking Spring to resolve a suitable bean. Constructor injection is generally best for required dependencies, and `@Qualifier` can resolve ambiguity when multiple beans match.

## 3. `@GetMapping`

### What is `@GetMapping`?

`@GetMapping` maps HTTP GET requests to a controller handler method. It is a specialized form of `@RequestMapping(method = RequestMethod.GET)`.

It is commonly used to retrieve resources or read data.

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping
    public String getUsers() {
        return "List of users";
    }

    @GetMapping("/{id}")
    public String getUserById(@PathVariable Long id) {
        return "User ID: " + id;
    }
}
```

Example requests:

```http
GET /api/users
GET /api/users/10
```

The second request extracts `10` from the URL using `@PathVariable`.

### Using query parameters

```java
@GetMapping("/search")
public String searchUsers(
        @RequestParam String name) {
    return "Searching for: " + name;
}
```

Example request:

```http
GET /api/users/search?name=Danish
```

Here, `@RequestParam` extracts the query parameter, while `@PathVariable` extracts a value embedded in the URL path.

**Interview answer:** `@GetMapping` maps GET requests to controller methods, typically for retrieving data. It supports path variables, query parameters, and other request-mapping attributes.

## 4. `@PostMapping`

### What is `@PostMapping`?

`@PostMapping` maps HTTP POST requests to a controller handler method. It is commonly used to create resources or submit data for processing.

It is a specialized form of `@RequestMapping(method = RequestMethod.POST)`.

### Example: Creating a user

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public String createUser(@RequestBody UserRequest request) {
        return "Created user: " + request.getName();
    }
}
```

Request DTO:

```java
public class UserRequest {

    private String name;
    private String email;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

Example request:

```http
POST /api/users
Content-Type: application/json

{
  "name": "Danish",
  "email": "danish@example.com"
}
```

`@RequestBody` tells Spring to deserialize the HTTP request body into a Java object. For JSON requests, this is commonly handled by Jackson when the appropriate dependencies are present.

### Using a path variable with POST

```java
@PostMapping("/{id}/submit")
public String submitUser(
        @PathVariable Long id,
        @RequestBody UserRequest request) {

    return "Submitted user " + id
            + " with name " + request.getName();
}
```

Example request:

```http
POST /api/users/10/submit
```

**Interview answer:** `@PostMapping` maps POST requests to a controller method. `@RequestBody` binds the request body to a Java object, and `@PathVariable` extracts values from the URL path.

## 5. `@Repository`

### What is `@Repository`?

`@Repository` marks a class as a data-access component. It communicates the class's persistence-related role and makes it eligible for Spring component scanning.

Spring can also apply persistence exception translation to eligible repository beans, converting supported technology-specific persistence exceptions into Spring's `DataAccessException` hierarchy when the required infrastructure is configured.

### Example

```java
@Repository
public class UserRepository {

    public String findById(Long id) {
        // In a real application, query the database here.
        return "User " + id;
    }
}
```

Injecting the repository into a service:

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public String getUser(Long id) {
        return userRepository.findById(id);
    }
}
```

### Repository with Spring Data JPA

For many applications, you do not need to implement the data-access methods manually. Spring Data JPA can generate a repository implementation:

```java
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

Spring Data creates the implementation at runtime when its repository infrastructure is configured.

**Interview answer:** `@Repository` identifies the persistence layer. It makes a repository class eligible for component scanning and supports persistence exception translation. Spring Data repositories can also provide generated data-access implementations.

## 6. `@Service`

### What is `@Service`?

`@Service` marks a class as a service-layer component. It is typically used for business logic, validation, orchestration, and coordination between repositories or external services.

It is a specialized stereotype annotation built on `@Component`.

### Example

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public String getUserDetails(Long id) {
        // Business logic can be applied here.
        return userRepository.findById(id);
    }
}
```

A service can coordinate multiple dependencies:

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final PaymentService paymentService;

    public OrderService(
            OrderRepository orderRepository,
            PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
    }

    public void placeOrder(Order order) {
        paymentService.processPayment(order);
        orderRepository.save(order);
    }
}
```

This illustrates the service layer coordinating business operations. In production, order placement and payment processing often require additional transaction, idempotency, and failure-handling design.

**Interview answer:** `@Service` identifies the business-logic layer. It makes the class eligible for Spring component scanning and allows Spring to inject its dependencies.

## 7. `@Controller`

### What is `@Controller`?

`@Controller` marks a Spring MVC controller that handles web requests. It is generally used in server-rendered web applications, where a handler method often returns a view name.

### Example

```java
@Controller
public class HomeController {

    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("message", "Welcome!");
        return "home";
    }
}
```

In this example:

1. A client sends `GET /home`.
2. Spring routes the request to the `home()` method.
3. The controller adds data to the `Model`.
4. The method returns the view name `home`.
5. Spring MVC resolves and renders the corresponding view when the view technology is configured.

The returned string is treated as a view name in this example, not as a literal response body.

### `@Controller` vs `@RestController`

`@RestController` is effectively a combination of `@Controller` and `@ResponseBody`. It makes handler return values the response body by default, typically serialized as JSON when returning objects.

Example:

```java
@RestController
public class UserApiController {

    @GetMapping("/api/message")
    public String message() {
        return "Hello, World!";
    }
}
```

The response body is:

```text
Hello, World!
```

With a typical JSON message API, an object return value is serialized into JSON by an appropriate message converter.

**Interview answer:** `@Controller` is used for Spring MVC request handling, often returning view names. `@RestController` is used for REST APIs where return values are written directly to the HTTP response body.

## 8. Comparison of these annotations

| Annotation | Primary purpose | Typical layer |
|---|---|---|
| `@Component` | General Spring-managed component | Any layer |
| `@Autowired` | Inject a dependency | Dependency wiring |
| `@GetMapping` | Map HTTP GET requests | Controller |
| `@PostMapping` | Map HTTP POST requests | Controller |
| `@Repository` | Identify data-access components | Persistence |
| `@Service` | Identify business-logic components | Service |
| `@Controller` | Handle web requests, often returning views | Presentation |

## 9. How do they work together?

A typical Spring Boot application follows this structure:

```text
Client
  |
  | HTTP GET /api/users/10
  v
UserController
  |
  | calls
  v
UserService
  |
  | calls
  v
UserRepository
  |
  v
Database
```

Example responsibilities:

- `@RestController` or `@Controller` handles incoming HTTP requests.
- `@GetMapping` or `@PostMapping` maps requests to handler methods.
- `@Service` contains business logic.
- `@Repository` handles data access.
- Constructor injection (optionally marked with `@Autowired`) connects the layers.
- `@Component` can be used for supporting classes that do not fit a more specific stereotype.

Remember that Spring must discover or otherwise register these components, and their dependencies must be resolvable for the application context to start successfully.

## Quick interview revision

1. **What is `@Component`?** A general-purpose annotation for a Spring-managed component.
2. **What is `@Autowired`?** An annotation used for dependency injection; a single constructor does not require it.
3. **What is `@GetMapping`?** Maps HTTP GET requests to controller methods.
4. **What is `@PostMapping`?** Maps HTTP POST requests to controller methods.
5. **What is `@Repository`?** Identifies data-access components and supports persistence exception translation.
6. **What is `@Service`?** Identifies the business-logic layer.
7. **What is `@Controller`?** Handles web requests and commonly returns view names.
8. **What is the difference between `@Controller` and `@RestController`?** `@RestController` writes return values to the response body by default.
9. **What is the difference between `@PathVariable` and `@RequestParam`?** The former reads from the URL path; the latter reads request parameters.
10. **Which injection style is preferred?** Constructor injection for required dependencies.


---

# What Is the Use of `@ComponentScan` in Spring Boot?

## 1. What is `@ComponentScan`?

`@ComponentScan` is a Spring annotation that tells Spring where to search for classes annotated with stereotype annotations and register eligible classes as beans in the Spring ApplicationContext.

It commonly discovers classes annotated with:

- `@Component`
- `@Service`
- `@Repository`
- `@Controller`
- `@RestController`
- `@Configuration`

**In simple terms:** `@ComponentScan` helps Spring automatically discover application components instead of requiring you to register each component manually.

## 2. Example without `@ComponentScan`

Suppose the project has this structure:

```text
com.example.myapp
├── MyApplication.java
└── services
    └── UserService.java
```

The service is defined as:

```java
package com.example.myapp.services;

import org.springframework.stereotype.Service;

@Service
public class UserService {

    public String getMessage() {
        return "Hello from UserService!";
    }
}
```

If you create an application context without configuring component scanning or otherwise registering `UserService`, Spring will not automatically discover this class.

Attempting to inject it may result in a `NoSuchBeanDefinitionException` or an application startup failure.

## 3. Using `@ComponentScan`

You can explicitly specify the package to scan:

```java
package com.example.myapp;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("com.example.myapp.services")
public class MyConfig {
}
```

Spring scans `com.example.myapp.services` and its subpackages for eligible components, including `UserService`.

You can then inject the service:

```java
package com.example.myapp;

import org.springframework.stereotype.Component;

@Component
public class MessagePrinter {

    private final UserService userService;

    public MessagePrinter(UserService userService) {
        this.userService = userService;
    }

    public void printMessage() {
        System.out.println(userService.getMessage());
    }
}
```

For this example, `MessagePrinter` must also be discovered or registered. Since it is in `com.example.myapp`, you could scan the root package instead:

```java
@Configuration
@ComponentScan("com.example.myapp")
public class MyConfig {
}
```

This scans the root package and its subpackages, allowing Spring to discover both classes.

## 4. How does `@ComponentScan` work in Spring Boot?

In a typical Spring Boot application:

```java
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

`@SpringBootApplication` already includes `@ComponentScan`.

By default, Spring scans from the package containing `MyApplication` and its subpackages.

For example:

```text
com.example.myapp
├── MyApplication.java
├── controller
│   └── UserController.java
├── service
│   └── UserService.java
└── repository
    └── UserRepository.java
```

Because all these packages are under `com.example.myapp`, Spring can discover their eligible components without an explicit `@ComponentScan` declaration.

**Best practice:** Place the main Spring Boot application class in the root package of your application.

## 5. Important attributes of `@ComponentScan`

### A. `basePackages`

Specifies one or more package names to scan.

```java
@ComponentScan(basePackages = {
    "com.example.myapp.service",
    "com.example.myapp.repository"
})
```

### B. `basePackageClasses`

Specifies marker classes whose packages should be scanned. This is refactoring-friendly because it avoids hardcoding package names.

```java
@ComponentScan(basePackageClasses = {
    UserService.class,
    UserRepository.class
})
```

Spring scans the packages containing these classes and their subpackages.

### C. Default behavior

If no package is specified, Spring uses the package of the class declaring `@ComponentScan` as the starting point.

Note that `@ComponentScan` does not automatically scan every package in the entire project or every dependency on the classpath.

## 6. What happens if a component is outside the base package?

Consider this structure:

```text
com.example.myapp
├── MyApplication.java
└── service
    └── UserService.java

com.example.shared
└── SharedService.java
```

If the application class is in `com.example.myapp`, the default component scan does not cover `com.example.shared`.

One option is to specify both packages:

```java
@SpringBootApplication(
    scanBasePackages = {
        "com.example.myapp",
        "com.example.shared"
    }
)
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

Alternatively, use an explicit `@ComponentScan` configuration.

**Important:** `scanBasePackages` configures component scanning; it does not change the package used by Spring Boot's auto-configuration package registration. If auto-configuration needs to recognize additional packages, that may require separate configuration.

## 7. Difference between `@ComponentScan` and `@EnableAutoConfiguration`

| `@ComponentScan` | `@EnableAutoConfiguration` |
|---|---|
| Discovers application components in specified packages. | Applies suitable Spring Boot configurations based on dependencies, conditions, and properties. |
| Finds classes such as `@Service` and `@Controller`. | Can configure infrastructure such as web MVC components and data-source support. |
| Controls application component scanning. | Controls Spring Boot auto-configuration. |

Both are included in `@SpringBootApplication`.

## 8. Interview-ready answer

**Question: What is the use of `@ComponentScan`?**

`@ComponentScan` tells Spring which packages to scan for eligible components, such as classes annotated with `@Component`, `@Service`, `@Repository`, and `@Controller`. Spring registers the discovered components as beans in the application context, allowing dependency injection.

In Spring Boot, `@SpringBootApplication` already includes `@ComponentScan`. By default, scanning starts from the package containing the main application class and includes its subpackages. If components are located outside that hierarchy, I can specify additional packages using `basePackages` or `basePackageClasses`.

## Quick revision

- `@ComponentScan` discovers eligible Spring components.
- Discovered components are registered as Spring beans.
- The default scan starts from the declaring class's package and includes subpackages.
- `@SpringBootApplication` includes `@ComponentScan`.
- Use `basePackages` for explicit package names.
- Use `basePackageClasses` for type-safe package selection.
- Components outside the scan scope must be explicitly included or registered another way.


---

# What Does `@ResponseBody` Mean in Spring Boot?

## 1. What is `@ResponseBody`?

`@ResponseBody` is a Spring annotation that tells Spring MVC to write a controller method's return value directly to the HTTP response body instead of interpreting it as a view name.

Spring uses an appropriate `HttpMessageConverter` to convert the returned value into the response format. For example, Jackson commonly serializes Java objects into JSON when the relevant dependencies and content negotiation settings are available.

**In simple terms:** `@ResponseBody` returns data directly to the client rather than rendering a server-side view.

## 2. Example using `@ResponseBody`

```java
@Controller
public class UserController {

    @ResponseBody
    @GetMapping("/user")
    public User getUser() {
        return new User(1L, "Danish");
    }
}
```

Assuming a suitable `User` class and JSON message converter are available, a request to `GET /user` could return:

```json
{
  "id": 1,
  "name": "Danish"
}
```

### How does it work?

1. The client sends an HTTP request to `/user`.
2. Spring MVC invokes the `getUser()` method.
3. The method returns a Java `User` object.
4. An appropriate `HttpMessageConverter` serializes the object into the negotiated response format, commonly JSON.
5. Spring writes the serialized data to the HTTP response body.

Spring does not attempt to resolve a view named after the returned object.

## 3. Can `@ResponseBody` return a String?

Yes. The returned string is written directly to the response body.

```java
@Controller
public class MessageController {

    @ResponseBody
    @GetMapping("/message")
    public String getMessage() {
        return "Hello, World!";
    }
}
```

Response body:

```text
Hello, World!
```

Without `@ResponseBody`, a string returned from a conventional `@Controller` handler is generally treated as a view name.

## 4. Can we apply `@ResponseBody` at the class level?

Yes. When applied at the class level, it applies response-body behavior to all handler methods in that controller.

```java
@Controller
@ResponseBody
public class UserController {

    @GetMapping("/user")
    public User getUser() {
        return new User(1L, "Danish");
    }

    @GetMapping("/message")
    public String getMessage() {
        return "Hello, World!";
    }
}
```

Both methods write their return values directly to the response body.

## 5. Difference between `@ResponseBody` and `@RestController`

`@RestController` is a convenience annotation combining `@Controller` and `@ResponseBody`.

| Feature | `@ResponseBody` | `@RestController` |
|---|---|---|
| Type | Can annotate a method or class | Annotates a controller class |
| Purpose | Makes return values response bodies | Provides response-body behavior for controller methods by default |
| Typical use | Returning data from selected MVC handlers | Building REST APIs |
| View resolution | Bypassed for the annotated handler | Bypassed for ordinary handler return values |

Example:

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public User getUser() {
        return new User(1L, "Danish");
    }
}
```

This provides the same response-body behavior for the handler without requiring a separate `@ResponseBody` annotation.

## 6. Does `@ResponseBody` always return JSON?

No. It does not specifically mean JSON. Spring chooses an appropriate message converter based on the returned value, available converters, the request's `Accept` header, and the configured content negotiation.

Common examples include:

- Java object → JSON, when a suitable JSON converter is available.
- `String` → plain text in typical configurations.
- Other supported types → their corresponding response representation.

If no suitable converter is available for the return type and requested media type, Spring may return an HTTP error.

## 7. Interview-ready answer

**Question: What does `@ResponseBody` signify?**

`@ResponseBody` tells Spring MVC to write a controller method's return value directly to the HTTP response body instead of treating it as a view name. Spring uses an appropriate `HttpMessageConverter` to serialize the return value, commonly converting Java objects into JSON using Jackson.

It can be applied to an individual handler method or to a controller class. For REST APIs, `@RestController` is a convenient alternative because it combines `@Controller` and `@ResponseBody`.

## Quick revision

- `@ResponseBody` → writes the return value directly to the HTTP response body.
- It prevents ordinary view-name resolution for the annotated handler.
- `HttpMessageConverter` handles conversion of the return value.
- JSON is common, but the annotation does not guarantee JSON.
- `@RestController` = `@Controller` + `@ResponseBody`.


---

