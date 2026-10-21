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


---

# Object Creation and Advanced Core Java Concepts

This guide covers common Core Java interview questions about object creation, immutability, restricting object creation, class loaders, the `main()` method, the diamond problem, and wrapper classes.

## 1. What are the different ways to create an object in Java?

An object is an instance of a class. It typically contains instance state and provides behavior through methods.

Java supports several ways to create or obtain an object.

### Method 1: Using the `new` keyword

This is the most common approach. It invokes a constructor to initialize the object.

```java
class Employee {
    private String name;

    Employee(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

public class Main {
    public static void main(String[] args) {
        Employee employee = new Employee("Danish");

        System.out.println(employee.getName());
    }
}
```

Output:

```text
Danish
```

### Method 2: Using reflection

Reflection allows an application to create objects dynamically at runtime.

```java
import java.lang.reflect.Constructor;

class Employee {
    public Employee() {
        System.out.println("Employee created");
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = Class.forName("Employee");

        Constructor<?> constructor =
                clazz.getDeclaredConstructor();

        Object employee = constructor.newInstance();

        System.out.println(employee.getClass().getSimpleName());
    }
}
```

Output:

```text
Employee created
Employee
```

`Class.forName()` loads or initializes a class as specified by its API; it does not itself create an instance. `Constructor.newInstance()` performs the instantiation.

**Interview note:** `Class.newInstance()` is deprecated. Prefer `getDeclaredConstructor().newInstance()`.

### Method 3: Using `clone()`

An object can be copied using the cloning mechanism when the class supports it.

```java
class Employee implements Cloneable {
    String name;

    Employee(String name) {
        this.name = name;
    }

    @Override
    public Employee clone() {
        try {
            return (Employee) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(e);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Employee first = new Employee("Ali");
        Employee second = first.clone();

        System.out.println(first.name);
        System.out.println(second.name);
        System.out.println(first == second);
    }
}
```

Output:

```text
Ali
Ali
false
```

The two objects are different instances, but `clone()` normally performs a shallow copy. Mutable objects referenced by fields may still be shared.

### Method 4: Using deserialization

Deserialization can reconstruct an object from a serialized representation without invoking the object's ordinary constructor in the usual way.

```java
import java.io.*;

class Employee implements Serializable {
    private static final long serialVersionUID = 1L;

    String name;

    Employee(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        Employee employee = new Employee("Sara");

        try (ObjectOutputStream out = new ObjectOutputStream(
                new FileOutputStream("employee.ser"))) {
            out.writeObject(employee);
        }

        try (ObjectInputStream in = new ObjectInputStream(
                new FileInputStream("employee.ser"))) {
            Employee copy = (Employee) in.readObject();

            System.out.println(copy.name);
        }
    }
}
```

Output:

```text
Sara
```

Use Java serialization only where appropriate. Never deserialize untrusted data without suitable safeguards.

### Method 5: Using a factory method

A factory method creates and returns an object. The method can choose which implementation to instantiate.

```java
interface Notification {
    void send(String message);
}

class EmailNotification implements Notification {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class NotificationFactory {
    public static Notification create(String type) {
        if ("email".equalsIgnoreCase(type)) {
            return new EmailNotification();
        }

        throw new IllegalArgumentException(
                "Unsupported notification type: " + type);
    }
}

public class Main {
    public static void main(String[] args) {
        Notification notification =
                NotificationFactory.create("email");

        notification.send("Hello");
    }
}
```

Output:

```text
Email: Hello
```

Factory methods are useful when object creation involves validation, configuration, or selecting an implementation.

### Quick comparison

| Approach | Typical use |
|---|---|
| `new` | Ordinary object creation |
| Reflection | Runtime-dynamic instantiation |
| `clone()` | Copying an existing object |
| Deserialization | Reconstructing serialized objects |
| Factory method | Encapsulating creation logic |

## 2. How do you create an immutable class?

An immutable class is a class whose observable state cannot change after the object has been constructed.

### Rules for creating an immutable class

1. Declare the class `final`, or otherwise prevent subclasses from introducing mutability.
2. Make instance fields `private` and preferably `final`.
3. Initialize the fields through the constructor.
4. Do not provide setters or methods that mutate internal state.
5. Defensively copy mutable inputs.
6. Do not expose mutable internal objects directly.

### Example

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

