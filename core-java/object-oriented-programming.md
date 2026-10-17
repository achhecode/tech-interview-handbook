---
title: Object-Oriented Programming
parent: Core Java
nav_order: 1
---

---
title: Object-Oriented Programming
parent: Core Java
nav_order: 1
---

# Object-Oriented Programming
{: .no_toc }

The four pillars of OOP, with Java examples and common interview questions.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

  1. TOC
  {:toc}
</details>

---

## What are the four pillars of OOP? Explain with examples.

Easy
{: .label .label-green }

Object-Oriented Programming (OOP) is a programming approach that organizes code around objects, which combine data and behavior. Java uses four fundamental OOP principles to make applications easier to maintain, reuse, and extend.

- **Encapsulation:** Bundles data and related methods within a class while controlling access to its internal state. For example, a `BankAccount` class can keep its `balance` private and provide a `deposit()` method to update it safely.

- **Abstraction:** Hides implementation details and exposes only the essential operations an object provides. In Java, interfaces and abstract classes help achieve abstraction. For example, a `Payment` interface defines a `pay()` method without specifying how each payment method processes a transaction.

- **Inheritance:** Allows a class to acquire and extend the accessible behavior of another class using `extends`. For example, `Car` can extend `Vehicle` and reuse common vehicle functionality. Inheritance is appropriate when there is a genuine *is-a* relationship.

- **Polymorphism:** Allows the same interface or method call to produce different behavior depending on the context or actual object type. Java supports compile-time polymorphism through method overloading and runtime polymorphism through method overriding.

Java example: Abstraction and runtime polymorphism

```java
interface Payment {
    void pay(double amount);
}

class UpiPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Payment via UPI: " + amount);
    }
}

class CardPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Payment via Card: " + amount);
    }
}

public class Main {
    public static void main(String[] args) {
        Payment payment = new UpiPayment();
        payment.pay(100);

        payment = new CardPayment();
        payment.pay(200);
    }
}
```

**Output:**
```text
Payment via UPI: 100.0
Payment via Card: 200.0
```

**Explanation:**

- `Payment` provides an abstraction by defining the `pay()` contract.
- `UpiPayment` and `CardPayment` implement that contract differently.
- The same `payment.pay()` call executes the implementation corresponding to the actual object at runtime. This demonstrates runtime polymorphism.

**Follow-up interview questions**

- What is the difference between method overloading and method overriding?
- Can static or private methods be overridden in Java?
- What is the difference between an abstract class and an interface?
- What is the difference between encapsulation and abstraction?
- Does Java support multiple inheritance?
- Why is composition sometimes preferred over inheritance?

---

## What is the difference between an Abstract Class and an Interface in Java?

Medium
{: .label .label-yellow }

An **abstract class** is used when related classes need to share common state and implementation, whereas an **interface** defines a contract that different classes can implement, even if they are not related through inheritance.

Key differences

| Feature | Abstract Class | Interface |
|---|---|---|
| **Purpose** | Provides a common base with shared state and behavior. | Defines a contract that implementing classes agree to follow. |
| **Methods** | Can contain abstract and concrete methods. | Can contain abstract, default, static, and private methods (private methods since Java 9). |
| **Variables** | Can have instance and static fields, including mutable or final fields. | Fields are implicitly `public static final` constants. |
| **Inheritance** | A class can extend only one class. | A class can implement multiple interfaces. |
| **Constructors** | Can have constructors to initialize inherited state. | Cannot have constructors. |
| **Access modifiers** | Members can use different access modifiers. | Abstract methods are implicitly `public`; other method types have their own access rules. |
| **Instantiation** | Cannot be instantiated directly. | Cannot be instantiated directly. |
| **Main method** | Can declare a `main()` method. | Can also declare a `main()` method. |

Example: Abstract class

```java
abstract class Vehicle {
    protected String brand;

    Vehicle(String brand) {
        this.brand = brand;
    }

    abstract void start();

    void displayBrand() {
        System.out.println(brand);
    }
}

class Car extends Vehicle {
    Car(String brand) {
        super(brand);
    }

    @Override
    void start() {
        System.out.println("Car starts with a key");
    }
}
```

