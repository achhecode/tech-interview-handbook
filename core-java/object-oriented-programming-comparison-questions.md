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


---

---
title: Abstract Class vs Interface in Java
parent: Core Java
nav_order: 22
---

# Abstract Class vs Interface in Java

Java provides `abstract` classes and `interface` types to support abstraction, polymorphism, and flexible application design.

## 1. What is an abstract class?

An abstract class is declared using the `abstract` keyword. It cannot be instantiated directly and can contain both abstract methods and concrete methods.

An abstract method declares a method without providing its implementation.

### Example

```java
abstract class Animal {

    protected String name;

    Animal(String name) {
        this.name = name;
    }

    // Abstract method
    abstract void makeSound();

    // Concrete method
    void sleep() {
        System.out.println(name + " is sleeping");
    }
}

class Dog extends Animal {

    Dog(String name) {
        super(name);
    }

    @Override
    void makeSound() {
        System.out.println("Woof!");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog("Tommy");

        dog.makeSound();
        dog.sleep();
    }
}
```

Output:

```text
Woof!
Tommy is sleeping
```

The `Animal` class defines shared state (`name`), a constructor, an abstract behavior (`makeSound()`), and a concrete behavior (`sleep()`).

## 2. What is an interface?

An interface defines a contract that implementing classes agree to fulfill.

An interface cannot be instantiated directly. Its fields are implicitly `public static final`, and its abstract methods are implicitly `public abstract` unless they are declared as `default`, `static`, or `private` methods where permitted.

### Example

```java
interface Payment {
    void pay(double amount);
}

class CreditCardPayment implements Payment {

    @Override
    public void pay(double amount) {
        System.out.println(
            "Paid ₹" + amount + " using credit card"
        );
    }
}

class UpiPayment implements Payment {

    @Override
    public void pay(double amount) {
        System.out.println(
            "Paid ₹" + amount + " using UPI"
        );
    }
}

public class Main {
    public static void main(String[] args) {
        Payment payment = new UpiPayment();
        payment.pay(1500);
    }
}
```

Output:

```text
Paid ₹1500.0 using UPI
```

Both payment implementations satisfy the same `Payment` contract, but each provides its own behavior.

## 3. Abstract class vs interface

| Feature | Abstract class | Interface |
|---|---|---|
| Keyword | `abstract class` | `interface` |
| Instantiation | Cannot be instantiated directly | Cannot be instantiated directly |
| Methods | Abstract and concrete methods | Abstract, default, static, and private methods, as permitted |
| Instance variables | Allowed | Not allowed |
| Fields | Instance and static fields | Implicitly `public static final` |
| Constructors | Allowed | Not allowed |
| Inheritance | A class can extend only one class | A class can implement multiple interfaces |
| Access modifiers | Methods and fields can use applicable access modifiers | Abstract interface methods are implicitly public; default/static methods are public unless private methods are explicitly declared |
| State management | Can maintain per-object state | Does not hold per-object instance state |
| Primary purpose | Shared state and implementation | Contract and capabilities |

## 4. Can an interface have implemented methods?

Yes. Modern Java interfaces can contain several kinds of methods.

### A. Abstract method

```java
interface Printer {
    void print();
}
```

Implementing classes must provide the implementation unless they are abstract themselves.

### B. Default method — Java 8+

```java
interface Printer {
    default void connect() {
        System.out.println("Printer connected");
    }

    void print();
}
```

A default method provides an implementation that implementing classes can inherit or override.

### C. Static method — Java 8+

```java
interface Printer {
    static void showInfo() {
        System.out.println("Printer interface");
    }
}
```

Call it using the interface name:

```java
Printer.showInfo();
```

### D. Private method — Java 9+

```java
interface Printer {
    default void printDocument() {
        log();
        System.out.println("Printing document");
    }

    private void log() {
        System.out.println("Print operation started");
    }
}
```

Private interface methods allow code reuse within the interface. They are not inherited as public methods by implementing classes.

## 5. Multiple inheritance: why interfaces are useful

Java does not allow a class to extend two classes, but it does allow a class to implement multiple interfaces.

```java
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Duck implements Flyable, Swimmable {

    @Override
    public void fly() {
        System.out.println("Duck is flying");
    }

    @Override
    public void swim() {
        System.out.println("Duck is swimming");
    }
}
```

A `Duck` can implement both capabilities without requiring multiple class inheritance.

