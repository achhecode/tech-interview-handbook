---
title: Singleton
parent: Design Patterns and LLD
nav_order: 2
---

# Singleton
{: .no_toc }

A creational design pattern that ensures a class has only one instance and provides a global access point to it.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

1. TOC
{:toc}
</details>

---

## 1. What is the Singleton Design Pattern?

{: .label .label-green }

**Answer**

The Singleton is a creational design pattern that ensures a class has only one instance within a defined scope and provides a single access point to that instance.

It is useful when multiple parts of an application must share the same object, such as a configuration provider or an application-wide registry.

### Key characteristics

- **Private constructor:** Prevents other classes from directly creating instances.
- **Single instance:** The class controls how its instance is created.
- **Global access point:** A static method provides access to the instance.
- **Controlled initialization:** The instance can be created eagerly or lazily.

### Example: Basic Singleton

```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {
        // Prevent direct instantiation
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

**Usage**

```java
Singleton first = Singleton.getInstance();
Singleton second = Singleton.getInstance();

System.out.println(first == second); // true
```

Both variables refer to the same object.

**Important:** This basic implementation is not thread-safe. Two threads could enter `getInstance()` simultaneously and create separate objects.

---

## 2. How can the Singleton Pattern be broken?

{: .label .label-yellow }

**Answer**

A Singleton implementation can be bypassed or produce multiple instances when it does not account for reflection, serialization, cloning, concurrency, or multiple class loaders.

### 2.1 Reflection

Java reflection can access a private constructor and invoke it to create another instance.

```java
import java.lang.reflect.Constructor;

public class ReflectionExample {

    public static void main(String[] args) throws Exception {
        Singleton first = Singleton.getInstance();

        Constructor<Singleton> constructor =
                Singleton.class.getDeclaredConstructor();

        constructor.setAccessible(true);

        Singleton second = constructor.newInstance();

        System.out.println(first == second); // false
    }
}
```

**Why does it happen?**

The private constructor prevents normal Java code from instantiating the class, but reflection can bypass that restriction when access checks can be suppressed.

Modern Java module and security restrictions may prevent certain reflective operations.

### 2.2 Multithreading

The basic implementation is not thread-safe.

```java
public static Singleton getInstance() {
    if (instance == null) {
        instance = new Singleton();
    }
    return instance;
}
```

Two threads can both observe `instance == null` and create separate instances.

### 2.3 Serialization and deserialization

If a Singleton implements `Serializable`, deserialization can create a new object instead of returning the existing instance unless the class handles deserialization appropriately.

### 2.4 Cloning

If a Singleton implements `Cloneable`, calling `clone()` may create another instance unless cloning is prevented or explicitly handled.

### 2.5 Dependency injection and factories

Dependency injection and factory methods do not inherently break the Singleton pattern. They can create multiple instances if configured to do so, or provide different implementations for testing.

**Note:** A Singleton is a design constraint, not an automatic guarantee against every mechanism that can create objects.

---

## 3. How do you prevent a Singleton from being broken?

{: .label .label-green }

**Answer**

Choose an implementation based on the guarantees your application needs. For most Java applications, the initialization-on-demand holder idiom or an enum Singleton is a good choice.

### 3.1 Use a private constructor

A private constructor prevents ordinary external instantiation.

```java
private Singleton() {
}
```

However, a private constructor alone does not protect against reflection or other object-creation mechanisms.

### 3.2 Use a final instance reference

Declare the instance reference as `final` when the instance is created during class initialization.

```java
private static final Singleton INSTANCE = new Singleton();
```

The reference cannot be reassigned, although the object's internal state may still be mutable.

### 3.3 Use a thread-safe implementation

The following implementations safely publish a single instance under normal class-loader usage.

#### Option A: Eager initialization

```java
public class Singleton {

    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

**How it works**

- The JVM initializes the static field when the class is initialized.
- Class initialization is thread-safe.
- The instance is created even if `getInstance()` is never called.

Use eager initialization when creating the object is inexpensive and always necessary.

#### Option B: Initialization-on-demand holder idiom

```java
public class Singleton {

    private Singleton() {
    }

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

**How it works**

- The outer class can be loaded without immediately creating the Singleton instance.
- The nested `Holder` class is initialized only when `getInstance()` accesses it.
- JVM class initialization guarantees thread-safe initialization.
- No explicit synchronization is needed.

This is a good default when you need lazy initialization without synchronization overhead on every access.

#### Option C: Enum Singleton

```java
public enum AppConfig {
    INSTANCE;

