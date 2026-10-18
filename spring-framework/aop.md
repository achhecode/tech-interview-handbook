---
title: Aspect-Oriented Programming (AOP)
parent: Spring Framework
nav_order: 4
---

# Aspect-Oriented Programming (AOP)
{: .no_toc }

Cross-cutting concerns, advice types and proxies.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

# Spring AOP (Aspect-Oriented Programming): Concepts, Annotations, and Pointcuts

## 1. What is AOP?

**Aspect-Oriented Programming (AOP)** is a programming approach that separates cross-cutting concerns from the main business logic of an application.

Cross-cutting concerns are functionalities needed across multiple classes or methods, such as:

- Logging
- Security and authorization
- Transaction management
- Performance monitoring
- Auditing
- Exception monitoring

Without AOP, we might repeat the same logging or timing code in many service methods. AOP allows us to define this behavior once and apply it to selected methods.

### Example without AOP

```java
public void createEmployee() {
    System.out.println("Method started");

    // Business logic
    System.out.println("Creating employee");

    System.out.println("Method completed");
}
```

If every service method contains similar logging code, it creates duplication.

With AOP, logging can be moved into a separate aspect, leaving the service focused on business logic.

## 2. Important AOP terminology

| Concept | Meaning |
|---|---|
| Aspect | A module that encapsulates a cross-cutting concern, such as logging. |
| Join point | A point in program execution where advice can apply. In Spring AOP, this is a method execution on a Spring bean. |
| Advice | The action performed by an aspect at a selected join point. |
| Pointcut | An expression that selects the join points where advice should run. |
| Target object | The object whose method is intercepted. |
| Proxy | An object Spring uses to intercept method calls and apply additional behavior. |
| Weaving | The process of combining aspects with the target program. Spring AOP typically does this through runtime proxies. |

**Interview tip:** Spring AOP primarily supports method-execution join points. AspectJ supports additional join-point types, such as field access and constructor execution.

## 3. Types of advice in Spring AOP

Spring AOP provides five commonly used advice annotations.

| Annotation | When it executes |
|---|---|
| `@Before` | Before the target method executes. |
| `@After` | After the method finishes, whether normally or exceptionally. |
| `@AfterReturning` | After the method completes successfully. |
| `@AfterThrowing` | When the method exits by throwing an exception. |
| `@Around` | Wraps the method execution and can control whether and when the target method proceeds. |

### Example: Using AOP for logging

First, enable AOP support by adding the `spring-boot-starter-aop` dependency to your Spring Boot project.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

Create a service:

```java
@Service
public class EmployeeService {

    public String getEmployee() {
        return "Employee details";
    }

    public void deleteEmployee() {
        System.out.println("Deleting employee");
    }
}
```

Now create an aspect:

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {

    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {
    }

    @Before("serviceMethods()")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println(
            "Starting: " + joinPoint.getSignature().getName()
        );
    }

    @AfterReturning(
        pointcut = "serviceMethods()",
        returning = "result"
    )
    public void logAfterReturning(
            JoinPoint joinPoint, Object result) {
        System.out.println(
            "Completed: " + joinPoint.getSignature().getName()
        );
    }

    @After("serviceMethods()")
    public void logAfter(JoinPoint joinPoint) {
        System.out.println(
            "Execution finished: " + joinPoint.getSignature().getName()
        );
    }

    @AfterThrowing(
        pointcut = "serviceMethods()",
        throwing = "exception"
    )
    public void logException(
            JoinPoint joinPoint, Throwable exception) {
        System.out.println(
            "Failed: " + joinPoint.getSignature().getName()
        );
    }
}
```

The package `com.example.service` is an example; replace it with your application's actual service package.

Here, `serviceMethods()` defines a reusable pointcut. The advice methods reference it so that logging is applied consistently to the selected service methods.

Note that `@After` runs after both successful and exceptional completion. It does not mean only after success.

## 4. How do you define pointcuts?

A **pointcut** specifies which method executions should be intercepted by an aspect.

Spring AOP supports several pointcut designators, including `execution`, `within`, `this`, `target`, `args`, `@annotation`, `@within`, and `@target`.

### A. `execution()` — Match method signatures

Syntax:

```java
execution(modifiers-pattern? return-type-pattern
          declaring-type-pattern? method-name-pattern
          (parameters-pattern) throws-pattern?)
```

Example:

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void allServiceMethods() {
}
```

This selects method executions for methods declared in types directly within `com.example.service`.

Understand the expression:

- First `*`: Any return type.
- `com.example.service.*`: Types directly in the package.
- Second `*`: Any method name.
- `(..)`: Any number and type of parameters.

To include service subpackages, use:

```java
@Pointcut("execution(* com.example.service..*(..))")
public void allServiceMethodsIncludingSubpackages() {
}
```

### B. `within()` — Match types in a package

```java
@Pointcut("within(com.example.service..*)")
public void servicePackage() {
}
```

This matches join points associated with types in the specified package and its subpackages.

### C. `@annotation()` — Match methods by annotation