If multiple inherited default methods have the same signature, Java requires the class to resolve the conflict where applicable.

## 6. When should you use an abstract class vs an interface?

Use an **abstract class** when:

- Related classes share common instance state.
- A base class should provide constructors and common implementation.
- You want to define protected helper methods for subclasses.
- The classes have a meaningful shared base type.

Use an **interface** when:

- You want to define a contract or capability.
- Unrelated classes should support the same behavior.
- A class needs to implement multiple contracts.
- You want to decouple application code from concrete implementations.

### Real-world example: Spring Boot

An interface can define a service contract:

```java
public interface NotificationService {
    void send(String message);
}
```

Different implementations can provide email and SMS notifications:

```java
@Service
public class EmailNotificationService
        implements NotificationService {

    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}
```

```java
@Service
public class SmsNotificationService
        implements NotificationService {

    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}
```

The interface allows callers to depend on the abstraction rather than a specific implementation. In a real Spring application, if multiple beans implement the same interface, use an appropriate selection mechanism such as `@Qualifier` or `@Primary`.

## 7. Common interview questions

### Q1. Can an abstract class have a constructor?

Yes. Its constructor is called when a concrete subclass is instantiated, typically through `super(...)`.

### Q2. Can an abstract class have no abstract methods?

Yes. A class may be declared abstract even when all its methods have implementations. This prevents direct instantiation and can make the class serve as a base type.

### Q3. Can an interface extend another interface?

Yes. An interface can extend one or more interfaces.

```java
interface A {
    void methodA();
}

interface B {
    void methodB();
}

interface C extends A, B {
    void methodC();
}
```

### Q4. Can an abstract class implement an interface without implementing all its methods?

Yes. It can leave some or all interface methods unimplemented, provided the class is declared abstract. A concrete subclass must implement the remaining abstract methods.

### Q5. Can an interface have a constructor?

No. Interfaces cannot be instantiated and do not have constructors.

### Q6. Can an abstract class implement an interface and also extend a class?

Yes. Java permits one superclass and multiple implemented interfaces.

```java
abstract class Base {}

interface A {}

interface B {}

abstract class Example extends Base implements A, B {}
```

### Q7. What is the difference between `extends` and `implements`?

- A class uses `extends` to inherit from a class.
- An interface uses `extends` to inherit from one or more interfaces.
- A class uses `implements` to implement one or more interfaces.

## 8. Quick revision

- **Abstract class:** Shared state, constructors, common implementation, and abstract methods.
- **Interface:** Contract, capabilities, multiple-interface implementation, and loose coupling.
- A class can extend only one class but can implement multiple interfaces.
- Interfaces can have `default` and `static` methods since Java 8, and private methods since Java 9.
- Choose based on the design requirement, not simply because one construct seems more modern than the other.


---


---
title: Static Methods vs Default Methods in Java 8
parent: Core Java
nav_order: 23
---

# Static Methods vs Default Methods in Java 8

Java 8 introduced **default methods and static methods in interfaces**. These features allow interfaces to provide method implementations in addition to abstract method declarations.

The key difference is:

- **Default method:** An instance method with a default implementation that implementing classes inherit and can override.
- **Static method:** A method associated with the interface itself, called using the interface name and not inherited as an instance method by implementing classes.

## 1. Default methods

A default method is declared using the `default` keyword and has a method body inside an interface.

Default methods allow developers to add new behavior to existing interfaces while preserving compatibility with many existing implementations.

### Example

```java
interface Vehicle {

    void start();

    default void honk() {
        System.out.println("Vehicle is honking");
    }
}

class Car implements Vehicle {

    @Override
    public void start() {
        System.out.println("Car started");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();

        car.start();
        car.honk();
    }
}
```

Output:

```text
Car started
Vehicle is honking
```

The `Car` class implements `start()` but does not need to implement `honk()`, because the interface already provides a default implementation.

### Overriding a default method

An implementing class can override the default behavior.

```java
class Bike implements Vehicle {

    @Override
    public void start() {
        System.out.println("Bike started");
    }

    @Override
    public void honk() {
        System.out.println("Bike horn");
    }
}
```

Now `Bike` uses its own implementation of `honk()`.

## 2. Static methods in interfaces

A static method is declared using the `static` keyword and belongs to the interface itself.

It is useful for utility or helper operations closely related to the interface.

### Example