public final class Employee {
    private final String name;
    private final int age;
    private final List<String> skills;

    public Employee(String name, int age, List<String> skills) {
        this.name = Objects.requireNonNull(name);
        this.age = age;
        this.skills = new ArrayList<>(skills);
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public List<String> getSkills() {
        return List.copyOf(skills);
    }
}
```

Usage:

```java
List<String> skills = new ArrayList<>();
skills.add("Java");

Employee employee = new Employee("Danish", 30, skills);

skills.add("Python");

System.out.println(employee.getSkills());
```

Output:

```text
[Java]
```

The employee's state is unaffected by later changes to the original input list. The getter returns an unmodifiable copy of the list contents.

For Java 10+, `List.copyOf()` is another convenient way to create an unmodifiable defensive copy.

**Important:** `final` on a reference prevents reassignment of the reference; it does not automatically make the referenced object immutable.

## 3. How can you restrict object creation in Java?

There are several approaches, depending on whether you want to restrict creation completely, control it through a factory, or ensure a single instance.

### Method 1: Private constructor

```java
public class Utility {
    private Utility() {
        // Prevent external instantiation
    }

    public static int square(int number) {
        return number * number;
    }
}
```

Usage:

```java
System.out.println(Utility.square(5));
```

Output:

```text
25
```

The constructor cannot be called from unrelated classes. A private constructor is useful for utility classes, factories, and controlled instance creation.

### Method 2: Singleton pattern

A singleton provides one shared instance through its intended access API.

For a simple, robust singleton, an enum is a good option:

```java
enum ApplicationConfig {
    INSTANCE;

    public void display() {
        System.out.println("Application configuration");
    }
}

public class Main {
    public static void main(String[] args) {
        ApplicationConfig.INSTANCE.display();
    }
}
```

Output:

```text
Application configuration
```

Enum singletons provide serialization guarantees and resist ordinary reflective instantiation.

A conventional lazy singleton can also be implemented with a holder class:

```java
class DatabaseConnection {
    private DatabaseConnection() {}

    private static class Holder {
        private static final DatabaseConnection INSTANCE =
                new DatabaseConnection();
    }

    public static DatabaseConnection getInstance() {
        return Holder.INSTANCE;
    }
}
```

The holder class is initialized when `getInstance()` first accesses it, and class initialization provides thread safety.

### Method 3: Abstract class or interface

An abstract class or interface cannot be instantiated directly.

```java
abstract class Shape {
    abstract double area();
}

interface Payment {
    void pay(double amount);
}
```

Concrete subclasses and implementing classes can still be instantiated. This approach restricts direct creation of the abstract type, not object creation altogether.

### Method 4: Factory method

A factory can control which objects are created and under what conditions.

```java
class User {
    private final String name;

    private User(String name) {
        this.name = name;
    }

    public static User create(String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name is required");
        }

        return new User(name);
    }

    public String getName() {
        return name;
    }
}
```

The constructor is private, so callers must use `User.create()`.

## 4. How do you create a custom class loader?

A class loader loads class definitions into the JVM. Java provides built-in class loaders, and applications can create custom loaders for specialized requirements such as plugin systems.

A typical approach is to extend `ClassLoader` and override `findClass()`, rather than overriding `loadClass()` without preserving its parent-delegation behavior.

### Example: Load a class from a directory

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class DirectoryClassLoader extends ClassLoader {

    private final Path root;

    public DirectoryClassLoader(Path root, ClassLoader parent) {
        super(parent);
        this.root = root;
    }

    @Override
    protected Class<?> findClass(String name)
            throws ClassNotFoundException {

        Path classFile = root.resolve(
                name.replace('.', '/') + ".class");

        try {
            byte[] bytes = Files.readAllBytes(classFile);

            return defineClass(
                    name, bytes, 0, bytes.length);

        } catch (IOException e) {
            throw new ClassNotFoundException(
                    "Could not load " + name, e);
        }
    }
}
```

Usage:

```java
import java.nio.file.Path;

public class Main {
    public static void main(String[] args)
            throws Exception {

        try (var loader = new DirectoryClassLoader(
                Path.of("/tmp/plugin-classes"),
                Main.class.getClassLoader())) {

            Class<?> pluginClass =
                    loader.loadClass("plugins.HelloPlugin");

            Object plugin = pluginClass
                    .getDeclaredConstructor()
                    .newInstance();

            System.out.println(plugin.getClass().getName());
        }
    }
}
```

