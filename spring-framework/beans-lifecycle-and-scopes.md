---
title: Beans, Lifecycle and Scopes
parent: Spring Framework
nav_order: 2
---

# Beans, Lifecycle and Scopes
{: .no_toc }

Bean creation, lifecycle callbacks, cyclic dependencies and scopes.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

---
title: "Spring Bean Lifecycle, Scopes, Dependency Injection, and Thread Safety"
parent: "Core Java"
nav_order: 73
---

# Spring Bean Lifecycle, Scopes, Dependency Injection, and Thread Safety

A **Spring bean** is an object instantiated, configured, and managed by the Spring IoC container. Understanding its lifecycle, scopes, dependency injection mechanisms, and thread-safety requirements is essential for Spring Boot interviews.

## 1. What Is the Spring Bean Lifecycle?

The Spring bean lifecycle describes the stages a bean goes through, from creation to initialization and eventual destruction.

### Main lifecycle stages

1. **Instantiation:** Spring creates the bean using a constructor, factory method, or another supported instantiation mechanism.
2. **Dependency injection:** Spring supplies the bean's dependencies.
3. **Aware callbacks:** If implemented, interfaces such as `BeanNameAware` and `BeanFactoryAware` receive relevant container information.
4. **Before-initialization processing:** Registered `BeanPostProcessor` implementations can process the bean before initialization callbacks.
5. **Initialization callbacks:** Methods such as `@PostConstruct`, `InitializingBean.afterPropertiesSet()`, and a configured `initMethod` can run.
6. **After-initialization processing:** `BeanPostProcessor` implementations can further process or wrap the bean, for example with a proxy.
7. **Bean ready for use:** The fully initialized bean is available for application use.
8. **Destruction:** For beans managed through a supported container lifecycle, destruction callbacks can run when the container shuts down.

The exact sequence includes additional internal steps, and not every bean uses every callback.

### Example using lifecycle annotations

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class ReportService {

    public ReportService() {
        System.out.println("1. Constructor");
    }

    @PostConstruct
    public void initialize() {
        System.out.println("2. Initialization");
    }

    public void generateReport() {
        System.out.println("Generating report");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("3. Cleanup");
    }
}
```

For modern Spring applications using Jakarta annotations, `@PostConstruct` runs after dependency injection and before the bean is made available for normal use. `@PreDestroy` runs during the supported bean-destruction process.

Typical output:

```text
1. Constructor
2. Initialization
Generating report
3. Cleanup
```

The cleanup message appears only if the bean's destruction lifecycle is triggered, such as during a normal Spring application-context shutdown.

### Other lifecycle callbacks

| Callback | Purpose |
|---|---|
| `@PostConstruct` | Runs after dependency injection during initialization |
| `InitializingBean.afterPropertiesSet()` | Initialization callback implemented by a bean |
| `initMethod` | Configured custom initialization method |
| `@PreDestroy` | Runs before bean destruction |
| `DisposableBean.destroy()` | Destruction callback implemented by a bean |
| `destroyMethod` | Configured custom destruction method |
| `BeanPostProcessor` | Processes beans before and after initialization |

Example of a configured initialization and destruction method:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
class AppConfig {

    @Bean(initMethod = "start", destroyMethod = "stop")
    WorkerService workerService() {
        return new WorkerService();
    }
}

class WorkerService {

    public void start() {
        System.out.println("Worker started");
    }

    public void stop() {
        System.out.println("Worker stopped");
    }
}
```

**Important:** Spring generally manages the full destruction lifecycle for singleton beans. Prototype beans receive initialization processing, but Spring does not automatically invoke their configured destruction callbacks after handing them to the client.

## 2. What Are Spring Bean Scopes?

A **bean scope** defines how many instances Spring creates and the context in which those instances are shared.

### Types of bean scopes

| Scope | Behavior | Typical use |
|---|---|---|
| `singleton` | One instance per bean definition per Spring IoC container | Stateless services and repositories |
| `prototype` | A new instance each time the bean is requested from the container | Objects requiring separate mutable state |
| `request` | One instance per HTTP request | Request-specific data |
| `session` | One instance per HTTP session | User-session state |
| `application` | One instance per `ServletContext` | Web-application-wide state |
| `websocket` | One instance per WebSocket session | WebSocket-specific state |

The `request`, `session`, `application`, and `websocket` scopes require an appropriate web application environment.

### Singleton scope