```java
interface Vehicle {

    static void showInfo() {
        System.out.println("Vehicle interface");
    }
}

public class Main {
    public static void main(String[] args) {
        Vehicle.showInfo();
    }
}
```

Output:

```text
Vehicle interface
```

Notice that the method is called using `Vehicle.showInfo()`, without creating an object.

### Can an implementing class override a static interface method?

No. A static interface method is not inherited as an instance method by the implementing class.

```java
interface Vehicle {
    static void showInfo() {
        System.out.println("Vehicle interface");
    }
}

class Car implements Vehicle {
    static void showInfo() {
        System.out.println("Car class");
    }
}

public class Main {
    public static void main(String[] args) {
        Vehicle.showInfo();
        Car.showInfo();
    }
}
```

Output:

```text
Vehicle interface
Car class
```

`Car.showInfo()` is a separate static method declared in `Car`. It does not override `Vehicle.showInfo()`.

## 3. Static vs default methods

| Feature | Default method | Static method |
|---|---|---|
| Introduced in interfaces | Java 8 | Java 8 |
| Keyword | `default` | `static` |
| Method type | Instance method | Static method |
| Has a method body? | Yes | Yes |
| Called using | An instance | Interface name |
| Inherited by implementing classes? | Yes | No, not as an inherited instance method |
| Can implementing classes override it? | Yes | No |
| Can access `this`? | Yes, because it is an instance method | No |
| Main purpose | Provide default instance behavior | Provide interface-related utility operations |

## 4. Can default methods access instance variables?

Interfaces cannot declare ordinary instance fields. Their fields are implicitly `public static final`.

However, a default method can access instance members of the implementing object through `this`, provided those members are accessible.

```java
interface Printable {
    default void print() {
        System.out.println("Printing...");
        printDetails();
    }

    void printDetails();
}

class Document implements Printable {
    private String title = "Java Notes";

    @Override
    public void printDetails() {
        System.out.println(title);
    }
}

public class Main {
    public static void main(String[] args) {
        new Document().print();
    }
}
```

Output:

```text
Printing...
Java Notes
```

The default method calls `printDetails()`, which is dynamically dispatched to the implementation in `Document`.

A static interface method has no `this` reference and cannot directly access instance members of an implementing object. It can access an object if that object is explicitly passed as an argument.

## 5. What if two interfaces have the same default method?

If a class implements two unrelated interfaces that provide conflicting default methods with the same signature, the class must resolve the conflict by overriding the method.

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

class Example implements A, B {

    @Override
    public void display() {
        A.super.display();
        B.super.display();
        System.out.println("Example");
    }
}

public class Main {
    public static void main(String[] args) {
        new Example().display();
    }
}
```

Output:

```text
A
B
Example
```

The syntax `A.super.display()` invokes the default implementation provided by interface `A`.

## 6. Real-world use in Java applications

### Default method: shared behavior

```java
interface NotificationService {

    void send(String message);

    default void log(String message) {
        System.out.println("Notification: " + message);
    }
}
```

Different implementations inherit `log()` but can override it when necessary.

### Static method: utility behavior

```java
interface NotificationService {

    void send(String message);