The class file for `plugins.HelloPlugin` must exist at `/tmp/plugin-classes/plugins/HelloPlugin.class`, with the correct package declaration and accessible no-argument constructor.

### Important interview points

- `loadClass()` generally follows parent delegation before calling `findClass()`.
- `findClass()` is the usual extension point for loading classes from a custom source.
- `defineClass()` converts valid class-file bytes into a JVM class definition.
- Classes with the same fully qualified name but different defining class loaders are distinct JVM types.
- Custom class loaders require careful security, lifecycle, and class-identity handling.

## 5. Can we write the main method without `static`?

The answer depends on the Java version.

### Before Java 25

The conventional entry point required `public static void main(String[] args)`.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

A method declared as `public void main(String[] args)` was not a valid conventional entry point for launching the class.

### Java 25 and later

Java 25 finalized support for instance `main` methods. A suitable instance entry point can be launched without declaring `static`.

```java
class Main {
    void main() {
        System.out.println("Hello Java 25");
    }
}
```

With Java 25 or later, the launcher can create an instance of `Main` and invoke its eligible instance `main()` method.

A supported instance entry point can also accept command-line arguments:

```java
class Main {
    void main(String[] args) {
        System.out.println("Arguments: " + args.length);
    }
}
```

For traditional applications, interviews, and compatibility with older Java versions, continue to use:

```java
public static void main(String[] args)
```