Singleton is the default Spring bean scope.

```java
import org.springframework.stereotype.Service;

@Service
public class NotificationService {
    public void send(String message) {
        System.out.println(message);
    }
}
```

Spring normally creates one `NotificationService` instance per bean definition in its container. Multiple components can share that instance.

A Spring singleton is **not necessarily a JVM-wide singleton**. Separate application contexts can each have their own instance.

### Prototype scope

Use `@Scope("prototype")` when a new bean instance is needed for each container lookup.

```java
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("prototype")
public class TaskContext {

    private String taskName;

    public String getTaskName() {
        return taskName;
    }

    public void setTaskName(String taskName) {
        this.taskName = taskName;
    }
}
```

Requesting this bean twice from the container ordinarily produces two different instances.

```java
TaskContext first = context.getBean(TaskContext.class);
TaskContext second = context.getBean(TaskContext.class);

System.out.println(first == second); // false
```

Here, `context` is an initialized Spring `ApplicationContext`.

**Interview trap:** Injecting a prototype bean directly into a singleton bean does not automatically create a fresh prototype on every method call. The prototype is normally resolved when the singleton is created. For a new instance per operation, consider `ObjectProvider<TaskContext>`, a provider, or an appropriate scoped proxy.

### Request scope

A request-scoped bean is associated with one HTTP request.

```java
import org.springframework.context.annotation.RequestScope;
import org.springframework.stereotype.Component;

@Component
@RequestScope
public class RequestContext {

    private String correlationId;

    public String getCorrelationId() {
        return correlationId;
    }

    public void setCorrelationId(String correlationId) {
        this.correlationId = correlationId;
    }
}
```

A new `RequestContext` is associated with each request. It is useful for request-specific information such as a correlation ID, but request state should not be treated as globally shared application state.

## 3. What Is a Stateless Bean in Spring?

A **stateless bean** does not retain request-specific or invocation-specific mutable state in its instance fields between method calls.

Example:

```java
import org.springframework.stereotype.Service;

@Service
public class CalculatorService {

    public int add(int a, int b) {
        return a + b;
    }
}
```

The method uses local variables and its arguments; it does not store the result in shared mutable instance state.

Stateless service beans are commonly implemented using Spring's default singleton scope.

### Stateless vs stateful beans

| Stateless bean | Stateful bean |
|---|---|
| Does not retain per-invocation mutable state in instance fields | Retains mutable state between invocations |
| Easier to share between concurrent requests | Requires careful state isolation or synchronization |
| Often suitable for singleton scope | May require a different scope or a thread-safe design |
| Example: calculation service | Example: object storing a mutable workflow state |

A stateless singleton can safely serve concurrent calls **provided its dependencies and other shared resources are also used safely**. Statelessness alone does not guarantee that every operation is thread-safe.

Also, a stateless Spring bean is not automatically an EJB stateless session bean; these are different concepts.

## 4. How Is a Bean Injected in Spring?

Spring supports several dependency injection mechanisms.

### A. Constructor injection — recommended

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

Spring resolves the `PaymentService` bean and passes it into the `OrderService` constructor. When a class has one constructor, `@Autowired` is generally unnecessary.

**Advantages:**
- Supports immutable dependencies using `final` fields.
- Makes required dependencies explicit.
- Simplifies unit testing.
- Prevents the object from being constructed without its required dependencies.

### B. Setter injection

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
class ReportController {

    private ReportService reportService;

    @Autowired
    public void setReportService(ReportService reportService) {
        this.reportService = reportService;
    }
}
```

Useful for optional or changeable dependencies, though changing dependencies after initialization should be considered carefully.

### C. Field injection

```java
@Component
class ReportController {

    @Autowired
    private ReportService reportService;
}
```

Spring supports field injection, but constructor injection is generally preferred because dependencies are less explicit and ordinary unit tests cannot initialize the field without reflection or Spring.

### Is interface injection supported by Spring?

The phrase **interface injection** is not one of Spring's standard dependency injection mechanisms. A class may implement an interface, but that alone does not inject a dependency.

A common alternative is to inject an interface through a constructor:

```java
interface PaymentGateway {
    void pay();
}

@Service
class CardPaymentGateway implements PaymentGateway {
    public void pay() {
        System.out.println("Card payment");
    }
}

@Service
class CheckoutService {

    private final PaymentGateway gateway;