Here, `Vehicle` provides shared state (`brand`) and common behavior (`displayBrand()`), while each concrete subclass must implement `start()`.

Example: Interface

```java
interface Payment {
    void pay(double amount);
}

class UpiPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid via UPI: " + amount);
    }
}

class CardPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid via Card: " + amount);
    }
}
```

Here, `Payment` defines a common contract. Different payment implementations can fulfill it in their own ways without needing to share a parent class.

When should you use each?

**Use an abstract class when:**
- Related subclasses share common state or implementation.
- You need constructors to initialize common fields.
- You want to provide protected or package-private helper methods to subclasses.

**Use an interface when:**
- You want to define a capability or contract for different classes.
- Multiple unrelated classes should support the same behavior.
- You want to reduce coupling and make implementations easier to replace or mock in tests.

**Interview tip**

An abstract class and an interface can both contain implemented methods, so the distinction is not simply *partial abstraction versus complete abstraction*.

Remember this rule of thumb:

- **Abstract class:** What common foundation do related classes share?
- **Interface:** What capability or contract must an implementation provide?

---


## What is the use of a constructor in an abstract class?

Easy
{: .label .label-green }

Although an abstract class cannot be instantiated directly, it can have a constructor. The constructor is executed when an object of a concrete subclass is created, allowing the abstract class to initialize inherited fields and perform common initialization tasks.

Why do we need a constructor in an abstract class?

1. **Initialize common fields:** Set up properties shared by all subclasses, such as an employee's name or a bank account's account number.

2. **Enforce validation:** Validate input values during initialization to ensure that objects start in a valid state.

3. **Reuse initialization logic:** Avoid duplicating the same initialization code in every subclass.

4. **Initialize the parent class:** Ensure that inherited state is initialized before the subclass performs its own initialization.

Java example

```java
abstract class BankAccount {
    protected String accountNumber;
    protected double balance;

    BankAccount(String accountNumber, double balance) {
        if (balance < 0) {
            throw new IllegalArgumentException(
                "Initial balance cannot be negative"
            );
        }

        this.accountNumber = accountNumber;
        this.balance = balance;

        System.out.println("BankAccount constructor called");
    }
}

class SavingsAccount extends BankAccount {

    SavingsAccount(String accountNumber, double balance) {
        super(accountNumber, balance);
        System.out.println("SavingsAccount constructor called");
    }
}

public class Main {
    public static void main(String[] args) {
        SavingsAccount account =
            new SavingsAccount("ACC101", 5000);

        System.out.println(account.accountNumber);
        System.out.println(account.balance);
    }
}
```

**Output:**

```text
BankAccount constructor called
SavingsAccount constructor called
ACC101
5000.0
```

**How does it work?**

1. `new SavingsAccount(...)` initiates object creation.
2. `super(accountNumber, balance)` invokes the abstract class constructor.
3. The parent constructor validates and initializes the inherited fields.
4. The subclass constructor completes its initialization.

**Important:** An abstract class constructor cannot be called to create an abstract class object directly. It is invoked as part of constructing a concrete subclass object. Also, a private constructor in an abstract class prevents ordinary subclass construction outside that class, so it is not a typical way to control abstract class instantiation.

Interview takeaway

**An abstract class constructor initializes common state and performs shared validation for its subclasses.** Even though the abstract class cannot be instantiated directly, its constructor participates in the creation of every subclass object that invokes it.

---

## What is Abstraction, and What Are Its Advantages? Why Use It Instead of a Concrete Class?

Medium
{: .label .label-yellow }

**Abstraction is an object-oriented programming principle that exposes essential functionality while hiding implementation details.** It allows developers to interact with an object through a well-defined contract without needing to know how its functionality is implemented internally.

In Java, abstraction is commonly achieved using **interfaces and abstract classes**.

For example, when using an ATM, you select an operation such as withdrawing money without needing to understand the internal banking systems, database queries, or transaction processing.

**Java example: Concrete class vs. abstraction**

**1. Without abstraction**