Suppose we have a custom annotation:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Audited {
}
```

Apply it to a method:

```java
@Audited
public void transferMoney() {
    // Business logic
}
```

Define the pointcut:

```java
@Pointcut("@annotation(com.example.annotation.Audited)")
public void auditedMethods() {
}
```

Now the aspect can apply auditing only to methods annotated with `@Audited`.

### D. `args()` — Match runtime argument types

```java
@Pointcut("args(String, ..)")
public void methodsWithStringFirstArgument() {
}
```

This matches method executions whose runtime arguments have a `String` as the first argument, followed by zero or more additional arguments.

### E. Combine pointcuts

You can combine expressions using logical operators:

```java
@Pointcut("execution(* com.example.service..*(..))")
public void serviceMethods() {
}

@Pointcut("@annotation(com.example.annotation.Audited)")
public void auditedMethods() {
}

@Before("serviceMethods() && auditedMethods()")
public void auditServiceMethods(JoinPoint joinPoint) {
    System.out.println("Auditing " + joinPoint.getSignature());
}
```

This selects methods satisfying both conditions: they are in the service package hierarchy and carry the `@Audited` annotation.

## 5. What is `@Around` advice?

`@Around` is especially useful for measuring execution time because it runs both before and after the target invocation and can inspect the result or exception.

```java
@Aspect
@Component
public class PerformanceAspect {

    @Around("execution(* com.example.service..*(..))")
    public Object measureExecutionTime(
            ProceedingJoinPoint joinPoint) throws Throwable {

        long start = System.nanoTime();

        try {
            return joinPoint.proceed();
        } finally {
            long elapsed = System.nanoTime() - start;

            System.out.println(
                joinPoint.getSignature().getName()
                + " took " + elapsed + " ns"
            );
        }
    }
}
```

Important points:

- `ProceedingJoinPoint` is used with `@Around` advice.
- `joinPoint.proceed()` invokes the underlying method.
- Returning its result preserves the method's return value.
- The `finally` block measures elapsed time even if the method throws.
- If `proceed()` is never called, the target method does not execute.

Use `@Around` only when you need control over the invocation. Simpler advice types are usually easier to understand and maintain.

## 6. Important limitations of Spring AOP

Spring AOP generally works through proxies, so there are a few limitations to remember:

1. **Self-invocation:** A method calling another method on the same object typically bypasses the proxy, so the internal call may not trigger the second method's advice.
2. **Proxy-based interception:** Only calls that pass through the Spring proxy are intercepted.
3. **Private methods:** Private methods are not intercepted through standard Spring AOP proxies.
4. **Aspect registration:** The aspect must be registered as a Spring bean, and AOP support must be configured, typically through the Spring Boot AOP starter.
5. **Weaving:** Spring AOP normally uses runtime proxies; full AspectJ weaving can support a broader set of join points.

Also, Spring's declarative transaction management commonly uses AOP proxies, which is why self-invocation can affect `@Transactional` behavior.

## 7. Interview-ready answer

**Question: Describe AOP, its annotations, and how you define pointcuts.**

AOP stands for Aspect-Oriented Programming. It helps separate cross-cutting concerns such as logging, security, auditing, and performance monitoring from core business logic.

Its main concepts are aspects, join points, advice, pointcuts, proxies, and weaving. In Spring AOP, join points are primarily method executions on Spring beans.

The main advice annotations are `@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`, and `@Around`. We define pointcuts using expressions such as `execution()`, `within()`, and `@annotation()`. For example, `execution(* com.example.service..*(..))` selects method executions in a service package and its subpackages.

We can define a reusable pointcut with `@Pointcut` and reference it from multiple advice methods. Spring AOP typically applies the advice through runtime proxies, so self-invocation and calls that bypass the proxy are important limitations.

## Quick revision

- `@Aspect`: Declares an aspect.
- `@Pointcut`: Defines which join points match.
- `@Before`: Runs before the target method.
- `@After`: Runs after completion, including exceptional completion.
- `@AfterReturning`: Runs after successful completion.
- `@AfterThrowing`: Runs when an exception escapes.
- `@Around`: Wraps and can control method execution.
- `execution()`: Matches method signatures.
- `within()`: Matches types within a package or type scope.
- `@annotation()`: Matches methods with a particular annotation.
- **Spring AOP:** Proxy-based, primarily method-execution interception.


---


 
---
title: Design Pattern Used by Spring AOP
parent: Design Patterns and LLD
nav_order: 15
---

# Which Design Pattern Is Used by Spring AOP?
{: .no_toc }

Spring AOP primarily uses the **Proxy Design Pattern** to apply cross-cutting concerns to Spring-managed objects without modifying their business logic.
{: .fs-6 .fw-300 }

## 1. Short Interview Answer

Spring AOP primarily uses the **Proxy Design Pattern**, not the Decorator Pattern.

Spring creates a proxy object around a target bean. When a method is called through this proxy, Spring can execute additional logic—such as logging, security checks, transaction management, or performance monitoring—before or after invoking the actual target method.

Spring AOP typically uses **JDK dynamic proxies** for beans implementing interfaces and **CGLIB-based subclass proxies** when class-based proxying is needed.

Although Proxy and Decorator have similar structures, their primary intentions differ:

- **Proxy:** Controls access to an object and can intercept method calls.
- **Decorator:** Dynamically adds responsibilities or behavior to an object.

## 2. How Spring AOP Works

Consider a service that transfers money. We want to log every method call without adding logging code directly to the service.

### Target service

```java
@Service
public class PaymentService {