    CheckoutService(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    public void checkout() {
        gateway.pay();
    }
}
```

Spring can inject the `PaymentGateway` implementation. If multiple beans implement the interface, use `@Qualifier`, `@Primary`, or another explicit selection strategy.

### How does autowiring work?

Spring resolves dependencies primarily by type. If multiple candidates match, qualifiers, primary candidates, bean names, and other resolution rules can help determine which bean to inject. If the dependency remains ambiguous, Spring can report an error.

## 5. Can a Singleton Bean Handle Multiple Parallel Requests?

**Yes.** Spring's singleton scope does not serialize method calls. Multiple request-handling threads can invoke methods on the same singleton bean concurrently.

Consider this unsafe example:

```java
@Service
public class CounterService {

    private int count = 0;

    public int increment() {
        return ++count;
    }
}
```

Two threads can access and update `count` at the same time. Because incrementing an integer is a read-modify-write operation, updates can be lost.

A thread-safe alternative for a simple counter is:

```java
import java.util.concurrent.atomic.AtomicInteger;
import org.springframework.stereotype.Service;

@Service
public class CounterService {

    private final AtomicInteger count = new AtomicInteger();

    public int increment() {
        return count.incrementAndGet();
    }
}
```

`AtomicInteger` makes this increment operation atomic. More complex invariants may require synchronization, locks, or another concurrency design.

### Best practices for singleton beans

- Prefer stateless services.
- Keep request-specific data in local variables or appropriately scoped objects, not shared mutable fields.
- Use thread-safe structures or synchronization when shared mutable state is necessary.
- Use database transactions for appropriate database consistency requirements; `@Transactional` does not automatically make arbitrary Java fields thread-safe.
- Ensure that injected dependencies are also safe for concurrent use.

## 6. Quick Interview Questions

**Q1. What is the default Spring bean scope?**

`singleton` — one instance per bean definition per Spring IoC container.

**Q2. Does Spring automatically destroy prototype beans?**

No. Spring creates and initializes prototype beans but generally does not manage their complete destruction lifecycle after returning them to the caller.

**Q3. When does `@PostConstruct` execute?**

After dependency injection, during bean initialization, before the bean is normally made available for use.

**Q4. Can a singleton bean handle multiple requests simultaneously?**

Yes. Multiple threads can call the same singleton instance concurrently. Shared mutable state and its dependencies must be designed for thread safety.

**Q5. Is a singleton bean always thread-safe?**

No. Singleton describes instance scope, not thread safety.

**Q6. What happens when a prototype bean is injected directly into a singleton?**

The prototype is ordinarily resolved once when the singleton is constructed. Repeatedly calling the singleton's method does not automatically produce a new prototype instance.

**Q7. Which dependency injection approach is preferred?**

Constructor injection is generally preferred for required dependencies because it makes dependencies explicit, supports immutability, and improves testability.

## One-Line Interview Answer

The Spring bean lifecycle covers instantiation, dependency injection, initialization, use, and destruction; bean scopes determine instance sharing, and singleton beans can handle concurrent requests safely when shared mutable state and dependencies are properly managed.


---



---
title: "Circular Dependencies in Spring: Eager vs Lazy Initialization"
parent: "Core Java"
nav_order: 74
---

# Circular Dependencies in Spring: Eager vs Lazy Initialization

A **circular dependency** occurs when two or more Spring beans depend on one another, directly or indirectly.

For example:

- Bean A depends on Bean B.
- Bean B depends on Bean A.

Spring must resolve these dependencies while creating and initializing the beans. Whether it can resolve the cycle depends on the injection type, bean configuration, and Spring version.

## 1. What Is a Circular Dependency?

Consider two services that depend on each other.

```java
@Service
class BeanA {

    private final BeanB beanB;

    BeanA(BeanB beanB) {
        this.beanB = beanB;
    }
}
```

```java
@Service
class BeanB {

    private final BeanA beanA;