Suppose an application directly depends on a concrete payment implementation.

```java
class UpiPayment {
    public void pay(double amount) {
        System.out.println("Processing UPI payment: " + amount);
    }
}

class OrderService {
    private UpiPayment payment = new UpiPayment();

    public void checkout(double amount) {
        payment.pay(amount);
    }
}
```

Here, `OrderService` is tightly coupled to `UpiPayment`. If we want to introduce card payments, we may need to modify `OrderService` and its dependencies.

**2. With abstraction**

Define a common contract and let different concrete classes implement it.

```java
interface Payment {
    void pay(double amount);
}

class UpiPayment implements Payment {
    public void pay(double amount) {
        System.out.println("Processing UPI payment: " + amount);
    }
}

class CardPayment implements Payment {
    public void pay(double amount) {
        System.out.println("Processing card payment: " + amount);
    }
}

class OrderService {
    private final Payment payment;

    OrderService(Payment payment) {
        this.payment = payment;
    }

    public void checkout(double amount) {
        payment.pay(amount);
    }
}
```

Usage:

```java
Payment payment = new UpiPayment();
OrderService order = new OrderService(payment);
order.checkout(100);

Payment cardPayment = new CardPayment();
OrderService cardOrder = new OrderService(cardPayment);
cardOrder.checkout(200);
```

**What is the advantage here?**

- `OrderService` depends on the `Payment` interface rather than a specific implementation.
- We can introduce another payment method without changing the checkout logic.
- We can inject a mock payment implementation during unit testing.
- Each payment implementation can change its internal logic without affecting `OrderService`, as long as it honors the interface contract.

Notice that the concrete classes still perform the actual work. **Abstraction does not replace concrete classes; it separates the contract from its implementation.**

**Advantages of abstraction**

1. **Reduced complexity:** Users of a class interact with essential operations instead of implementation details.
2. **Loose coupling:** Components depend on contracts rather than specific implementations.
3. **Flexibility:** Implementations can be replaced without rewriting the code that uses them.
4. **Maintainability:** Internal implementation changes are less likely to affect other components.
5. **Testability:** Mock or fake implementations can be supplied for testing.
6. **Extensibility:** New implementations can be added with minimal changes to existing business logic.

**Why not just use concrete classes?**

Concrete classes are perfectly suitable when the implementation is simple and unlikely to vary. Abstraction becomes valuable when different implementations need to be interchangeable, when multiple components should share a contract, or when the application needs to be easier to test and extend.

For example, in Spring Boot, a service can depend on a `PaymentGateway` interface while Spring injects a suitable implementation through dependency injection.

**Interview takeaway**

**Abstraction is not merely about hiding code. Its major design benefit is separating what a component does from how it does it.** This separation makes it easier to change implementations, reduce coupling, and extend applications without unnecessarily modifying existing business logic.


---

## What are Aggregation, Composition, and Inheritance in Java?

Medium
{: .label .label-yellow }

Aggregation, composition, and inheritance are object-oriented programming concepts used to establish relationships between classes.

- **Inheritance** represents an *is-a* relationship, where a subclass inherits accessible behavior from a parent class.
- **Composition** represents a strong *has-a* relationship, where an object owns and manages the lifecycle of its component objects.
- **Aggregation** represents a weaker *has-a* relationship, where an object references other objects that can exist independently.

1. Inheritance — Is-a relationship

Inheritance allows a class to reuse or specialize the behavior of another class using the `extends` keyword.

```java
class Vehicle {
    void start() {
        System.out.println("Vehicle started");
    }
}

class Car extends Vehicle {
    void drive() {
        System.out.println("Car is driving");
    }
}
```

Usage:

```java
Car car = new Car();
car.start();
car.drive();
```

**Explanation:** `Car` extends `Vehicle`, so it inherits the accessible `start()` method.

**How to identify it:** Look for `extends` in a class declaration.

2. Composition — Strong has-a relationship

Composition allows a class to contain and own another object as one of its components. The component's lifecycle is typically managed by the owning object.