    static boolean isValid(String message) {
        return message != null && !message.isBlank();
    }
}
```

Usage:

```java
boolean valid = NotificationService.isValid("Hello");
```

The validation utility belongs to the interface and does not require an instance.

## 7. Common interview questions

### Q1. Why were default methods introduced in Java 8?

They allow interfaces to evolve by adding new methods with implementations, reducing the need to modify every existing implementation.

### Q2. Can a default method be declared in a class?

No. The `default` keyword for method implementations is used in interfaces. Classes use ordinary concrete methods.

### Q3. Can a static method in an interface be called through an implementing class?

No, not as an inherited interface method. Call it through the interface name. If the implementing class declares a static method with the same signature, that is a separate method.

### Q4. Can an interface contain both static and default methods?

Yes. An interface can declare abstract, default, and static methods together.

### Q5. Can a default method be changed to abstract in an implementing class?

An implementing class can override a default method with a concrete implementation. A concrete class cannot leave it unimplemented as an abstract method; an abstract class has more flexibility in declaring abstract methods.

### Q6. Can static methods be overloaded?

Yes. An interface can declare multiple static methods with the same name but different parameter lists.

## 8. Quick revision

- `default` methods are instance methods with implementations and can be overridden.
- `static` methods belong to the interface and are called through the interface name.
- Default methods support interface evolution and shared behavior.
- Static methods provide related utility operations.
- Conflicting default methods from multiple interfaces must be resolved by the implementing class.


---

# HashSet vs TreeSet in Java

## 1. What Is the Difference Between HashSet and TreeSet?

Both `HashSet` and `TreeSet` implement the `Set` interface and store **unique elements**, but they differ in how they store and order those elements.

- **HashSet:** Uses a hash table and does not guarantee iteration order. It generally provides O(1) average-time performance for `add()`, `remove()`, and `contains()`, assuming good hash distribution.
- **TreeSet:** Uses a self-balancing tree (typically a Red-Black tree through `TreeMap`) and maintains elements in sorted order. Basic operations generally take O(log n) time.

## 2. Comparison Table

| Feature | HashSet | TreeSet |
|---|---|---|
| Internal implementation | Hash table | Red-Black tree via `TreeMap` |
| Ordering | No guaranteed order | Sorted order |
| Duplicate elements | Not allowed | Not allowed |
| Add, remove, contains | O(1) average | O(log n) |
| Null elements | Allows one null | Generally rejects null with natural ordering |
| Sorting | Not supported automatically | Natural ordering or custom `Comparator` |
| Range queries | Not directly supported | Supports `headSet()`, `tailSet()`, `subSet()` |
| Comparable required | No | Yes, unless a suitable comparator is supplied |
| Best use case | Fast membership checks | Sorted data and range operations |

## 3. Example: HashSet

```java
import java.util.HashSet;
import java.util.Set;

public class HashSetExample {
    public static void main(String[] args) {
        Set<String> fruits = new HashSet<>();

        fruits.add("banana");
        fruits.add("apple");
        fruits.add("orange");
        fruits.add("apple"); // Duplicate ignored

        System.out.println(fruits);
        System.out.println(fruits.contains("banana"));
    }
}
```

Possible output:

```text
[orange, banana, apple]
true
```

The printed order is only an example. `HashSet` does not guarantee this order, and duplicates are ignored.

## 4. Example: TreeSet

```java
import java.util.Set;
import java.util.TreeSet;

public class TreeSetExample {
    public static void main(String[] args) {
        Set<String> fruits = new TreeSet<>();

        fruits.add("banana");
        fruits.add("apple");
        fruits.add("orange");
        fruits.add("apple"); // Duplicate ignored

        System.out.println(fruits);
        System.out.println(fruits.contains("banana"));
    }
}
```

Output:

```text
[apple, banana, orange]
true
```

`TreeSet` automatically sorts the elements according to their natural ordering. For strings, this is lexicographical ordering.

## 5. How to Use a Custom Comparator with TreeSet

You can define a different sorting order by supplying a `Comparator`.

```java
import java.util.Comparator;
import java.util.Set;
import java.util.TreeSet;