    public void transfer() {
        System.out.println("Transferring money...");
    }
}
```

The `PaymentService` contains only its business logic.

### Define an aspect

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void logBeforeMethod() {
        System.out.println("Method execution started");
    }
}
```

The `@Before` advice executes before methods matching the pointcut expression.

### What happens during execution?

When another Spring bean calls `paymentService.transfer()`, the flow is conceptually:

```text
Client
  |
  v
Spring AOP Proxy
  |
  +--> LoggingAspect: logBeforeMethod()
  |
  v
Target: PaymentService.transfer()
  |
  v
Return to Client
```

Example output:

```text
Method execution started
Transferring money...
```

The client calls the proxy, and the proxy intercepts the invocation to apply the aspect before delegating to the target object.

## 3. Simplified Logic Behind the Proxy

The following pseudocode illustrates the idea. It is a conceptual example, not Spring's actual implementation.

```java
class PaymentServiceProxy {

    private final PaymentService target;

    PaymentServiceProxy(PaymentService target) {
        this.target = target;
    }

    public void transfer() {
        System.out.println("Method execution started");

        target.transfer();
    }
}
```

Instead of calling the target directly:

```java
paymentService.transfer();
```

the client invokes the proxy, which performs additional work and then calls the target.

Spring manages proxy creation and advice invocation automatically when AOP is configured appropriately.

## 4. Proxy vs Decorator Pattern

| Feature | Proxy Pattern | Decorator Pattern |
|---|---|---|
| Primary purpose | Control or intercept access to an object | Add responsibilities to an object |
| Typical use | Security, lazy loading, transactions, AOP | Adding features such as compression or extra formatting |
| Object relationship | Proxy delegates to a target | Decorator wraps a component |
| Behavior | May intercept calls, check permissions, or manage lifecycle | Extends behavior while preserving the component interface |
| Spring example | AOP proxies and `@Transactional` | No single universal Spring feature is defined by this pattern |

Both patterns can wrap an object and delegate method calls, so their structures can look similar. The key distinction is their intent.

## 5. Other Concepts Involved in Spring AOP

Spring AOP also uses several related concepts:

- **Aspect:** Groups cross-cutting logic, such as logging or auditing.
- **Advice:** Defines when that logic runs, such as `@Before`, `@After`, or `@Around`.
- **Pointcut:** Specifies which method executions should be intercepted.
- **Target object:** The original Spring bean containing business logic.
- **Proxy:** Intercepts calls and coordinates the advice and target invocation.
- **Weaving:** The process of associating aspects with application behavior. Spring AOP typically does this at runtime through proxies.

### Example with `@Transactional`

```java
@Service
public class EmployeeService {

    @Transactional
    public void createEmployee() {
        // Database operations
    }
}
```

When the method is called through the Spring proxy, transaction advice can start a transaction, invoke the target method, and commit or roll back according to the configured transaction rules.

The service does not need to implement transaction-management logic itself.

## 6. Important Interview Follow-up: Proxy Limitations

Spring's standard proxy-based AOP has some limitations:

1. **Self-invocation:** Calling another advised method using `this.someMethod()` generally bypasses the proxy.
2. **Direct object creation:** Objects created using `new` outside Spring's bean management do not automatically receive Spring AOP proxies.
3. **Method visibility and proxy type:** Private methods cannot be intercepted through standard proxy-based method interception. CGLIB also cannot override final methods, and final classes cannot be subclassed.
4. **Calls must pass through the proxy:** Advice normally runs when an invocation enters through the Spring-managed proxy.

These limitations also matter for annotations such as `@Transactional`, which commonly rely on Spring's proxy infrastructure.

## 7. Interview-Ready Conclusion

Spring AOP primarily uses the **Proxy Design Pattern**. Spring creates a proxy around a target bean and intercepts method calls to execute cross-cutting logic such as logging, security, and transaction management. It commonly uses JDK dynamic proxies or CGLIB-based proxies. Although the proxy may look structurally similar to a decorator, the main intent in Spring AOP is interception and control of method invocation rather than simply adding new responsibilities.

## Quick Revision

- **Primary pattern:** Proxy
- **Purpose:** Apply cross-cutting concerns without changing business code
- **JDK proxy:** Typically used when proxying interfaces
- **CGLIB proxy:** Uses subclassing for class-based proxying
- **Related concepts:** Aspect, advice, pointcut, target, weaving
- **Common examples:** `@Transactional`, logging, method security
- **Decorator distinction:** Adds responsibilities; Proxy primarily controls or intercepts access