```java
class Engine {
    void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private final Engine engine;

    Car() {
        this.engine = new Engine();
    }

    void start() {
        engine.start();
    }
}
```

Usage:

```java
Car car = new Car();
car.start();
```

**Explanation:** `Car` creates and owns its `Engine` component in this example. The car delegates engine startup to that component.

**How to identify it:** Look for a member object whose ownership and lifecycle are tied to the containing object. Creating the object inside the constructor is one possible indication, but ownership is the important distinction.

3. Aggregation — Weak has-a relationship

Aggregation allows a class to reference objects that are created and can exist independently of the containing object.

```java
import java.util.List;

class Student {
    private final String name;

    Student(String name) {
        this.name = name;
    }

    String getName() {
        return name;
    }
}

class University {
    private final List<Student> students;

    University(List<Student> students) {
        this.students = students;
    }

    void displayStudents() {
        for (Student student : students) {
            System.out.println(student.getName());
        }
    }
}
```

Usage:

```java
Student s1 = new Student("Aman");
Student s2 = new Student("Sara");

University university = new University(List.of(s1, s2));
university.displayStudents();
```

**Explanation:** The students are created independently and passed to the university. They can continue to exist even if the `University` object is no longer referenced.

**How to identify it:** Look for references to independently managed objects, often supplied through a constructor or setter.

Comparison table

| Feature | Inheritance | Composition | Aggregation |
|---|---|---|---|
| Relationship | Is-a | Strong has-a | Weak has-a |
| Java syntax | `extends` | Object ownership and references | Object references |
| Purpose | Reuse or specialize behavior | Build an object from owned components | Associate independently existing objects |
| Lifecycle | Parent-class


---

## What are Aggregation, Composition, and Inheritance in Java?

Medium
{: .label .label-yellow }

Aggregation, composition, and inheritance are object-oriented programming concepts used to establish relationships between classes.

- **Inheritance** represents an *is-a* relationship, where a subclass inherits accessible behavior from a parent class.
- **Composition** represents a strong *has-a* relationship, where an object owns and manages the lifecycle of its component objects.
- **Aggregation** represents a weaker *has-a* relationship, where an object references other objects that can exist independently.

1. Inheritance — Is-a relationship

Inheritance allows a class to reuse or specialize the behavior of another class using the `extends` keyword.

```java
class Vehicle {
    void start() {
        System.out.println("Vehicle started");
    }
}

class Car extends Vehicle {
    void drive() {
        System.out.println("Car is driving");
    }
}
```

Usage:

```java
Car car = new Car();
car.start();
car.drive();
```

**Explanation:** `Car` extends `Vehicle`, so it inherits the accessible `start()` method.

**How to identify it:** Look for `extends` in a class declaration.

2. Composition — Strong has-a relationship

Composition allows a class to contain and own another object as one of its components. The component's lifecycle is typically managed by the owning object.

```java
class Engine {
    void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private final Engine engine;

    Car() {
        this.engine = new Engine();
    }

    void start() {
        engine.start();
    }
}
```

Usage:

```java
Car car = new Car();
car.start();
```

**Explanation:** `Car` creates and owns its `Engine` component in this example. The car delegates engine startup to that component.

**How to identify it:** Look for a member object whose ownership and lifecycle are tied to the containing object. Creating the object inside the constructor is one possible indication, but ownership is the important distinction.

3. Aggregation — Weak has-a relationship

Aggregation allows a class to reference objects that are created and can exist independently of the containing object.

```java
import java.util.List;

class Student {
    private final String name;

    Student(String name) {
        this.name = name;
    }

    String getName() {
        return name;
    }
}

class University {
    private final List<Student> students;

    University(List<Student> students) {
        this.students = students;
    }

    void displayStudents() {
        for (Student student : students) {
            System.out.println(student.getName());
        }
    }
}
```

Usage:

```java
Student s1 = new Student("Aman");
Student s2 = new Student("Sara");

University university = new University(List.of(s1, s2));
university.displayStudents();
```

**Explanation:** The students are created independently and passed to the university. They can continue to exist even if the `University` object is no longer referenced.