public class TreeSetComparatorExample {
    public static void main(String[] args) {
        Set<Integer> numbers =
                new TreeSet<>(Comparator.reverseOrder());

        numbers.add(10);
        numbers.add(5);
        numbers.add(20);
        numbers.add(15);

        System.out.println(numbers);
    }
}
```

Output:

```text
[20, 15, 10, 5]
```

Here, `TreeSet` maintains the numbers in descending order.

**Important:** `TreeSet` determines duplicates using its ordering comparison. If `compare(a, b)` returns `0`, the set treats the elements as equivalent, even if `a.equals(b)` is false.

## 6. When Should You Use Each?

Choose **HashSet** when:
- You need to remove duplicates.
- You need fast membership checks.
- You do not need sorted iteration.

Choose **TreeSet** when:
- You need elements sorted automatically.
- You need range-based operations such as `subSet()`, `headSet()`, or `tailSet()`.
- You need operations such as `first()`, `last()`, `floor()`, or `ceiling()`.

## 7. One-Line Interview Answer

**HashSet uses a hash table and provides O(1) average-time basic operations without guaranteeing order, whereas TreeSet uses a self-balancing tree, maintains sorted unique elements, and provides O(log n) basic operations.**


---


# ArrayList vs LinkedList in Java

## 1. What Is the Difference Between ArrayList and LinkedList?

Both `ArrayList` and `LinkedList` implement the `List` interface, but they use different internal data structures.

- **ArrayList:** Uses a resizable array. It provides fast index-based access but may need to shift elements when inserting or removing elements in the middle.
- **LinkedList:** Uses a doubly linked list. Each node stores an element and links to the previous and next nodes. It supports efficient insertion and removal at a known node, but index-based access requires traversal.

## 2. Comparison Table

| Feature | ArrayList | LinkedList |
|---|---|---|
| Internal data structure | Resizable array | Doubly linked list |
| `get(index)` | O(1) | O(n) |
| Add at end | Amortized O(1) | O(1) |
| Insert at a specific index | O(n) | O(n) to locate position; O(1) to link the node |
| Remove at a specific index | O(n) | O(n) to locate position; O(1) to unlink the node |
| Add/remove at beginning | O(n) | O(1) |
| Memory usage | Generally lower | Generally higher due to node links |
| Cache locality | Better | Generally poorer |
| Iteration | Generally faster in typical workloads | Often slower due to pointer traversal |
| Implements `Deque` | No | Yes |

## 3. Example: ArrayList vs LinkedList

```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class ListComparisonExample {
    public static void main(String[] args) {
        List<String> arrayList = new ArrayList<>();
        List<String> linkedList = new LinkedList<>();

        arrayList.add("one");
        arrayList.add("two");
        arrayList.add("three");
        arrayList.add("four");

        linkedList.add("one");
        linkedList.add("two");
        linkedList.add("three");
        linkedList.add("four");

        // Random access
        System.out.println(arrayList.get(1));  // O(1)
        System.out.println(linkedList.get(1)); // O(n)

        // Insert at index 2
        arrayList.add(2, "two-and-a-half");
        linkedList.add(2, "two-and-a-half");

        System.out.println(arrayList);
        System.out.println(linkedList);

        // Iterate over elements
        for (String element : arrayList) {
            System.out.println(element);
        }

        for (String element : linkedList) {
            System.out.println(element);
        }
    }
}
```

Output:

```text
two
two
[one, two, two-and-a-half, three, four]
[one, two, two-and-a-half, three, four]
one
two
two-and-a-half
three
four
one
two
two-and-a-half
three
four
```

Both lists produce the same logical result, but their internal operations differ.

## 4. Is Insertion Faster in LinkedList?

Not always. This is a common interview trap.

Consider:

```java
linkedList.add(500, "new");
```

`LinkedList` must first traverse the list to locate the insertion position, which takes O(n) time in the general case. Linking the new node then takes O(1).

By comparison:

```java
arrayList.add(500, "new");
```

`ArrayList` may need to shift subsequent elements, making the operation O(n).

Both operations are O(n) overall in the general case. However, `LinkedList` can insert or remove in O(1) once the required node or iterator position is already known.

## 5. When Should You Use Each?

Choose **ArrayList** when:
- You frequently access elements by index.
- You iterate over the list frequently.
- You mostly append elements and occasionally insert or remove them.
- You want lower per-element memory overhead and good cache locality.

Choose **LinkedList** when:
- You frequently add or remove elements at the beginning or end.
- You need `Deque` operations and have a reason to use a linked structure.
- You already have an iterator positioned where insertion or removal is needed.

For typical queue or stack operations, also consider `ArrayDeque`, which is often more efficient than `LinkedList`.

## 6. One-Line Interview Answer

**ArrayList uses a resizable array and provides O(1) index-based access, whereas LinkedList uses a doubly linked list and provides O(1) insertion or removal at a known node, but O(n) index-based access.**


---

# Set vs List in Java

## 1. What Is the Difference Between Set and List?

Both `Set` and `List` are interfaces in the Java Collections Framework. They represent collections of elements but have different rules.

- **List:** Allows duplicate elements and preserves positional order. Elements can be accessed using an index.
- **Set:** Prevents duplicate elements according to its equality or ordering rules. It does not provide index-based access, and its ordering depends on the implementation.

## 2. Comparison Table

| Feature | List | Set |
|---|---|---|
| Duplicate elements | Allowed | Not allowed |
| Ordering | Preserves positional order | Depends on implementation |
| Index-based access | Supported | Not supported |
| Common implementations | `ArrayList`, `LinkedList`, `Vector` | `HashSet`, `LinkedHashSet`, `TreeSet` |
| Null elements | Generally allowed | Depends on implementation |
| Access by position | `get(index)` | Iterate or use implementation-specific operations |
| Typical use case | Ordered data with possible duplicates | Unique elements |

## 3. Example: List vs Set

```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class ListVsSetExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();

        list.add("one");
        list.add("two");
        list.add("three");
        list.add("two");

        Set<String> set = new HashSet<>();

        set.add("one");
        set.add("two");
        set.add("three");
        set.add("two");

        System.out.println("List: " + list);
        System.out.println("Set: " + set);

        System.out.println("List element at index 1: "
                + list.get(1));
    }
}
```

Possible output:

```text
List: [one, two, three, two]
Set: [one, two, three]
List element at index 1: two
```

The `List` retains both occurrences of `"two"`, while the `Set` stores only one. The displayed `HashSet` order is illustrative; it is not guaranteed.

## 4. Types of List and Set

### List implementations

- `ArrayList`: Fast index-based access and general-purpose list operations.
- `LinkedList`: Doubly linked list that also implements `Deque`.
- `Vector`: Synchronized, legacy list implementation.

### Set implementations

- `HashSet`: Unique elements without guaranteed iteration order.
- `LinkedHashSet`: Unique elements in insertion order.
- `TreeSet`: Unique elements in sorted order.

**Important:** A `Set` does not always mean unordered. `LinkedHashSet` preserves insertion order, while `TreeSet` maintains sorted order.

## 5. When Should You Use List or Set?

Choose **List** when:
- Duplicate values are valid.
- You need index-based access.
- The order of elements matters.

Choose **Set** when:
- You need to prevent duplicate elements.
- You need efficient membership checks with `HashSet`.
- You need insertion order or sorted order using `LinkedHashSet` or `TreeSet`.

## 6. One-Line Interview Answer

**List allows duplicates and supports index-based access while preserving positional order; Set prevents duplicates and does not provide index-based access, with iteration order depending on its implementation.**


---

# HashSet vs HashMap in Java

## 1. What Is the Difference Between HashSet and HashMap?

Both `HashSet` and `HashMap` belong to the Java Collections Framework and use hashing internally, but they serve different purposes.

- **HashSet:** Stores unique elements. Use it when you need to maintain a collection of distinct values.
- **HashMap:** Stores key-value pairs. Use it when you need to associate a key with a value and retrieve that value using the key.

Internally, `HashSet` uses a `HashMap` to store its elements as keys, with a shared dummy object as the value.

## 2. Comparison Table

| Feature | HashSet | HashMap |
|---|---|---|
| Data stored | Individual elements | Key-value pairs |
| Duplicate elements/keys | Duplicate elements are rejected | Duplicate keys are not stored separately |
| Duplicate values | Not applicable as a separate value field | Allowed |
| Retrieval | `contains(element)` | `get(key)` |
| Insertion | `add(element)` | `put(key, value)` |
| Internal implementation | Backed by `HashMap` | Hash table |
| Ordering | No guaranteed iteration order | No guaranteed iteration order |
| Null support | Allows one null element | Allows one null key and multiple null values |
| Equality and hashing | Uses `hashCode()` and `equals()` | Uses `hashCode()` and `equals()` for keys |

## 3. Example: HashSet vs HashMap

```java id="r7f9xk"
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