    BeanB(BeanA beanA) {
        this.beanA = beanA;
    }
}
```

Spring needs `BeanB` to construct `BeanA`, but it also needs `BeanA` to construct `BeanB`.

This creates a constructor dependency cycle.

### What happens?

Spring cannot fully construct either bean first. With constructor injection, the cycle normally results in a `BeanCurrentlyInCreationException` or a related bean-creation failure, often reported as an `UnsatisfiedDependencyException`.

**Important:** Constructor injection does not solve circular dependencies. It makes required dependencies explicit and allows Spring to detect the unresolvable cycle.

## 2. How Does Spring Handle Circular Dependencies?

The behavior depends on the injection approach.

| Dependency type | Typical behavior |
|---|---|
| Constructor injection | Circular dependency cannot normally be resolved |
| Setter injection | May be resolvable for eligible singleton beans |
| Field injection | May be resolvable for eligible singleton beans |
| `@Lazy` injection | Can defer resolving one dependency until it is needed |
| `ObjectProvider<T>` | Can look up a dependency on demand |
| Redesigning the dependency graph | Usually the cleanest long-term solution |

Spring Framework can resolve certain circular references involving singleton beans by exposing an early reference before the bean is fully initialized. This is not universally supported for every bean type or configuration.

In particular, Spring Boot has disabled circular references by default since Spring Boot 2.6. Applications should not depend on automatic circular-reference resolution.

## 3. Solution 1: Use `@Lazy`

The `@Lazy` annotation can defer bean initialization or dependency resolution until the bean is needed.

```java
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Service;

@Service
class BeanA {

    private final BeanB beanB;

    BeanA(@Lazy BeanB beanB) {
        this.beanB = beanB;
    }
}
```

```java
import org.springframework.stereotype.Service;

@Service
class BeanB {

    private final BeanA beanA;

    BeanB(BeanA beanA) {
        this.beanA = beanA;
    }
}
```

Here, Spring can inject a lazy-resolution proxy for `BeanB` into `BeanA`. That allows `BeanA` to be constructed without immediately requiring the fully initialized `BeanB`.

The dependency is resolved when the proxy is used.

**Caution:** `@Lazy` can break a creation-time cycle, but it does not remove the underlying architectural dependency. The dependency may still fail when first accessed if the configuration is invalid.

## 4. Solution 2: Use `ObjectProvider`

`ObjectProvider<T>` is useful when a dependency should be resolved only when needed.

```java
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.stereotype.Service;

@Service
class BeanA {

    private final ObjectProvider<BeanB> beanBProvider;

    BeanA(ObjectProvider<BeanB> beanBProvider) {
        this.beanBProvider = beanBProvider;
    }

    public void execute() {
        BeanB beanB = beanBProvider.getObject();
        beanB.process();
    }
}
```

```java
import org.springframework.stereotype.Service;

@Service
class BeanB {

    public void process() {
        System.out.println("Processing");
    }
}
```

Spring injects the provider rather than requiring `BeanB` immediately during construction.

This can defer dependency resolution, but it does not guarantee that every circular design is safe. Avoid calling back into the same dependency cycle during initialization.

## 5. Solution 3: Use Setter Injection

For eligible singleton beans, setter injection can allow Spring to instantiate objects before completing their dependency wiring.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
class BeanA {

    private BeanB beanB;

    @Autowired
    public void setBeanB(BeanB beanB) {
        this.beanB = beanB;
    }
}
```

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
class BeanB {

    private BeanA beanA;

    @Autowired
    public void setBeanA(BeanA beanA) {
        this.beanA = beanA;
    }
}
```

Because the objects can be instantiated before all dependencies are supplied, Spring may resolve this kind of singleton cycle when circular references are allowed.

However, **setter injection is not a universal solution**. Resolution depends on container configuration and bean characteristics. In Spring Boot versions that disallow circular references by default, this cycle may still fail.

## 6. Solution 4: Redesign the Dependency Graph

Usually, the best long-term fix is to remove the circular dependency.

For example, if both services need shared functionality, extract that functionality into a third service.

Before:

```text
BeanA <----> BeanB
```

After:

```text
BeanA ----> SharedService <---- BeanB
```

The exact direction of dependencies should reflect the application's design; the goal is to remove the direct cycle.

This approach improves separation of concerns, testability, and maintainability without relying on proxies or container-specific circular-reference behavior.

## 7. What Are Eager and Lazy Initialization?

### Eager initialization

By default, Spring eagerly creates non-lazy singleton beans during application-context startup.

```java
@Component
class ReportService {
    ReportService() {
        System.out.println("ReportService created");
    }
}
```

When the application context starts, Spring normally creates this singleton even if no application code has requested it yet.

**Benefits:**
- Many configuration and dependency problems are detected at startup.
- Beans are ready when first needed.

**Trade-off:** Startup can take longer, and beans that are never used may still be created.

### Lazy initialization

Using `@Lazy` defers initialization until the bean is first requested, subject to how it is injected or accessed.

```java
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;

@Component
@Lazy
class ExpensiveService {