**How to identify it:** Look for references to independently managed objects, often supplied through a constructor or setter.

Comparison table

| Feature | Inheritance | Composition | Aggregation |
|---|---|---|---|
| Relationship | Is-a | Strong has-a | Weak has-a |
| Java syntax | `extends` | Object ownership and references | Object references |
| Purpose | Reuse or specialize behavior | Build an object from owned components | Associate independently existing objects |
| Lifecycle | Parent-class state is part of the subclass object | Part lifecycle is tied to the whole by design | Part can exist independently |
| Example | `Car extends Vehicle` | `Car` owns an `Engine` | `University` references `Student` objects |

How to identify these relationships in an existing codebase

- Search for `extends` to find class inheritance.
- Inspect instance fields to find objects that a class contains or references.
- Inspect constructors and setters to determine whether those objects are created internally or supplied externally.
- Examine ownership and lifecycle requirements to distinguish composition from aggregation.

**Important:** The `implements` keyword indicates that a class implements an interface; it is not class inheritance through `extends`. Also, the presence of `new` inside a constructor does not automatically prove composition, and passing an object through a constructor does not automatically prove aggregation. The intended ownership and lifecycle determine the relationship.

Interview takeaway

Remember the three relationships:

- **Inheritance:** Car *is a* Vehicle.
- **Composition:** Car *owns an* Engine.
- **Aggregation:** University *has* Students who can exist independently.


---

## What is Multilevel Inheritance in Java?

Easy
{: .label .label-green }

**Multilevel inheritance** is a type of inheritance in which a class inherits from another derived class, forming a chain of inheritance.

In Java, a subclass inherits accessible fields and methods from its parent class and, indirectly, from its ancestors. This allows classes to reuse existing functionality and extend it with their own behavior.

For example:

`Animal → Mammal → Dog`

- `Animal` is the base class.
- `Mammal` extends `Animal`.
- `Dog` extends `Mammal`.

Java example

```java
class Animal {
    void eat() {
        System.out.println("Animal eats food");
    }
}

class Mammal extends Animal {
    void walk() {
        System.out.println("Mammal walks");
    }
}

class Dog extends Mammal {
    void bark() {
        System.out.println("Dog barks");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog();

        dog.eat();   // Inherited from Animal
        dog.walk();  // Inherited from Mammal
        dog.bark();  // Defined in Dog
    }
}
```

Output

```text
Animal eats food
Mammal walks
Dog barks
```

How does it work?

1. `Mammal` extends `Animal` and inherits its accessible `eat()` method.
2. `Dog` extends `Mammal` and inherits `walk()` as well as the accessible `eat()` method inherited through `Mammal`.
3. A `Dog` object can therefore access methods defined at all three levels, subject to Java's access rules.

Advantages

- **Code reusability:** Common functionality can be defined once in a base class.
- **Extensibility:** Each subclass can introduce specialized behavior.
- **Logical hierarchy:** Related classes can be organized into a clear inheritance chain.

Important interview points

- Java supports multilevel inheritance through classes.
- Java does not support multiple inheritance of classes, meaning a class cannot extend two classes directly.
- Constructors are not inherited, although superclass constructors execute during subclass object creation.
- Private members are not directly accessible in subclasses.

**Interview takeaway:** Multilevel inheritance forms a chain in which a subclass can reuse accessible functionality from both its immediate parent and its ancestors.

---

## When do you use Encapsulation and Abstraction in your project?

Medium
{: .label .label-yellow }

In my Java and Spring Boot projects, I use **encapsulation to protect an object's internal state and maintain data integrity**, while I use **abstraction to hide implementation details and reduce dependencies between components**.

1. Encapsulation — Protecting data

I use encapsulation when designing entity classes, DTOs, and domain models that contain data requiring controlled access or validation.

**Example:** In a banking application, an `Account` class should not allow other classes to modify its balance arbitrarily. Instead, it exposes methods that enforce business rules.

```java
public class Account {
    private double balance;

    public Account(double balance) {
        if (balance < 0) {
            throw new IllegalArgumentException(
                "Balance cannot be negative"
            );
        }
        this.balance = balance;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException(
                "Deposit must be positive"
            );
        }
        balance += amount;
    }
}
```