public class HashSetVsHashMapExample {
    public static void main(String[] args) {

        // HashMap: stores key-value pairs
        Map<String, Integer> hashMap = new HashMap<>();

        hashMap.put("one", 1);
        hashMap.put("two", 2);
        hashMap.put("three", 3);

        // Duplicate key replaces the existing value
        hashMap.put("two", 22);

        System.out.println(hashMap.get("two"));

        // HashSet: stores unique elements
        Set<Integer> hashSet = new HashSet<>();

        hashSet.add(1);
        hashSet.add(2);
        hashSet.add(3);
        hashSet.add(2); // Duplicate ignored

        System.out.println(hashSet.contains(2));
        System.out.println(hashSet.size());
    }
}
```

Output:

```text id="k2fz5g"
22
true
3
```

Explanation:

- `hashMap.put("two", 22)` replaces the previous value associated with `"two"`.
- `hashSet.add(2)` does not add a second copy of `2`.
- `hashMap.get("two")` retrieves the value using its key.
- `hashSet.contains(2)` checks whether the element exists.

## 4. How Does HashSet Work Internally?

Conceptually, `HashSet` uses a `HashMap` like this:

```java id="n9c1hf"
private transient HashMap<Object, Object> map;

private static final Object PRESENT = new Object();

public boolean add(Object element) {
    return map.put(element, PRESENT) == null;
}
```

This is a simplified illustration, not the complete JDK implementation.

When you add an element to a `HashSet`, the element becomes a key in the internal map, and the dummy object `PRESENT` is used as its value. Since map keys are unique, duplicate set elements are not added.

## 5. When Should You Use Each?

Choose **HashSet** when:
- You need unique elements.
- You need to check whether an element exists.
- You do not need to associate separate values with keys.

Choose **HashMap** when:
- You need key-value associations.
- You need to retrieve a value using a key.
- Multiple keys may map to the same value.

## 6. One-Line Interview Answer

**HashSet stores unique elements and is internally backed by a HashMap, whereas HashMap stores key-value pairs, enforces unique keys, and allows duplicate values.**


---

# LinkedHashMap vs PriorityQueue in Java

## 1. What Is the Difference Between LinkedHashMap and PriorityQueue?

`LinkedHashMap` and `PriorityQueue` serve different purposes in the Java Collections Framework.

- **LinkedHashMap:** Stores key-value pairs and preserves insertion order by default. It is useful when you need to retrieve values using keys while maintaining predictable iteration order.
- **PriorityQueue:** Stores individual elements and organizes them according to their natural ordering or a custom `Comparator`. It is useful when you need to process the highest- or lowest-priority element first.

Internally, `LinkedHashMap` combines a hash table with a doubly linked list, while `PriorityQueue` typically uses a binary heap.

## 2. Comparison Table

| Feature | LinkedHashMap | PriorityQueue |
|---|---|---|
| Interface | Implements `Map` | Implements `Queue` |
| Data stored | Key-value pairs | Individual elements |
| Internal structure | Hash table + doubly linked list | Binary heap |
| Ordering | Insertion order by default | Priority order |
| Duplicate keys/elements | Duplicate keys replace existing values | Duplicate elements are allowed |
| Access | Retrieve values using keys | Examine or remove the head element |
| Insert | O(1) average | O(log n) |
| Retrieve | `get(key)` is O(1) average | `peek()` is O(1) |
| Remove | O(1) average by key | `poll()` is O(log n) |
| Null support | Allows one null key and null values | Does not allow null elements |
| Best use case | Ordered key-value storage | Priority-based processing |

## 3. Example: LinkedHashMap

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LinkedHashMapExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new LinkedHashMap<>();

        map.put("one", 1);
        map.put("two", 2);
        map.put("three", 3);

        System.out.println(map);
        System.out.println(map.get("two"));
    }
}
```