    ExpensiveService() {
        System.out.println("ExpensiveService created");
    }
}
```

The bean is normally created when first requested from the container or when a non-lazy bean first requires it. A lazy bean injected directly into an eager bean is not necessarily deferred unless the injection is also configured for lazy resolution, for example with `@Lazy` on the injection point.

### Eager vs lazy initialization

| Feature | Eager initialization | Lazy initialization |
|---|---|---|
| Creation time | Usually during application-context startup for singleton beans | When first needed |
| Startup time | May be longer | May be shorter |
| Error discovery | Many creation errors appear at startup | Some errors may appear on first use |
| Resource usage | Resources may be allocated even if unused | Unused lazy beans may never be created |
| Annotation | Default for singleton beans | `@Lazy` |

## 8. How Does Spring Manage Bean Creation?

At a high level:

1. Spring loads bean definitions from component scanning, configuration classes, and other sources.
2. It creates eager singleton beans during context startup.
3. It resolves dependencies through constructor, setter, field, or other supported injection mechanisms.
4. It initializes beans and applies relevant lifecycle callbacks and post-processors.
5. It makes initialized beans available for normal use.

For supported singleton circular references, Spring may expose an early reference while a bean is still being created. That mechanism is not equivalent to completing the bean's initialization early, and it cannot solve every cycle.

## 9. Common Interview Questions

**Q1. Can Spring resolve a circular dependency with constructor injection?**

Normally, no. Each constructor requires the other bean before either can finish construction.

**Q2. Can setter injection resolve circular dependencies?**

It can for eligible singleton beans when circular references are supported and enabled, but it is not guaranteed in every configuration.

**Q3. What is the purpose of `@Lazy`?**

It defers bean initialization or dependency resolution until the bean is required. Applied at an injection point, it can provide a lazy-resolution proxy that helps break a creation-time cycle.

**Q4. What is the difference between `@Lazy` and `ObjectProvider`?**

`@Lazy` can defer dependency resolution through a proxy. `ObjectProvider` gives the application an explicit way to request the dependency when needed.

**Q5. Are Spring singleton beans eager by default?**

Yes. Non-lazy singleton beans are normally initialized during application-context startup.

**Q6. Does lazy initialization always improve performance?**

No. It can reduce startup work, but it can also defer expensive initialization and failures until runtime. It does not necessarily reduce total work.

**Q7. What is the best way to fix circular dependencies?**

Prefer redesigning the classes so that their dependencies are not cyclic. Use lazy resolution or setter injection only when the dependency relationship is justified and the trade-offs are understood.

## One-Line Interview Answer

A circular dependency occurs when Spring beans depend on each other; constructor cycles normally fail, eligible singleton cycles may be resolvable through setter or field injection when supported, and `@Lazy` or `ObjectProvider` can defer dependency resolution, while redesigning the dependency graph is usually the cleanest solution. Eager initialization creates non-lazy singleton beans at startup, whereas lazy initialization defers creation until needed.


---


## 1. Is a Singleton Bean Thread-Safe in Spring?

**No. A Spring singleton bean is not automatically thread-safe.** Singleton scope means that Spring maintains one shared instance per bean definition per container. Multiple request-processing threads can access that same instance concurrently.

### Why can a singleton bean have thread-safety issues?

A singleton bean that is stateless—meaning it does not keep request-specific mutable state in instance fields—is generally safe for concurrent use, provided its dependencies are also safe to use concurrently.

**Example: Stateless singleton bean**

```java
@Service
public class CalculatorService {

    public int add(int a, int b) {
        return a + b;
    }
}
```

This service is generally thread-safe because each invocation uses its own method-local variables and does not modify shared state.

**Example: Stateful singleton bean**

```java
@Service
public class CounterService {

    private int count = 0;

    public int increment() {
        return ++count;
    }
}
```

If multiple threads call `increment()` concurrently, race conditions can occur because `++count` is not an atomic operation.

### How can you make a singleton bean thread-safe?

**Option 1: Use `AtomicInteger`**

```java
@Service
public class CounterService {

    private final AtomicInteger count = new AtomicInteger();

    public int increment() {
        return count.incrementAndGet();
    }