**Why use it?**
- Prevents uncontrolled modification of internal data.
- Enforces business rules and validation.
- Keeps an object's state consistent.
- Makes changes to internal implementation safer.

Notice that I don't provide a general-purpose `setBalance()` method because every balance change should follow the appropriate business rules.

2. Abstraction — Hiding implementation details

I use abstraction when different implementations should expose a common contract. This is especially useful in service layers, integrations, and microservice applications.

**Example:** Suppose an application supports multiple payment providers. The order service should not need to know the internal details of each provider.

```java
public interface PaymentGateway {
    void processPayment(double amount);
}
```

```java
public class UpiPaymentGateway implements PaymentGateway {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing UPI payment: " + amount);
    }
}
```

```java
public class CardPaymentGateway implements PaymentGateway {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing card payment: " + amount);
    }
}
```

```java
public class OrderService {
    private final PaymentGateway paymentGateway;

    public OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    public void placeOrder(double amount) {
        paymentGateway.processPayment(amount);
    }
}
```

**Why use it?**
- Allows implementations to be replaced without changing the business logic.
- Reduces coupling between services and infrastructure.
- Makes unit testing easier by allowing mock implementations.
- Makes it easier to introduce additional payment providers.

In Spring Boot, dependency injection can supply the appropriate `PaymentGateway` implementation to `OrderService`.

3. How do you use both together?

In a real application, these principles complement each other:

- **Encapsulation:** The `Account` class protects its balance and validates transactions.
- **Abstraction:** A `PaymentGateway` interface defines the contract for processing payments.
- **Combined benefit:** Business logic interacts with clearly defined operations while internal state and implementation details remain controlled.

Interview takeaway

In my projects, I use encapsulation mainly at the **class and domain-model level** to protect state and enforce validation. I use abstraction mainly at the **service and architecture level** to define contracts and make implementations interchangeable.

Both help improve maintainability, testability, and reliability, but they solve different design problems.

---

## What is Polymorphism, How Is It Achieved, and How Is Encapsulation Implemented in Java?

Medium
{: .label .label-yellow }

**Polymorphism** and **encapsulation** are two fundamental principles of object-oriented programming in Java.

- **Polymorphism** allows the same method name or interface to represent different behaviors depending on the arguments or the actual object type.
- **Encapsulation** combines data and related behavior within a class while restricting direct access to internal state and providing controlled ways to interact with it.

1 Polymorphism in Java

Java supports two commonly discussed forms of polymorphism: **compile-time polymorphism (method overloading)** and **runtime polymorphism (method overriding)**.

### A. Method overloading — Compile-time polymorphism

Method overloading occurs when multiple methods in the same class have the same name but different parameter lists. The compiler selects the appropriate method based on the arguments and the available method signatures.

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

Usage:

```java
Calculator calculator = new Calculator();

System.out.println(calculator.add(10, 20));
System.out.println(calculator.add(10.5, 20.5));
System.out.println(calculator.add(10, 20, 30));
```

Output:

```text
30
31.0
60
```

**Key point:** Changing only the return type is not sufficient to overload a method. The parameter list must differ.

### B. Method overriding — Runtime polymorphism

Method overriding occurs when a subclass provides its own implementation of an inherited instance method, following Java's overriding rules.

```java
class Animal {
    void speak() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {
    @Override
    void speak() {
        System.out.println("Dog barks");
    }
}

class Cat extends Animal {
    @Override
    void speak() {
        System.out.println("Cat meows");
    }
}
```

Usage:

```java
Animal animal = new Dog();
animal.speak();

animal = new Cat();
animal.speak();
```

Output:

```text
Dog barks
Cat meows
```

**Explanation:** Although the reference type is `Animal`, Java invokes the overridden method corresponding to the actual object at runtime. This is runtime polymorphism.

Advantages of polymorphism

- Reduces the need for repetitive type-specific code.
- Makes applications easier to extend with new implementations.
- Encourages programming to interfaces and reusable designs.
- Simplifies testing and maintenance when combined with abstraction.

