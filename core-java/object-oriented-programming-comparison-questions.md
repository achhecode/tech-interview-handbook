---
title: OOP Concepts — Comparison Questions
parent: Core Java
nav_order: 2
---

# OOP Concepts — Comparison Questions
{: .no_toc }

Abstraction, encapsulation, polymorphism, inheritance, composition, and aggregation.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

  1. TOC
  {:toc}
</details>

---

## 1. What is the difference between Abstraction and Encapsulation?

Medium
{: .label .label-yellow }

**Abstraction** hides implementation details and exposes only the essential functionality. **Encapsulation** bundles data and related behavior within a class while controlling access to its internal state.

For example, a payment interface provides abstraction by defining how a payment can be initiated, while a bank account class uses encapsulation to protect its balance.

| Feature | Abstraction | Encapsulation |
|---|---|---|
| **Focus** | What an object does. | How data and behavior are organized and protected. |
| **Goal** | Reduce complexity by exposing essential operations. | Control access to data and preserve object validity. |
| **Java mechanisms** | Interfaces and abstract classes. | Classes, access modifiers, and controlled methods. |
| **Example** | A `Payment` interface defines `pay()`. | A `BankAccount` keeps `balance` private. |

Java example

```java
interface Payment {
    void pay(double amount);
}

class BankAccount {
    private double balance;

    public BankAccount(double balance) {
        if (balance < 0) {
            throw new IllegalArgumentException("Invalid balance");
        }
        this.balance = balance;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Invalid amount");
        }
        balance += amount;
    }
}
```

**Explanation:**
- `Payment` demonstrates abstraction by defining a contract without specifying how payment works.
- `BankAccount` demonstrates encapsulation by keeping `balance` private and allowing changes only through validated methods.

**Interview takeaway:** Abstraction hides implementation complexity; encapsulation controls access to an object's internal state.

---

## 2. What is the difference between Abstraction and Polymorphism?

Medium
{: .label .label-yellow }

**Abstraction** defines the essential operations available to a user while hiding implementation details. **Polymorphism** allows different implementations or types to respond differently to the same method call.

| Feature | Abstraction | Polymorphism |
|---|---|---|
| **Focus** | Exposing essential functionality. | Supporting different behaviors through a common operation or method name. |
| **Goal** | Simplify usage and hide implementation details. | Make code flexible and extensible. |
| **Mechanisms** | Interfaces and abstract classes. | Method overloading and method overriding. |
| **Example** | A `Payment` interface declares `pay()`. | UPI and card payments implement `pay()` differently. |

Java example

```java
interface Payment {
    void pay(double amount);
}

class UpiPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("UPI payment: " + amount);
    }
}

class CardPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Card payment: " + amount);
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

**Explanation:**

- The `Payment` interface provides abstraction by declaring the `pay()` contract.
- `UpiPayment` and `CardPayment` provide different implementations.
- The same `payment.pay()` call executes the implementation associated with the actual object at runtime, demonstrating runtime polymorphism.

**Interview takeaway:** Abstraction defines what functionality is available; polymorphism allows that functionality to behave differently across implementations.

---

## 3. What is the difference between Inheritance and Composition?

Medium
{: .label .label-yellow }

**Inheritance** allows a class to extend another class and reuse or specialize its accessible behavior. **Composition** allows a class to build its functionality by holding references to other objects and delegating work to them.

Inheritance models an **is-a** relationship, whereas composition commonly models a **has-a** relationship.

Java example: Inheritance

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

Here, `Car` is a `Vehicle` and inherits its accessible `start()` behavior.

Java example: Composition

```java
class Engine {
    void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private final Engine engine = new Engine();

    void start() {
        engine.start();
        System.out.println("Car is ready");
    }
}
```

Here, `Car` has an `Engine` and delegates engine startup to it.

Key differences

| Feature | Inheritance | Composition |
|---|---|---|
| **Relationship** | Is-a | Has-a |
| **Implementation** | Uses `extends` in Java. | Stores or receives references to other objects. |
| **Code reuse** | Reuses inherited behavior. | Reuses behavior through delegation. |
| **Flexibility** | The class hierarchy is generally fixed at compile time. | Dependencies can often be replaced or configured at runtime. |
| **Coupling** | Can tightly couple subclasses to parent implementation details. | Often reduces coupling when components depend on suitable abstractions. |
| **Best suited for** | Genuine subtype relationships. | Combining independent capabilities or components. |

**Interview takeaway:** Prefer composition when a class needs another object's functionality without being a subtype of that object. Use inheritance when the subtype genuinely satisfies the parent type's contract.

---

## 4. What are Composition and Aggregation? Explain with examples.

Medium
{: .label .label-yellow }

**Composition** and **aggregation** are two forms of association between objects. Both describe a whole-part relationship, but they differ in how strongly the whole owns its parts.

- **Composition:** A strong ownership relationship. A part is treated as belonging to its containing object, and its lifecycle is typically managed as part of that whole.
- **Aggregation:** A weaker relationship. The containing object references another object that can exist independently and may be shared.

Java example: Composition

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

Here, `Car` creates and owns its `Engine`. The example illustrates composition through ownership and construction inside `Car`.

Java example: Aggregation

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

University university =
    new University(List.of(s1, s2));

university.displayStudents();
```

Here, the `Student` objects are created independently and then passed to `University`. They can exist independently of the university object, illustrating aggregation.

Key differences

| Feature | Composition | Aggregation |
|---|---|---|
| **Ownership** | Strong ownership by the whole. | Weak association or shared reference. |
| **Part's lifecycle** | Usually managed as part of the whole. | Independent of the whole. |
| **Object creation** | The whole often creates or exclusively owns the part. | Parts are often created externally and supplied to the whole. |
| **Example** | A car owning its engine as an internal component in a simplified model. | A university referencing independently existing students. |

**Important:** Java does not have dedicated keywords for composition or aggregation. These relationships are expressed through object references and design choices. A reference alone does not establish composition; ownership and lifecycle semantics matter.

**Interview takeaway:** Composition represents stronger ownership; aggregation represents a weaker relationship in which the part can exist independently.

---

Quick revision

| Concept | Remember this |
|---|---|
| **Abstraction** | Expose what an object can do; hide implementation details. |
| **Encapsulation** | Protect internal state through controlled access. |
| **Polymorphism** | The same operation can exhibit different behaviors. |
| **Inheritance** | Reuse or specialize behavior through an is-a relationship. |
| **Composition** | Build functionality by owning or combining other objects. |
| **Aggregation** | Reference objects that can exist independently. |

Common follow-up interview questions

- Can abstraction be achieved without an abstract class?
- Is encapsulation possible without getters and setters?
- What is the difference between compile-time and runtime polymorphism?
- Why is composition often preferred over inheritance?
- Can Java implement multiple interfaces?
- Does aggregation require a special Java keyword?
- What is the difference between association, aggregation, and composition?