Output:

```text
{one=1, two=2, three=3}
2
```

The map preserves insertion order and allows retrieving a value using its key.

## 4. Example: PriorityQueue

```java
import java.util.PriorityQueue;
import java.util.Queue;

public class PriorityQueueExample {
    public static void main(String[] args) {
        Queue<Integer> queue = new PriorityQueue<>();

        queue.add(3);
        queue.add(1);
        queue.add(2);

        System.out.println(queue.peek());

        while (!queue.isEmpty()) {
            System.out.println(queue.poll());
        }
    }
}
```

Output:

```text
1
1
2
3
```

By default, `PriorityQueue` uses natural ordering, so the smallest integer has the highest priority.

**Important:** Printing a `PriorityQueue` does not guarantee that all its elements will appear in sorted order. Only the head element is guaranteed to be the least element under natural ordering. Repeatedly calling `poll()` retrieves elements in priority order.

## 5. How Does PriorityQueue Work Internally?

A `PriorityQueue` typically uses a binary heap, commonly a min-heap by default.

```text
          1
         / \
        3   2
```

The smallest element is at the root. When an element is inserted or removed, the heap reorganizes itself to preserve the heap property.

A heap is not a fully sorted collection; it guarantees the priority relationship between the root and other elements.

## 6. When Should You Use Each?

Choose **LinkedHashMap** when:
- You need key-value associations.
- You want to preserve insertion order during iteration.
- You need efficient lookup by key.

Choose **PriorityQueue** when:
- You need to process elements by priority.
- You need efficient access to the smallest element or the element at the head according to a comparator.
- You are implementing task scheduling, top-K algorithms, or graph algorithms such as Dijkstra's algorithm.

## 7. One-Line Interview Answer

**LinkedHashMap stores key-value pairs and preserves insertion order, whereas PriorityQueue stores elements in a heap and retrieves them according to priority rather than insertion order.**


---