2 How is Encapsulation achieved in Java?

Encapsulation is commonly implemented using **access modifiers, private fields, and public methods that control access to an object's state**. Getters and setters are useful when appropriate, but they are not mandatory for every field.

Access modifiers in Java

| Modifier | Accessibility |
|---|---|
| `public` | Accessible wherever the declaring type and its members are accessible. |
| `private` | Accessible only within the declaring top-level class, including its applicable nested-class context. |
| `protected` | Accessible within the same package and, subject to Java's subclass access rules, from subclasses in other packages. |
| Default (no modifier) | Accessible within the same package. |

Java example

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        setName(name);
        setAge(age);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException(
                "Name cannot be empty"
            );
        }
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException(
                "Age cannot be negative"
            );
        }
        this.age = age;
    }
}
```

Usage:

```java
Person person = new Person("Aman", 25);

System.out.println(person.getName());
System.out.println(person.getAge());

person.setAge(26);

// person.age = -5; // Compilation error: age is private
```

Output:

```text
Aman
25
```

**Explanation:**

- The `name` and `age` fields are private, preventing direct access from unrelated classes.
- Getter methods provide controlled read access.
- Setter methods validate new values before updating the object's state.
- Invalid data is rejected, helping maintain object consistency.

Advantages of encapsulation

- Protects internal state from uncontrolled modification.
- Allows validation and business rules to be enforced.
- Reduces dependencies on internal implementation details.
- Makes classes easier to maintain and modify.

Quick revision

| Concept | How it is achieved | Main benefit |
|---|---|---|
| **Polymorphism** | Method overloading and overriding | Flexible behavior |
| **Encapsulation** | Access control and controlled operations | Data protection and integrity |

Interview takeaway

**Polymorphism determines which behavior is used, while encapsulation controls how an object's internal state is accessed and modified.** Together, they help create Java applications that are flexible, reusable, and maintainable.

---

## Method Overriding and Exception Handling in Java

When a subclass overrides a method of its parent class, it can throw the **same checked exception, a narrower checked exception, or no checked exception at all**. However, it cannot throw a broader checked exception or an unrelated checked exception that is not permitted by the parent method.

Rules for Exception Handling in Method Overriding

1. **Same exception:** The overriding method can throw the same checked exception as the parent method.
2. **Narrower exception:** It can throw a subclass of the exception declared by the parent method.
3. **No exception:** It can choose not to throw any exception.
4. **Broader checked exception:** It cannot throw a broader checked exception than the parent method declares.
5. **Unchecked exceptions:** It can throw unchecked exceptions, such as `RuntimeException`, without restrictions from the parent's `throws` clause.

Example

```java
import java.io.IOException;

class Animal {

    public void speak() throws Exception {
        System.out.println("Animal speaks");
    }

    public void eat() throws Exception {
        System.out.println("Animal eats");
    }
}

class Dog extends Animal {

    @Override
    public void speak() throws IOException {
        System.out.println("Dog barks");
        throw new IOException("Exception from Dog.speak()");
    }

    @Override
    public void eat() {
        System.out.println("Dog eats");
    }
}

public class Main {

    public static void main(String[] args) {
        Animal animal = new Dog();

        try {
            animal.speak();
        } catch (IOException e) {
            System.out.println("Caught IOException: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("Caught Exception: " + e.getMessage());
        }

        try {
            animal.eat();
        } catch (Exception e) {
            System.out.println("Caught Exception: " + e.getMessage());
        }
    }
}
```

Output

```text
Dog barks
Caught IOException: Exception from Dog.speak()
Dog eats
```

Explanation

- `Animal.speak()` declares `throws Exception`.
- `Dog.speak()` declares `throws IOException`. This is valid because `IOException` is a subclass of `Exception`.
- `Dog.eat()` does not declare any exception. This is also valid because an overriding method may remove the parent's checked-exception declaration.
- Although the reference variable is of type `Animal`, the overridden implementation in `Dog` executes at runtime.

**Important interview point:** The compiler checks the exceptions declared by the reference type when deciding which exceptions must be handled. Since `animal` is declared as `Animal`, calling `animal.speak()` requires handling `Exception`, not just `IOException`. Therefore, the example above will **not compile** as written because the first `catch (IOException e)` is followed by `catch (Exception e)`, but the method call's declared exception is `Exception`, which is handled by the second catch; the first catch is still reachable. The code compiles, and both exceptions are handled correctly.

---

## Can We Overload Methods with Different Return Types in Java?

**No. Java does not allow method overloading based only on different return types.** For overloading, methods must have the same name but different parameter lists.

The compiler determines which overloaded method to call using the method name and argument types, not the return type alone.

Invalid Example

```java
class Calculator {