Reference: [Oracle Java 25 language updates](https://docs.oracle.com/en/java/javase/25/language/java-se-language-updates.pdf).

## 6. Explain the diamond problem in Java

The diamond problem occurs when a class inherits the same method through multiple inheritance paths, potentially creating ambiguity about which implementation should be used.

Java does not allow a class to extend multiple classes. However, interface default methods can introduce a related conflict.

### Example: Conflicting default methods

```java
interface A {
    default void display() {
        System.out.println("A");
    }
}

interface B {
    default void display() {
        System.out.println("B");
    }
}

class C implements A, B {

    @Override
    public void display() {
        A.super.display();
        B.super.display();

        System.out.println("C");
    }
}

public class Main {
    public static void main(String[] args) {
        new C().display();
    }
}
```

Output:

```text
A
B
C
```

Because both interfaces provide a default implementation of `display()`, the implementing class must resolve the conflict.

### How Java resolves default-method conflicts

1. **Class wins:** An applicable concrete superclass method takes precedence over an interface default method.
2. **More specific interface wins:** If one interface extends another and both supply a matching default method, the more specific interface's implementation takes precedence.
3. **Explicit resolution:** If unrelated interfaces provide conflicting defaults, the class must override the method and can invoke a particular implementation with `A.super.display()`.

Java's single class inheritance avoids the classic ambiguity associated with inheriting state and implementations from multiple classes.

## 7. What is a wrapper class in Java?

A wrapper class represents a primitive value as an object.

Java provides these built-in wrapper classes:

| Primitive type | Wrapper class |
|---|---|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

Wrapper classes are useful with collections, generics, APIs that require objects, and operations such as parsing and conversion.

### Example: Integer wrapper

```java
public class Main {
    public static void main(String[] args) {
        int primitive = 10;

        Integer wrapped = primitive; // Autoboxing
        int unwrapped = wrapped;     // Unboxing

        System.out.println(wrapped);
        System.out.println(unwrapped);

        int number = Integer.parseInt("123");
        System.out.println(number);
    }
}
```

Output:

```text
10
10
123
```

### Autoboxing and unboxing

**Autoboxing:** Java automatically converts a primitive to its wrapper object.

```java
Integer number = 10;
```

**Unboxing:** Java automatically converts a wrapper object to its primitive value.

```java
int value = number;
```

Be careful when unboxing a `null` wrapper:

```java
Integer number = null;

// Throws NullPointerException
// int value = number;
```

### Should you write a custom wrapper class?

Usually, use Java's built-in wrapper classes. They already provide conversion, comparison, parsing, and other useful operations.

A custom wrapper is appropriate when you need to encapsulate additional domain-specific behavior.

```java
public final class IntWrapper {
    private int value;

    public IntWrapper(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    public void increment() {
        value++;
    }

    public void decrement() {
        value--;
    }

    @Override
    public String toString() {
        return Integer.toString(value);
    }
}
```

Usage:

```java
IntWrapper number = new IntWrapper(5);

number.increment();

System.out.println(number.getValue());
System.out.println(number);
```

Output:

```text
6
6
```

Unlike the built-in `Integer`, this custom wrapper is mutable because its `value` can change. Wrapper classes do not have to be immutable, but immutable designs are often easier to reason about.

## 8. Quick revision

- **Object creation:** `new`, reflection, cloning, deserialization, and factory methods.
- **Immutable class:** Prevent changes to observable state after construction; use defensive copies for mutable data.
- **Restricting creation:** Private constructors, factories, singleton patterns, abstract classes, and interfaces.
- **Custom class loader:** Usually extend `ClassLoader` and implement `findClass()`.
- **Main method:** The traditional static entry point works across older Java versions; Java 25+ also supports eligible instance entry points.
- **Diamond problem:** Java requires conflicts between unrelated interface default methods to be resolved.
- **Wrapper classes:** Object representations of primitives, with autoboxing and unboxing support.

---

# Clone Method in Java

## 1. What is the clone() method in Java?

The `clone()` method is defined in Java's `Object` class and is used to create a copy of an existing object.

By default, `Object.clone()` performs a **shallow copy**: it creates a new object and copies the field values from the original object. Primitive fields are copied by value, while reference fields retain references to the same referenced objects.

The method is `protected` in `Object`, so a class typically overrides it to expose cloning through its own API.

### Why use clone()?

- **Copy an object's state:** Create a separate object with the same initial field values.
- **Modify a copy independently:** This works directly for primitive fields, but mutable objects referenced by fields may still be shared.
- **Avoid repeating initialization:** Copy an existing object's state when the class's cloning contract makes that appropriate.

Cloning is not necessarily the best choice for backups or creating default objects. Explicit copy constructors and factory methods are often easier to understand and maintain.

## 2. What is the Cloneable interface?

`Cloneable` is a marker interface in `java.lang`. It does not declare any methods.

It indicates that `Object.clone()` is permitted to create a field-by-field copy of instances of that class.

If a class does not implement `Cloneable`, calling `super.clone()` normally throws `CloneNotSupportedException`.

## 3. How to use clone() in Java

Consider this example:

```java
class Employee implements Cloneable {
    private int age;

    public Employee(int age) {
        this.age = age;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public Employee clone() {
        try {
            return (Employee) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(e);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Employee original = new Employee(25);
        Employee copy = original.clone();

        System.out.println(original.getAge());
        System.out.println(copy.getAge());

        copy.setAge(30);

        System.out.println(original.getAge());
        System.out.println(copy.getAge());

        System.out.println(original == copy);
    }
}
```

Output:

```text
25
25
25
30
false
```

### Explanation

1. `Employee` implements `Cloneable`.
2. The `clone()` method calls `super.clone()` to create a shallow copy.
3. Both objects initially have the same age.
4. Updating the primitive `age` field in the copy does not affect the original.
5. `original == copy` is `false` because they are distinct objects.

The covariant return type `Employee` allows callers to avoid casting the returned object.

## 4. Shallow copy vs deep copy

This is one of the most important clone-related interview questions.

### Shallow copy

A shallow copy creates a new outer object but copies reference fields as references. Both objects can therefore point to the same mutable nested object.

```java
class Address {
    String city;

    Address(String city) {
        this.city = city;
    }
}

class Employee implements Cloneable {
    String name;
    Address address;

    Employee(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    @Override
    public Employee clone() {
        try {
            return (Employee) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(e);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Employee first =
                new Employee("Ali", new Address("Delhi"));

        Employee second = first.clone();

        second.address.city = "Mumbai";

        System.out.println(first.address.city);
        System.out.println(second.address.city);
    }
}
```

Output:

```text
Mumbai
Mumbai
```

Both employees reference the same `Address` object.

### Deep copy

A deep copy creates independent copies of the mutable nested objects that need to be independent.

One approach is to copy the nested object explicitly:

```java
class Address {
    String city;

    Address(String city) {
        this.city = city;
    }

    Address(Address other) {
        this.city = other.city;
    }
}

class Employee {
    String name;
    Address address;

    Employee(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    public Employee deepCopy() {
        return new Employee(
                this.name,
                new Address(this.address)
        );
    }
}

public class Main {
    public static void main(String[] args) {
        Employee first =
                new Employee("Ali", new Address("Delhi"));

        Employee second = first.deepCopy();

        second.address.city = "Mumbai";

        System.out.println(first.address.city);
        System.out.println(second.address.city);
    }
}
```

Output:

```text
Delhi
Mumbai
```

The `Address` objects are now independent.

### Comparison

| Feature | Shallow copy | Deep copy |
|---|---|---|
| New outer object | Yes | Yes |
| Primitive fields | Values copied | Values copied |
| Reference fields | References copied | Relevant nested objects copied |
| Mutable nested state | May be shared | Independent when correctly copied |
| Implementation | Usually simpler | Requires more care |

A deep copy does not automatically mean recursively duplicating every referenced object. The required copy depth depends on the class's ownership and sharing rules.

## 5. clone() vs copy constructor

A copy constructor is often a clearer alternative to `clone()`.

```java
class Employee {
    private final String name;
    private final int age;

    public Employee(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Employee(Employee other) {
        this.name = other.name;
        this.age = other.age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

Usage:

```java
Employee first = new Employee("Danish", 30);
Employee second = new Employee(first);
```

| `clone()` | Copy constructor |
|---|---|
| Uses the `Cloneable` marker interface for `Object.clone()` | Does not require `Cloneable` |
| Default implementation performs a shallow copy | Copy behavior is explicitly defined |
| Inheritance and nested mutable state need careful handling | Easier to customize and document |
| May involve `CloneNotSupportedException` | Usually avoids cloning-specific checked exceptions |

## 6. Important interview questions

**Q1. Is clone() a constructor?**

No. It is a method inherited from `Object`. `Object.clone()` creates an object copy without invoking the ordinary constructor of the object's class.

**Q2. What happens if a class does not implement Cloneable?**

Calling `super.clone()` normally throws `CloneNotSupportedException`.

**Q3. Does clone() perform a deep copy?**

No. The default implementation performs a shallow copy. You must explicitly copy nested mutable objects when deep-copy behavior is required.

**Q4. Does clone() call the constructor?**

No. `Object.clone()` does not invoke the cloned class's ordinary constructor. This is one reason copy constructors can be easier to control.

**Q5. Can we override clone() with a different return type?**

Yes. Java supports covariant return types. For example, a method declared as `public Employee clone()` can override the inherited method whose return type is `Object`.

**Q6. Is cloning recommended in modern Java?**

It is supported, but it is often less convenient than copy constructors or static copy factory methods. Prefer explicit copying when it makes the intended behavior clearer.

## 7. Quick revision

- `Object.clone()` creates a shallow, field-by-field copy by default.
- `Cloneable` is a marker interface.
- `super.clone()` throws `CloneNotSupportedException` when the object is not cloneable.
- Primitive fields are copied independently; mutable objects referenced by fields may be shared.
- Deep copying requires explicitly copying the relevant nested mutable objects.
- Copy constructors and static copy factories are often clearer alternatives.


---

# Serialization and Deserialization in Java

## 1. What Is Serialization?

**Serialization** is the process of converting an object's state into a byte stream so that it can be stored or transmitted.

**Deserialization** is the reverse process: reconstructing an object from that byte stream.

Java provides `Serializable`, `ObjectOutputStream`, and `ObjectInputStream` to support its built-in object serialization mechanism.

### Why Is Serialization Needed?

Serialization is useful when an application's object state needs to be transferred or saved beyond the lifetime of the current object.

Common use cases include:

- **Persistence:** Saving object state to a file.
- **Network communication:** Transferring object data between Java applications.
- **Distributed systems:** Supporting mechanisms such as Java RMI.
- **Caching:** Storing object state for later retrieval, when a compatible serialization format is appropriate.
- **Object copying:** Creating deep copies in certain cases, provided the entire reachable object graph is serializable.

Serialization is not the only way to achieve these goals. Modern applications often use JSON, Protocol Buffers, database persistence, or other formats depending on interoperability, performance, and security requirements.

## 2. What Is the Serializable Interface?

`java.io.Serializable` is a **marker interface**. It does not declare methods; it indicates that a class supports Java's built-in serialization mechanism.

```java
import java.io.Serializable;

class Person implements Serializable {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

An object can be serialized with `ObjectOutputStream` when its class implements `Serializable` and the required serialization conditions are satisfied.

If a non-transient, non-static field refers to an object that is not serializable, serialization will normally fail with `NotSerializableException`, unless the class handles that field specially.

## 3. How Does Serialization Work?

The process follows these steps:

1. Create an object that supports serialization.
2. Create an `ObjectOutputStream` connected to an output stream.
3. Call `writeObject()` to serialize the object.
4. Store or transmit the resulting bytes.
5. Create an `ObjectInputStream` connected to the corresponding input stream.
6. Call `readObject()` to reconstruct the object.

```text
       Java Object
            |
            v
   ObjectOutputStream
            |
            v
       Byte Stream
            |
      File / Network
            |
            v
    ObjectInputStream
            |
            v
   Reconstructed Object
```

Java serialization preserves the object's serialized state and object-graph relationships. It does not simply write every field indiscriminately: static fields are not serialized as instance state, transient fields are skipped by default, and custom serialization methods can alter the behavior.

## 4. Serialization Example in Java

The following example serializes a `Person` object to a file named `person.ser`.

```java
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectOutputStream;
import java.io.Serializable;

class Person implements Serializable {
    private static final long serialVersionUID = 1L;

    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}

public class SerializationDemo {
    public static void main(String[] args) {
        Person person = new Person("John", 25);

        try (ObjectOutputStream out = new ObjectOutputStream(
                new FileOutputStream("person.ser"))) {

            out.writeObject(person);
            System.out.println("Object serialized successfully.");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**Output:**

```text
Object serialized successfully.
```

The `try-with-resources` statement automatically closes the stream. The file contains a Java serialization stream, not a human-readable JSON document.

## 5. How to Deserialize an Object

Use `ObjectInputStream` and its `readObject()` method to reconstruct the object.

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.io.ObjectInputStream;

public class DeserializationDemo {
    public static void main(String[] args) {
        try (ObjectInputStream in = new ObjectInputStream(
                new FileInputStream("person.ser"))) {

            Person person = (Person) in.readObject();

            System.out.println(person.getName());
            System.out.println(person.getAge());

        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

**Output:**

```text
John
25
```

Run the serialization example first to create `person.ser`, then run the deserialization example using the same class definition and a compatible classpath.

`readObject()` returns an `Object`, so a cast is required in this example. It can also throw `ClassNotFoundException` if the class needed to reconstruct the object cannot be found.

## 6. What Is serialVersionUID?

`serialVersionUID` is a version identifier used by Java's serialization mechanism to check compatibility between the serialized data and the class being used to deserialize it.

```java
private static final long serialVersionUID = 1L;
```

### Why Is It Used?

- Helps identify compatible class versions.
- Avoids relying on an automatically computed identifier that can change with class evolution.
- Makes serialization compatibility decisions more explicit.

If the serialized object's class descriptor contains a different `serialVersionUID` from the local class, deserialization normally fails with `InvalidClassException`.

However, using the same ID does not guarantee that every class change is compatible. Developers must still manage changes to serialized fields and class behavior carefully.

## 7. What Is the Difference Between transient, static, and Normal Fields?

| Field type | Serialized as part of the object's default state? | Explanation |
|---|---|---|
| Normal instance field | Yes, if serializable | Its value is normally saved |
| `transient` instance field | No | Excluded from default serialization |
| `static` field | No | Belongs to the class, not an individual object's serialized state |
| `static transient` field | No | Both modifiers exclude it from instance-state serialization |

Example:

```java
import java.io.Serializable;

class User implements Serializable {
    private static final long serialVersionUID = 1L;

    String username = "danish";
    transient String password = "secret";
    static String applicationName = "MyApp";
}
```

During default serialization, `username` is included, while `password` and `applicationName` are not serialized as instance state.

After ordinary default deserialization, a transient field generally has its default value unless custom deserialization logic restores it. For reference fields, the default value is `null`; for numeric primitives, it is zero.

**Important:** `transient` is not a complete security mechanism. Avoid serializing secrets unnecessarily, and never assume a serialized file is encrypted.

## 8. What Happens If a Parent Class Does Not Implement Serializable?

When a serializable class extends a non-serializable superclass:

- The subclass's serializable state can still be restored.
- The non-serializable superclass must have an accessible no-argument constructor so its state can be initialized during deserialization.
- The superclass's fields are not restored through the normal serialization mechanism; they are initialized by the superclass constructor.

If the required constructor is unavailable, deserialization can fail.

This is an important interview detail when working with inheritance and serializable classes.

## 9. What Is Custom Serialization?

Java allows a class to customize its default serialization behavior using specially named methods such as:

```java
private void writeObject(
        java.io.ObjectOutputStream out) throws java.io.IOException {
    out.defaultWriteObject();
}

private void readObject(
        java.io.ObjectInputStream in)
        throws java.io.IOException, ClassNotFoundException {
    in.defaultReadObject();
}
```

These methods can supplement default serialization, for example to validate or reconstruct state. They must use the appropriate signatures to be recognized by Java's serialization mechanism.

Other mechanisms include:

- `Externalizable`: Gives the class explicit control through `writeExternal()` and `readExternal()`.
- `writeReplace()` and `readResolve()`: Can customize object replacement during serialization and deserialization.

Custom serialization should be designed carefully, particularly when classes evolve or serialized data may come from an untrusted source.

## 10. Serialization vs Deserialization

| Feature | Serialization | Deserialization |
|---|---|---|
| Purpose | Convert object state into a byte stream | Reconstruct an object from a byte stream |
| Main class | `ObjectOutputStream` | `ObjectInputStream` |
| Main method | `writeObject()` | `readObject()` |
| Direction | Object → bytes | Bytes → object |
| Common exception | `IOException` | `IOException`, `ClassNotFoundException` |

## 11. Serialization vs JSON

| Feature | Java serialization | JSON |
|---|---|---|
| Representation | Java-specific binary stream | Text-based data format |
| Human-readable | Generally no | Yes |
| Java class compatibility | Depends on class availability and serialization compatibility | Can be consumed by many languages |
| Object graph references | Supports reference sharing and cycles | Usually requires explicit conventions for references and cycles |
| Typical use | Java-specific persistence or communication where appropriate | APIs, configuration, cross-language data exchange |

For modern REST APIs and cross-language communication, JSON or a schema-based format is often more suitable than Java native serialization.

## 12. Advantages and Disadvantages

### Advantages

- Provides built-in object serialization support.
- Can preserve object graphs and shared references.
- Supports Java-specific use cases such as legacy RMI systems.
- Can simplify some object persistence or transfer workflows.

### Disadvantages

- The format is Java-specific and can be sensitive to class changes.
- Serialized data may consume substantial storage or bandwidth.
- Deserializing untrusted data can create serious security vulnerabilities.
- Compatibility management and long-term storage can be difficult.
- It may be less interoperable than JSON or schema-based formats.

**Security warning:** Never deserialize untrusted Java serialization data without an appropriate security strategy. Native deserialization can trigger dangerous behavior through classes available on the classpath. Prefer safer formats where practical, and use strict deserialization filters when native serialization is unavoidable.

## 13. Common Interview Questions

### Q1. Is Serializable a marker interface?

Yes. It declares no methods; it marks classes as eligible for Java's built-in serialization mechanism.

### Q2. Are static variables serialized?

No. Static variables belong to the class, not an individual object, so they are not serialized as part of the object's default instance state.

### Q3. What is the purpose of transient?

It excludes an instance field from default serialization. Custom serialization code can still choose to handle that field explicitly.

### Q4. Does serialization call the constructor of a serializable class during deserialization?

For ordinary Java serialization, constructors and field initializers of serializable classes are not invoked as they normally are during construction. The no-argument constructor of the first non-serializable superclass is invoked to initialize that superclass's state.

### Q5. Can we serialize an object without implementing Serializable?

Not through the normal default mechanism for an ordinary class. If the object does not support serialization, `ObjectOutputStream.writeObject()` normally throws `NotSerializableException`. Alternatives include a custom external representation, a suitable serialization framework, or `Externalizable`, which itself extends `Serializable`.

### Q6. Is serialization the same as cloning?

No. Serialization converts object state into a stream. Cloning creates another object. A serialization round trip can sometimes create a deep copy, but it has extra requirements and is not generally the preferred cloning technique.

## 14. One-Line Interview Answer

Serialization in Java converts an object's state into a byte stream using mechanisms such as `Serializable` and `ObjectOutputStream`, enabling storage or transfer; deserialization uses `ObjectInputStream` to reconstruct the object.


---