    public void printMessage() {
        System.out.println("Application configuration");
    }
}
```

**Usage**

```java
AppConfig.INSTANCE.printMessage();
```

**Why is it robust?**

- The JVM handles instance creation and initialization safely.
- Java serialization preserves the enum constant's identity.
- Reflection cannot create additional enum constants through ordinary reflective constructor invocation.
- There is no need to write a custom `readResolve()` method.

An enum Singleton is generally the strongest straightforward choice when the Singleton can naturally be represented as an enum constant.

---

## 4. How do you protect a Singleton from reflection, serialization, and cloning?

{: .label .label-yellow }

**Answer**

Different techniques protect against different mechanisms. No single conventional class-based implementation automatically protects against every possible attack or runtime boundary.

### Reflection protection

A class-based Singleton can reject a second constructor invocation.

```java
public class Singleton {

    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {
        if (INSTANCE != null) {
            throw new IllegalStateException(
                    "Singleton instance already exists"
            );
        }
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

**Caution:** This example has a Java initialization-order problem: `INSTANCE` is still `null` while its constructor is running. Therefore, this guard does not work as intended.

A safer class-based guard uses a separate static flag, but its correctness requires careful consideration of class initialization and constructor execution. Prefer an enum Singleton when reflection resistance is important.

### Serialization protection

For a serializable class-based Singleton, use `readResolve()` to return the existing instance after deserialization.

```java
import java.io.Serializable;

public class SerializableSingleton implements Serializable {

    private static final long serialVersionUID = 1L;

    private static final SerializableSingleton INSTANCE =
            new SerializableSingleton();

    private SerializableSingleton() {
    }

    public static SerializableSingleton getInstance() {
        return INSTANCE;
    }

    private Object readResolve() {
        return INSTANCE;
    }
}
```

### Cloning protection

Do not implement `Cloneable` for a Singleton unless there is a specific reason. If the class inherits a public or protected cloning path, prevent cloning explicitly.

```java
@Override
protected Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException(
            "Singleton instances cannot be cloned"
    );
}
```

**Recommendation:** Prefer an enum Singleton if it fits the use case and you want strong built-in protection against reflection and serialization-based duplication.

---

## 5. What is the difference between a Java Singleton and a Spring Singleton?

{: .label .label-blue }

**Answer**

| Java Singleton | Spring Singleton |
|---|---|
| Implemented by the developer or language features. | Managed by the Spring container. |
| Usually provides a static access method or enum constant. | Retrieved through dependency injection or the application context. |
| Intended to control instance creation within a defined scope. | One bean instance per bean definition per Spring container. |
| Thread safety depends on the implementation and object's state. | Bean creation is managed by Spring, but mutable shared state still needs thread safety. |

### Example: Spring Singleton bean

```java
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {

    public String getMessage() {
        return "Employee service";
    }
}
```

Spring uses singleton scope by default for this bean.

```java
@Service
public class ReportService {

    private final EmployeeService employeeService;

    public ReportService(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }
}
```

Spring injects the managed `EmployeeService` instance into `ReportService`.

**Important:** Spring singleton scope does not mean one instance per JVM. Separate Spring containers can have separate instances, and the bean is not automatically thread-safe.

---

## 6. What are the advantages and disadvantages of the Singleton Pattern?

{: .label .label-blue }

**Answer**

### Advantages

- **Controlled instantiation:** Restricts instance creation within the intended scope.
- **Shared access:** Multiple components can use the same instance.
- **Lazy initialization:** The object can be created only when needed.
- **Reduced duplication:** Avoids repeatedly creating certain expensive objects.

### Disadvantages

- **Global state:** Hidden dependencies can make code harder to understand.
- **Testing difficulty:** Tests may share state between executions.
- **Concurrency risks:** Mutable state requires proper synchronization.
- **Lifecycle complexity:** Resetting or replacing an instance can be difficult.
- **Scope limitations:** A Java Singleton does not automatically guarantee one instance across class loaders, processes, or distributed services.

**Best practice:** Prefer dependency injection over directly accessing global Singleton state when practical. It improves testability and makes dependencies explicit.

---

## 7. When should you use the Singleton Pattern?

{: .label .label-blue }

**Answer**

Use a Singleton when a single shared instance is a genuine requirement, not simply to avoid creating objects.

### Suitable use cases

- Immutable application configuration.
- A shared registry within one application context.
- A stateless service with a controlled lifecycle.
- A coordinator that must have one instance within a specific scope.

### Avoid using it when

- Multiple independent configurations are required.
- Tests need isolated instances.
- The application runs across multiple processes or distributed nodes.
- Shared mutable state creates unnecessary concurrency problems.
- Dependency injection can manage the object's lifecycle more cleanly.

**Remember:** A Singleton in one JVM is not a distributed Singleton. In a multi-instance application, use appropriate distributed coordination if the requirement spans multiple processes or machines.

---

## 8. Quick interview revision

{: .label .label-green }

| Question | Short answer |
|---|---|
| What is Singleton? | A creational pattern that controls instance creation and provides shared access. |
| Why use a private constructor? | To prevent ordinary external instantiation. |
| Is the basic lazy Singleton thread-safe? | No. |
| Best lazy class-based implementation? | Initialization-on-demand holder idiom. |
| Most robust built-in Java option? | Enum Singleton, when suitable. |
| How does reflection break a class-based Singleton? | It may invoke the private constructor reflectively. |
| How does serialization break it? | Deserialization may create a new object without `readResolve()`. |
| How do you prevent cloning? | Avoid `Cloneable` or override `clone()` to reject cloning. |
| Is a Spring Singleton one instance per JVM? | No, it is one instance per bean definition per container. |
| Is a Singleton automatically thread-safe? | Instance creation may be safe, but mutable state still needs protection. |

---

## 9. Final recommendation

For interviews and production Java code, remember these three implementations:

1. **Eager initialization:** Simple and thread-safe when initialization is inexpensive.
2. **Holder idiom:** Lazy, thread-safe, and a strong general-purpose class-based choice.
3. **Enum Singleton:** The preferred choice when you need built-in protection against ordinary reflection and serialization duplication.

Use the pattern only when a single instance is a real requirement, and prefer dependency injection when it provides a cleaner design.