    public int add(int x, int y) {
        return x + y;
    }

    // Compilation error: duplicate method
    public double add(int x, int y) {
        return (double) (x + y);
    }
}
```

Both methods have the same name and identical parameter types. Changing only the return type does not create a valid overload.

Valid Example

```java
class Calculator {

    public int add(int x, int y) {
        return x + y;
    }

    public double add(double x, double y) {
        return x + y;
    }

    public int add(int x, int y, int z) {
        return x + y + z;
    }
}
```

Here, the methods are overloaded because their parameter lists differ.

Quick Revision

| Concept | Rule |
|---|---|
| Overloading | Same method name, different parameter lists |
| Return type in overloading | Cannot be the only difference |
| Overriding with checked exceptions | Same or narrower checked exception, or none |
| Overriding with broader checked exceptions | Not allowed |
| Overriding with unchecked exceptions | Allowed |

**Interview takeaway:** Method overriding supports runtime polymorphism, and the overriding method cannot broaden the checked exceptions declared by the parent method. Method overloading, meanwhile, is determined by the parameter list—not the return type.

---

## Can We Override a Static Method in Java? Why Not?

**No, we cannot override a static method in Java.** If a subclass defines a static method with the same signature as a static method in its parent class, it is called **method hiding**, not method overriding.

Why Can't We Override a Static Method?

The main reason is that **static methods belong to the class, whereas overridden instance methods are resolved based on the actual object at runtime.**

- **Static methods:** Resolved using the reference type or class name; they are not dynamically dispatched.
- **Instance methods:** Overridden methods support runtime polymorphism, so the actual object's implementation is invoked.

Example: Static Method Hiding

```java
class Parent {

    static void display() {
        System.out.println("Parent static method");
    }
}

class Child extends Parent {

    static void display() {
        System.out.println("Child static method");
    }
}

public class Main {

    public static void main(String[] args) {
        Parent parent = new Child();

        parent.display();
        Child.display();
        Parent.display();
    }
}
```

Output

```text
Parent static method
Child static method
Parent static method
```

Explanation

- `Parent parent = new Child();` creates a `Child` object but stores its reference in a `Parent` variable.
- `parent.display()` calls `Parent.display()` because static method selection depends on the reference type, not the runtime object.
- `Child.display()` calls the static method declared in `Child`.
- `Parent.display()` calls the static method declared in `Parent`.

Notice that the static method in `Child` does not override the method in `Parent`; it **hides** it.

Method Overriding vs. Method Hiding

| Feature | Method Overriding | Method Hiding |
|---|---|---|
| Applies to | Instance methods | Static methods |
| Binding | Runtime (dynamic dispatch) | Compile time (based on reference type) |
| Polymorphism | Supports runtime polymorphism | Does not support runtime polymorphism |
| `@Override` annotation | Valid | Invalid for a static method |
| Resolution | Actual object type | Reference type or class name |

Interview Follow-up: What Happens If We Use `@Override`?

Java produces a compilation error if we annotate a static method in a subclass with `@Override`, because static methods cannot override instance methods or other static methods.

```java
class Parent {
    static void display() {}
}

class Child extends Parent {

    @Override
    static void display() {} // Compilation error
}
```

**Interview takeaway:** Static methods are associated with classes, not individual objects. Therefore, they cannot be overridden; a static method with the same signature in a subclass hides the parent's static method.