    public int getCount() {
        return count.get();
    }
}
```

**Option 2: Use synchronization**

```java
public synchronized int increment() {
    return ++count;
}
```

**Option 3: Avoid shared mutable state**

Prefer stateless services and keep request-specific data in method-local variables.

### Does `@Transactional` make a singleton bean thread-safe?

No. `@Transactional` manages transaction boundaries; it does not make ordinary Java fields thread-safe. Database consistency and in-memory thread safety are separate concerns.

**Interview answer:** "Spring singleton scope guarantees a shared instance, not thread safety. Stateless singleton services are usually safe for concurrent requests, but mutable shared state requires appropriate concurrency controls such as atomic classes, synchronization, or locks."

---

## 2. How Does the Factory Design Pattern Work in Spring?

The Factory pattern encapsulates object creation. Spring uses factory-based abstractions to create beans, inject dependencies, and manage their lifecycle.

### A. `BeanFactory` and `ApplicationContext`

- `BeanFactory` is a core Spring IoC container interface.
- `ApplicationContext` extends `BeanFactory` and provides additional capabilities, including event publication and resource access.

Example:

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

MyService service = context.getBean(MyService.class);
```

Spring creates and manages the bean rather than requiring application code to construct all dependencies manually.

### B. Using `@Bean` as a factory method

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

The `@Bean` method tells Spring how to construct the object to register in the application context.

Spring manages the returned object according to its bean definition, including dependency resolution and applicable lifecycle callbacks.

### C. Using `FactoryBean`

`FactoryBean<T>` is useful when creating an object requires specialized construction logic.

```java
public class MyServiceFactory
        implements FactoryBean<MyService> {

    @Override
    public MyService getObject() {
        return new MyService();
    }

    @Override
    public Class<?> getObjectType() {
        return MyService.class;
    }
}
```

Spring exposes the object returned by `getObject()` as the product of the factory bean.

**Important distinction:**

- `context.getBean("myService")` retrieves the product of a `FactoryBean` registered under that name.
- `context.getBean("&myService")` retrieves the `FactoryBean` itself, assuming that bean is registered under `myService`.

**Interview answer:** "Spring uses factory-based abstractions such as `BeanFactory`, `ApplicationContext`, `@Bean` factory methods, and `FactoryBean` to create and manage objects. `FactoryBean` is especially useful when a bean requires custom construction logic."

---

## 3. How Is the Proxy Design Pattern Used in Spring?

The Proxy pattern provides a substitute object that controls access to another object. Spring uses proxies extensively for features such as AOP, declarative transactions, caching, and asynchronous method execution.

### How does it work?

```text
Client
  |
  v
Spring Proxy
  |
  +--> Logging
  |
  +--> Transaction handling
  |
  +--> Security / other advice
  |
  v
Target Bean
```

When a method call passes through the proxy, Spring can execute additional behavior before, after, or around the target method invocation.

### Example: Logging with Spring AOP

```java
@Aspect
@Component
public class LoggingAspect {

    @Before(
        "execution(* com.example.service.*.*(..))"
    )
    public void logBefore(JoinPoint joinPoint) {
        System.out.println(
            "Calling method: "
            + joinPoint.getSignature().getName()
        );
    }
}
```

Enable AspectJ-style Spring AOP proxy support in the application:

```java
@Configuration
@EnableAspectJAutoProxy
public class AopConfig {
}
```

Spring Boot applications can also configure AOP through their auto-configuration and dependencies. The logging aspect is applied when an eligible service invocation passes through the Spring AOP proxy.

### Types of proxies used by Spring

| Proxy type | How it works |
|---|---|
| JDK dynamic proxy | Implements the target's interfaces |
| CGLIB proxy | Creates a subclass of the target class |
| AspectJ weaving | Applies aspects through bytecode weaving rather than relying exclusively on Spring's runtime proxy mechanism |

**Important corrections:**

- Spring AOP commonly uses JDK dynamic proxies or CGLIB-based proxies.
- AspectJ weaving is not simply a third equivalent runtime proxy type; it is a separate aspect-application mechanism.
- JDK dynamic proxies are not always the default in every configuration. Spring Boot commonly uses class-based proxies by default, depending on the configuration.
- Proxy-based advice generally applies only to calls that pass through the proxy. A method calling another method on the same object (`this.someMethod()`) bypasses the proxy.

**Interview answer:** "Spring uses the Proxy pattern to apply cross-cutting concerns without placing them directly inside business logic. For example, `@Transactional` uses a Spring proxy to start and complete a transaction around an eligible method invocation."

---

## 4. What Happens When a Singleton Bean Depends on a Prototype Bean, or Vice Versa?

The result depends on **when the dependency is resolved**.

### Case 1: Prototype bean depends on a singleton bean

```java
@Component
@Scope("prototype")
public class PrototypeBean {

    private final SingletonBean singletonBean;

    public PrototypeBean(SingletonBean singletonBean) {
        this.singletonBean = singletonBean;
    }
}
```

```java
@Component
public class SingletonBean {
}
```

Every time Spring creates a new `PrototypeBean`, it injects the same singleton instance of `SingletonBean`.

Example:

```java
PrototypeBean first =
        context.getBean(PrototypeBean.class);

PrototypeBean second =
        context.getBean(PrototypeBean.class);
```

- `first` and `second` are different prototype instances.
- Both reference the same singleton dependency.

### Case 2: Singleton bean depends directly on a prototype bean

```java
@Component
@Scope("prototype")
public class PrototypeBean {
}
```

```java
@Component
public class SingletonBean {

    private final PrototypeBean prototypeBean;

    public SingletonBean(PrototypeBean prototypeBean) {
        this.prototypeBean = prototypeBean;
    }

    public PrototypeBean getPrototypeBean() {
        return prototypeBean;
    }
}
```

When Spring creates the singleton, it resolves the prototype dependency once and injects that instance.

```java
SingletonBean singleton =
        context.getBean(SingletonBean.class);

PrototypeBean first = singleton.getPrototypeBean();
PrototypeBean second = singleton.getPrototypeBean();
```

Here, `first == second` is `true`, because both calls return the same reference held by the singleton.

**Crucial correction:** Spring does not automatically create a new prototype instance every time a singleton method is called. The prototype is created when the dependency is resolved—not on every method invocation.

### How do you get a fresh prototype instance each time?

Use `ObjectProvider<PrototypeBean>`:

```java
@Component
public class SingletonBean {

    private final ObjectProvider<PrototypeBean> provider;

    public SingletonBean(ObjectProvider<PrototypeBean> provider) {
        this.provider = provider;
    }

    public PrototypeBean getPrototypeBean() {
        return provider.getObject();
    }
}
```

Now, each call to `getPrototypeBean()` asks Spring for a prototype instance, resulting in a new instance for each lookup under normal prototype scope behavior.

Alternatively, Spring supports method injection with `@Lookup` for suitable use cases.

### Comparison

| Dependency relationship | Result |
|---|---|
| Prototype → Singleton | Each prototype instance receives the shared singleton |
| Singleton → Prototype, direct injection | The singleton retains the prototype instance injected at creation |
| Singleton → Prototype through `ObjectProvider` | A fresh prototype is obtained for each provider lookup |

**Interview answer:** "A singleton bean injected into a prototype bean remains the same shared instance. A prototype injected directly into a singleton is created when the singleton is initialized and remains the same dependency. If the singleton needs a fresh prototype on demand, I use `ObjectProvider` or method injection."

---

## 5. How Can You Create a Prototype Bean?

A prototype-scoped bean is created by Spring whenever a new instance is requested from the container. Unlike singleton beans, prototype beans are not maintained as one shared instance.

### A. Using Java configuration

```java
@Configuration
public class AppConfig {

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public PrototypeBean prototypeBean() {
        return new PrototypeBean();
    }
}
```

Retrieve it from the container:

```java
PrototypeBean first =
        context.getBean(PrototypeBean.class);

PrototypeBean second =
        context.getBean(PrototypeBean.class);

System.out.println(first == second); // false
```

Each lookup creates a new instance.

### B. Using component scanning

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class PrototypeBean {
}
```

Spring discovers and registers the bean through component scanning.

### C. Using XML configuration

```xml
<bean id="prototypeBean"
      class="com.example.PrototypeBean"
      scope="prototype" />
```

### Important lifecycle difference

For prototype beans, Spring creates the instance and performs dependency injection and initialization callbacks. However, Spring does **not** automatically manage the complete destruction lifecycle after handing the instance to the caller. The caller is generally responsible for releasing resources acquired by the prototype bean.

**Interview answer:** "I can create a prototype bean using `@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)` on a component or `@Bean` method. Each request to the Spring container creates a new instance. If I inject it directly into a singleton, however, the singleton keeps the instance injected during its own creation."



---

