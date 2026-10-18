---
title: Language Fundamentals
parent: Core Java
nav_order: 3
---

# Language Fundamentals
{: .no_toc }

Strings, equals/hashCode, immutability and other basics.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---


---

# Java Core Interview Questions

## 1. What Does the `static` Keyword Signify in Java?

The `static` keyword indicates that a variable, method, or initialization block belongs to the **class rather than to individual objects**.

Java commonly uses `static` for class variables, utility methods, and one-time class initialization.

### 1.1 Static Variables

A static variable is shared among all instances of a class. Only one class-level copy exists.

```java
class Employee {
    static String company = "Achhe Code";
    String name;

    Employee(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        Employee e1 = new Employee("Danish");
        Employee e2 = new Employee("Rahul");

        System.out.println(e1.company);
        System.out.println(e2.company);

        Employee.company = "New Company";

        System.out.println(e1.company);
        System.out.println(e2.company);
    }
}
```

**Output:**

```text
Achhe Code
Achhe Code
New Company
New Company
```

Both objects share the same `company` variable. Accessing a static variable through the class name, as in `Employee.company`, is preferred.

### 1.2 Static Methods

A static method belongs to the class and can be called without creating an object.

```java
class Calculator {
    public static int add(int a, int b) {
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(Calculator.add(10, 20));
    }
}
```

**Output:**

```text
30
```

Important rules:

- A static method can directly access static members.
- It cannot directly access instance variables or instance methods because it has no `this` reference.
- Static methods cannot be overridden, but they can be hidden by a static method with the same signature in a subclass.

### 1.3 Static Blocks

A static block executes when the JVM initializes the class, before the class is actively used. It is commonly used for static initialization.

```java
class Example {
    static int value;

    static {
        value = 100;
        System.out.println("Static block executed");
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(Example.value);
    }
}
```

**Output:**

```text
Static block executed
100
```

The block runs once per class initialization, not once for every object.

---

## 2. What Is the Difference Between Static Variables and Instance Variables?

| Feature | Static Variable | Instance Variable |
|---|---|---|
| Belongs to | Class | Object |
| Number of copies | One per class initialization and defining class | One per object |
| Initialization | Receives a default value, or an explicit initialization value | Receives a default value, or an explicit initialization value |
| Access | Preferably through the class name | Through an object |
| Lifetime | Associated with the loaded class | Associated with the object |
| Typical use | Shared configuration, counters, constants | Object-specific state |

### Example

```java
class Employee {
    static String company = "Achhe Code";
    String name;

    Employee(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        Employee e1 = new Employee("Danish");
        Employee e2 = new Employee("Rahul");

        e1.name = "Amit";
        Employee.company = "Tech Company";

        System.out.println(e1.name);
        System.out.println(e2.name);
        System.out.println(e1.company);
        System.out.println(e2.company);
    }
}
```

**Output:**

```text
Amit
Rahul
Tech Company
Tech Company
```

The `name` variable is unique to each employee, while `company` is shared.

**Interview takeaway:** Use instance variables for object-specific data and static variables for class-level data shared by instances.

---

## 3. What Is a Covariant Return Type in Java?

A **covariant return type** allows an overriding method in a subclass to return a more specific reference type than the method declared in its parent class.

For example, if class `Dog` extends class `Animal`, an overriding method can return `Dog` instead of `Animal`.

### Example

```java
class Animal {
    Animal getAnimal() {
        return new Animal();
    }
}

class Dog extends Animal {
    @Override
    Dog getAnimal() {
        return new Dog();
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog();
        Dog result = dog.getAnimal();

        System.out.println(result instanceof Dog);
    }
}
```

**Output:**

```text
true
```

The parent method returns `Animal`, while the overriding method returns `Dog`. This is valid because `Dog` is a subtype of `Animal`.

### Important Rules

- Covariant return types apply to method overriding, not ordinary method overloading.
- The overriding method's return type must be compatible with the parent's return type; for reference types, it may be a subtype.
- Primitive return types cannot be changed covariantly.

**Interview takeaway:** Covariant return types allow subclasses to provide more specific return types while preserving the overriding relationship.

---

## 4. Can a Java Interface Have No Methods?

Yes. An interface without any declared methods is commonly called a **marker interface**. It identifies a class as having a particular property or as participating in a particular framework or runtime behavior.

### Example

```java
import java.io.Serializable;

class Employee implements Serializable {
    private String name;

    Employee(String name) {
        this.name = name;
    }
}
```

`Serializable` is a standard Java marker interface. Implementing it indicates that an object can participate in Java's serialization mechanism, subject to serialization requirements.

We can also define our own marker interface:

```java
interface MyMarkerInterface {
}

class MyClass implements MyMarkerInterface {
}
```

`MyClass` does not need to implement any methods because `MyMarkerInterface` declares none.

### Why Use Marker Interfaces?

- To mark classes for special handling.
- To support framework or runtime checks.
- To identify classes with a particular capability or semantic property.

**Interview takeaway:** A marker interface communicates metadata through type membership without requiring the implementing class to implement methods. Annotations are another way to express metadata, but they serve a different mechanism.

---

## 5. What Are the Exception Rules for Method Overriding?

When overriding a method in Java, the overriding method must follow the checked-exception restrictions of the parent method.

### Rules

1. It can declare the same checked exception as the parent method.
2. It can declare a narrower checked exception.
3. It can declare no checked exception.
4. It cannot declare a broader checked exception than the parent method permits.
5. It can throw unchecked exceptions, such as `RuntimeException`, even if the parent method does not declare them.

### Example

```java
import java.io.IOException;
import java.io.FileNotFoundException;
import java.sql.SQLException;

class Parent {
    void readFile() throws IOException {
        System.out.println("Reading file");
    }
}

class Child extends Parent {

    @Override
    void readFile() throws FileNotFoundException {
        System.out.println("Reading file in Child");
    }
}
```

This is valid because `FileNotFoundException` is a subclass of `IOException`.

### Invalid Example

```java
class Parent {
    void readFile() throws IOException {
    }
}

class Child extends Parent {

    // Compilation error:
    // Exception is broader than IOException.
    @Override
    void readFile() throws Exception {
    }
}
```

The compiler rejects the overriding method because `Exception` is broader than `IOException`.

### Unchecked Exceptions

```java
class Parent {
    void process() {
    }
}

class Child extends Parent {

    @Override
    void process() throws RuntimeException {
        throw new RuntimeException("Processing failed");
    }
}
```

This is valid because `RuntimeException` is unchecked.

**Interview takeaway:** An overriding method cannot broaden the checked exceptions declared by the parent method, but it may narrow them or omit them entirely.

---

## 6. What Are the Different Types of Access Modifiers in Java?

Java has four access levels: `public`, `protected`, package-private (the default when no modifier is specified), and `private`.

### Access Modifier Comparison

| Access Modifier | Same Class | Same Package | Subclass in Another Package | Unrelated Class in Another Package |
|---|---|---|---|---|
| `public` | Yes | Yes | Yes | Yes |
| `protected` | Yes | Yes | Yes, subject to protected-access rules | No |
| Default (package-private) | Yes | Yes | No | No |
| `private` | Yes | No | No | No |

### Example

```java
class Example {
    public int publicVar = 1;
    protected int protectedVar = 2;
    int defaultVar = 3;
    private int privateVar = 4;

    public void publicMethod() {
    }

    protected void protectedMethod() {
    }

    void defaultMethod() {
    }

    private void privateMethod() {
    }
}
```

- `public`: accessible wherever the declaring type and member are accessible.
- `protected`: accessible within the same package and through inheritance, subject to Java's cross-package protected-access rules.
- Default/package-private: accessible within the same package only.
- `private`: accessible within the declaring top-level class, including its nested classes under Java's access rules.

**Important:** Top-level classes can be `public` or package-private; they cannot be declared `private` or `protected`.

---

## 7. What Is the Difference Between `private` and `protected`?

The primary difference is **where the member can be accessed**.

| Feature | `private` | `protected` |
|---|---|---|
| Within the declaring class | Yes | Yes |
| Within the same package | No, outside the declaring class | Yes |
| In a subclass in another package | Not directly accessible | Accessible subject to protected-access rules |
| Direct access by unrelated classes | No | Only within the same package |
| Overriding | Private methods are not inherited or overridden in the usual sense | Eligible instance methods can be overridden |

### Example

```java
class Parent {
    private int privateValue = 10;
    protected int protectedValue = 20;

    public void display() {
        System.out.println(privateValue);
        System.out.println(protectedValue);
    }
}

class Child extends Parent {
    public void show() {
        // System.out.println(privateValue); // Compilation error
        System.out.println(protectedValue);  // Valid
    }
}

public class Main {
    public static void main(String[] args) {
        Child child = new Child();

        child.display();
        child.show();
    }
}
```

**Output:**

```text
10
20
20
```

The child class cannot directly access `privateValue`, but it can access `protectedValue` because both classes are in the same package in this example.

A private method in a parent class cannot be overridden because it is not inherited as an accessible method by the subclass. A subclass may declare a method with the same signature, but that is a separate method, not an override.

**Interview takeaway:** Use `private` to encapsulate implementation details and `protected` when subclasses or classes in the same package need access.

---

## Quick Revision

| Question | Key Answer |
|---|---|
| What does `static` mean? | The member belongs to the class rather than each object. |
| Static vs. instance variable? | Shared class-level state vs. object-specific state. |
| What is a covariant return type? | An overriding method returns a more specific reference type. |
| Can an interface have no methods? | Yes; it can be a marker interface. |
| What are overriding exception rules? | Checked exceptions cannot be broadened. |
| What are Java's access levels? | `public`, `protected`, package-private, and `private`. |
| `private` vs. `protected`? | Private is limited to the declaring class; protected also supports package access and qualifying subclass access. |


---

# How Java's `equals()` and `hashCode()` Methods Work Together

The `equals()` and `hashCode()` methods are inherited from Java's `Object` class. They work together to determine logical equality and organize objects in hash-based collections such as `HashMap`, `HashSet`, and `Hashtable`.

- `equals()` determines whether two objects should be considered logically equal.
- `hashCode()` returns an integer used by hash-based collections to help locate objects efficiently.

**Key principle:** If two objects are equal according to `equals()`, they must return the same hash code. The reverse is not necessarily true.

## 1. How Does `equals()` Work?

By default, `Object.equals()` compares object references. Two different objects are not equal by this implementation, even if their fields contain identical values.

### Example: Default `equals()`

```java
class Example {
}

public class Main {
    public static void main(String[] args) {
        Example obj1 = new Example();
        Example obj2 = new Example();

        System.out.println(obj1.equals(obj2));
        System.out.println(obj1.equals(obj1));
    }
}
```

**Output:**

```text
false
true
```

`obj1` and `obj2` refer to separate objects. Since `equals()` has not been overridden, their logical equality is not based on their contents.

### Overriding `equals()`

Suppose two users should be considered equal when they have the same username.

```java
class User {
    private final String username;

    User(String username) {
        this.username = username;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        User user = (User) obj;
        return java.util.Objects.equals(username, user.username);
    }
}
```

Now, two separately created `User` objects with the same username are considered equal.

## 2. How Does `hashCode()` Work?

The `hashCode()` method returns an integer representing an object for hashing purposes. Hash-based collections use it to determine the bucket in which an object or key should be stored.

The default implementation provided by `Object` does not guarantee that the hash code represents a memory address. Its exact behavior is implementation-dependent.

### Overriding `hashCode()`

```java
import java.util.Objects;

class User {
    private final String username;

    User(String username) {
        this.username = username;
    }

    @Override
    public int hashCode() {
        return Objects.hash(username);
    }
}
```

`Objects.hash(username)` calculates a hash code based on the supplied value. For equal usernames, it produces the same hash code.

**Important:** A hash code is not a unique identifier. Different objects can have the same hash code.

## 3. The Contract Between `equals()` and `hashCode()`

Java defines several important rules for these methods.

1. **Equal objects must have equal hash codes.** If `a.equals(b)` is `true`, then `a.hashCode()` must equal `b.hashCode()`.
2. **Equal hash codes do not imply equal objects.** Two unequal objects can have the same hash code. This is called a *hash collision*.
3. **Consistent equality:** Repeated calls to `equals()` should return the same result as long as the relevant information used in the comparison remains unchanged.
4. **Consistent hash codes:** Repeated calls to `hashCode()` must return the same value while the information used by the method remains unchanged.
5. **Override both methods together.** If you override `equals()`, you should generally override `hashCode()` to preserve the contract.

## 4. What Happens If We Override `equals()` but Not `hashCode()`?

Consider the following class:

```java
class User {
    private final String username;

    User(String username) {
        this.username = username;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        User user = (User) obj;
        return java.util.Objects.equals(username, user.username);
    }

    // hashCode() is not overridden
}
```

Now consider using it in a `HashSet`:

```java
import java.util.HashSet;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        Set<User> users = new HashSet<>();

        User user1 = new User("Alex");
        User user2 = new User("Alex");

        users.add(user1);
        users.add(user2);

        System.out.println(user1.equals(user2));
        System.out.println(users.size());
    }
}
```

**Expected output on typical JVM implementations:**

```text
true
2
```

The two users are equal according to `equals()`, but the inherited `hashCode()` may give them different hash codes. A `HashSet` can therefore place them in different buckets and retain both.

The result demonstrates a broken `equals()`/`hashCode()` contract.

## 5. Correct Implementation of `equals()` and `hashCode()`

Here is the corrected version:

```java
import java.util.Objects;

class User {
    private final String username;

    User(String username) {
        this.username = username;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        User user = (User) obj;
        return Objects.equals(username, user.username);
    }

    @Override
    public int hashCode() {
        return Objects.hash(username);
    }
}

public class Main {
    public static void main(String[] args) {
        User user1 = new User("Alex");
        User user2 = new User("Alex");

        java.util.Set<User> users = new java.util.HashSet<>();
        users.add(user1);
        users.add(user2);

        System.out.println(user1.equals(user2));
        System.out.println(user1.hashCode() == user2.hashCode());
        System.out.println(users.size());
    }
}
```

**Output:**

```text
true
true
1
```

Because both objects have the same username, `equals()` considers them equal and `hashCode()` returns the same value. The `HashSet` recognizes the duplicate and stores only one entry.

## 6. How Do Hash-Based Collections Use These Methods?

Collections such as `HashMap`, `HashSet`, and `Hashtable` use hashing to locate objects efficiently.

The general process is:

1. Call `hashCode()` to calculate a hash value.
2. Use the hash value to identify a bucket.
3. If necessary, compare keys or elements using `equals()`.
4. If an equal key already exists in a `HashMap`, the associated value is replaced. If an equal element already exists in a `HashSet`, the duplicate is not added.

This avoids scanning every element in the collection during typical lookups.

### How `HashMap` Uses Both Methods

Consider a product inventory where products are identified by their ID and name.

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

class Product {
    private final int id;
    private final String name;

    Product(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        Product product = (Product) obj;
        return id == product.id
                && Objects.equals(name, product.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}

public class Main {
    public static void main(String[] args) {
        Map<Product, Integer> inventory = new HashMap<>();

        Product p1 = new Product(101, "Laptop");
        Product p2 = new Product(101, "Laptop");

        inventory.put(p1, 50);

        System.out.println(inventory.get(p2));
    }
}
```

**Output:**

```text
50
```

Although `p1` and `p2` are different object instances, they are equal according to `equals()` and have the same hash code. Therefore, `inventory.get(p2)` finds the entry inserted using `p1`.

## 7. What Is a Hash Collision?

A hash collision occurs when two unequal objects produce the same hash code.

For example, consider a class whose hash code is the length of its name:

```java
class Item {
    private final String name;

    Item(String name) {
        this.name = name;
    }

    @Override
    public int hashCode() {
        return name.length();
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        Item item = (Item) obj;
        return java.util.Objects.equals(name, item.name);
    }
}
```

Both `"Table"` and `"Chair"` have five characters, so their corresponding `Item` objects have the same hash code. However, their names differ, so `equals()` returns `false`.

```java
import java.util.HashSet;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        Set<Item> items = new HashSet<>();

        items.add(new Item("Table"));
        items.add(new Item("Chair"));

        System.out.println(items.size());
    }
}
```

**Output:**

```text
2
```

A collision does not cause unequal objects to be treated as equal. The collection uses `equals()` to distinguish them.

A better hash function aims to distribute values across buckets more evenly, reducing collisions and helping maintain performance.

## 8. Why Can Objects Become Difficult to Find in a `HashSet` or `HashMap`?

If a field used by `equals()` or `hashCode()` changes after an object is added to a hash-based collection, the object may become difficult to find.

### Example

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

class Employee {
    String name;

    Employee(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        Employee employee = (Employee) obj;
        return Objects.equals(name, employee.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}

public class Main {
    public static void main(String[] args) {
        Set<Employee> employees = new HashSet<>();

        Employee emp = new Employee("John");
        employees.add(emp);

        emp.name = "Mike";

        System.out.println(employees.contains(emp));
    }
}
```

**Output for typical hash-based collection behavior:**

```text
false
```

The employee was inserted using the hash code calculated for `"John"`. After the name changes to `"Mike"`, the lookup uses a different hash code and may search a different bucket.

This result is not guaranteed for every possible hash implementation, but it illustrates why changing equality-related fields while an object is stored in a hash-based collection is unsafe.

**Best practice:** Use immutable objects or immutable equality-related fields for map keys and set elements.

## 9. How Does `HashSet` Use `equals()` and `hashCode()`?

A `HashSet` uses these methods together to enforce uniqueness according to logical equality.

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

class Book {
    private final String title;
    private final String author;

    Book(String title, String author) {
        this.title = title;
        this.author = author;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        Book book = (Book) obj;
        return Objects.equals(title, book.title)
                && Objects.equals(author, book.author);
    }

    @Override
    public int hashCode() {
        return Objects.hash(title, author);
    }
}

public class Main {
    public static void main(String[] args) {
        Set<Book> library = new HashSet<>();

        library.add(new Book("Java Programming", "John Smith"));
        library.add(new Book("Java Programming", "John Smith"));

        System.out.println(library.size());
    }
}
```

**Output:**

```text
1
```

The two book instances are logically equal, so the set retains only one.

## 10. Common Interview Questions

**Q1. Can two unequal objects have the same hash code?**

Yes. This is a hash collision. The `equals()` method determines whether the objects are logically equal.

**Q2. Can two equal objects have different hash codes?**

No. That violates the `equals()` and `hashCode()` contract and can cause incorrect behavior in hash-based collections.

**Q3. What happens if we override only `hashCode()`?**

Unequal objects can still be considered different because `equals()` retains its default reference-based behavior. A hash code alone does not establish logical equality.

**Q4. Why should we override both methods together?**

Because hash-based collections use `hashCode()` to locate a bucket and `equals()` to determine equality within that search. Both methods must follow a consistent contract.

**Q5. Does a hash collision mean `HashSet` removes an element?**

No. A collision only means the objects share a hash code. If `equals()` returns `false`, both objects can be stored.

**Q6. What is the difference between `==` and `equals()`?**

- `==` compares primitive values or, for reference types, whether two references refer to the same object.
- `equals()` compares logical equality as defined by the class. Its default implementation in `Object` compares references.

## Quick Revision

| Concept | Key Point |
|---|---|
| `equals()` | Determines logical equality |
| `hashCode()` | Returns an integer used in hashing |
| Equal objects | Must have equal hash codes |
| Equal hash codes | Do not guarantee equal objects |
| Hash collision | Unequal objects share a hash code |
| `HashMap` | Uses hashing and equality to locate keys |
| `HashSet` | Uses hashing and equality to prevent duplicate elements |
| Mutable keys | Can become difficult to retrieve after equality-related fields change |

**Interview takeaway:** Always implement `equals()` and `hashCode()` consistently when defining logical equality for objects used as keys in `HashMap` or elements in `HashSet`. This preserves correctness and allows hash-based collections to work as intended.


---

# Comparable vs Comparator in Java

Both `Comparable` and `Comparator` are Java interfaces used to define the ordering of objects. They are commonly used when sorting lists, arrays, and other collections.

The main difference is:

- **Comparable:** Defines the natural ordering of objects inside the class itself.
- **Comparator:** Defines an external or alternative ordering strategy, allowing the same objects to be sorted in multiple ways.

## 1. Comparable interface

`Comparable<T>` belongs to the `java.lang` package. A class implements it to define its natural ordering using the `compareTo()` method.

### Syntax

```java
public interface Comparable<T> {
    int compareTo(T other);
}
```

The `compareTo()` method returns:

| Return value | Meaning |
|---|---|
| Negative integer | Current object comes before the other object |
| Zero | Objects are equal according to this ordering |
| Positive integer | Current object comes after the other object |

### Example: Sort employees by age

```java
class Employee implements Comparable<Employee> {

    private final String name;
    private final int age;

    public Employee(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.age, other.age);
    }

    @Override
    public String toString() {
        return name + " - " + age;
    }
}
```

Usage:

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ComparableExample {
    public static void main(String[] args) {
        List<Employee> employees = new ArrayList<>();

        employees.add(new Employee("Danish", 30));
        employees.add(new Employee("Ali", 25));
        employees.add(new Employee("Sara", 28));

        Collections.sort(employees);

        employees.forEach(System.out::println);
    }
}
```

Output:

```text
Ali - 25
Sara - 28
Danish - 30
```

Because `Employee` implements `Comparable<Employee>`, `Collections.sort(employees)` uses the natural ordering defined by `compareTo()`.

**Key point:** The natural ordering is defined by the class, so the class must be modified to implement `Comparable` if it does not already do so.

## 2. Comparator interface

`Comparator<T>` belongs to the `java.util` package. It defines an external comparison strategy using the `compare()` method.

### Syntax

```java
public interface Comparator<T> {
    int compare(T first, T second);
}
```

Like `compareTo()`, the `compare()` method returns a negative integer, zero, or a positive integer to indicate the relative ordering of two objects.

### Example: Sort employees by name

Using the same `Employee` class:

```java
import java.util.Comparator;

class EmployeeNameComparator
        implements Comparator<Employee> {

    @Override
    public int compare(Employee first, Employee second) {
        return first.getName()
                .compareTo(second.getName());
    }
}
```

Usage:

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ComparatorExample {
    public static void main(String[] args) {
        List<Employee> employees = new ArrayList<>();

        employees.add(new Employee("Danish", 30));
        employees.add(new Employee("Ali", 25));
        employees.add(new Employee("Sara", 28));

        employees.sort(new EmployeeNameComparator());

        employees.forEach(System.out::println);
    }
}
```

Output:

```text
Ali - 25
Danish - 30
Sara - 28
```

The original `Employee` class defines its natural ordering by age, but the comparator sorts the same objects by name.

## 3. Comparator with Java 8 lambda expressions

In modern Java applications, comparators are commonly created using lambda expressions and factory methods rather than separate comparator classes.

### Sort by name

```java
employees.sort(
    (e1, e2) -> e1.getName().compareTo(e2.getName())
);
```

### Sort by age

```java
employees.sort(
    Comparator.comparingInt(Employee::getAge)
);
```

### Sort by age descending

```java
employees.sort(
    Comparator.comparingInt(Employee::getAge).reversed()
);
```

### Sort by name, then by age

```java
employees.sort(
    Comparator.comparing(Employee::getName)
              .thenComparingInt(Employee::getAge)
);
```

### Sort strings case-insensitively

```java
List<String> names = new ArrayList<>(
    List.of("danish", "Ali", "sara")
);

names.sort(String.CASE_INSENSITIVE_ORDER);

System.out.println(names);
```

Output:

```text
[Ali, danish, sara]
```

These approaches allow multiple sorting strategies without changing the original class.

## 4. Comparable vs Comparator

| Feature | Comparable | Comparator |
|---|---|---|
| Package | `java.lang` | `java.util` |
| Method | `compareTo(T other)` | `compare(T first, T second)` |
| Comparison logic | Inside the class | Separate from the class |
| Ordering | Usually natural/default ordering | Custom or alternative ordering |
| Number of strategies | Typically one natural ordering | Multiple strategies |
| Requires modifying the class? | Yes, if it does not implement `Comparable` | No |
| Common sorting API | `Collections.sort(list)` | `list.sort(comparator)` |
| Java 8 functional interface? | No | Yes |

Both interfaces can be used by sorting APIs, including `List.sort()` and the relevant `Arrays.sort()` overloads.

## 5. Important interview concepts

### Q1. Can a class implement both Comparable and Comparator?

Yes, but they serve different roles. A class may implement `Comparable` for its natural ordering and also implement `Comparator` to provide a particular comparison strategy. However, it is usually clearer to keep alternative sorting logic in separate comparator objects.

### Q2. What happens if Comparable is not implemented?

Calling a natural-order sorting method on non-null elements that do not implement `Comparable` generally results in a `ClassCastException`. Supply a suitable `Comparator` instead.

### Q3. What is the difference between compareTo() and compare()?

- `compareTo()` compares the current object with another object.
- `compare()` compares two supplied objects.

```java
e1.compareTo(e2);       // Comparable
comparator.compare(e1, e2); // Comparator
```

### Q4. How do you sort in descending order?

For a comparator:

```java
employees.sort(
    Comparator.comparingInt(Employee::getAge).reversed()
);
```

For natural ordering, one option is:

```java
Collections.sort(employees);
Collections.reverse(employees);
```

The second approach sorts first in natural order and then reverses the list.

### Q5. What if compare() returns zero for two different objects?

It means the comparator considers those objects equal for sorting purposes. It does not necessarily mean their `equals()` method returns `true`.

A well-designed comparator should ideally be consistent with `equals()` when used with sorted sets or maps, because those collections may treat elements or keys comparing as zero as equivalent.

### Q6. Why should we use Integer.compare() instead of subtraction?

Avoid this:

```java
return this.age - other.age;
```

Subtraction can overflow for integer values at extreme limits and produce an incorrect comparison result.

Prefer:

```java
return Integer.compare(this.age, other.age);
```

### Q7. What is a chained comparator?

A chained comparator applies multiple sorting criteria in sequence:

```java
Comparator<Employee> comparator =
    Comparator.comparingInt(Employee::getAge)
              .thenComparing(Employee::getName);
```

Employees are sorted by age first. When two employees have the same age, their names determine the order.

## 6. Quick revision

- **Comparable:** One natural ordering, defined using `compareTo()`.
- **Comparator:** External comparison logic, defined using `compare()`.
- Use `Collections.sort(list)` or `list.sort(null)` for natural ordering.
- Use `list.sort(comparator)` for custom ordering.
- Prefer `Comparator.comparing()`, `comparingInt()`, `thenComparing()`, and `reversed()` for concise sorting logic.
- Use `Integer.compare()` rather than subtraction when comparing integer fields.


---

# Static Keyword in Java

In Java, the `static` keyword indicates that a member belongs to the **class itself rather than to an individual object**.

It can be used with variables, methods, initialization blocks, and nested classes.

## 1. Static variables

A static variable, also called a *class variable*, is shared by all instances of a class. There is one class variable per loaded class definition, rather than a separate copy in every object.

### Example

```java
class Employee {
    static String company = "Achhe Code";

    String name;

    Employee(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        Employee e1 = new Employee("Ali");
        Employee e2 = new Employee("Danish");

        System.out.println(e1.company);
        System.out.println(e2.company);

        Employee.company = "OpenAI";

        System.out.println(e1.company);
        System.out.println(e2.company);
    }
}
```

Output:

```text
Achhe Code
Achhe Code
OpenAI
OpenAI
```

Both employees share the same `company` variable. Changing it through the class changes the value observed by both objects.

**Best practice:** Access static members through the class name, such as `Employee.company`, rather than through an object reference.

## 2. Static methods

A static method belongs to the class and can be called without creating an object.

```java
class Calculator {
    public static int add(int a, int b) {
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        int result = Calculator.add(10, 20);
        System.out.println(result);
    }
}
```

Output:

```text
30
```

### Important rules

- A static method can directly access static members.
- It cannot directly access instance variables or instance methods because there is no implicit current object (`this`).
- It can access instance members through an explicitly supplied object reference.
- Static methods cannot be overridden polymorphically in the same way as instance methods; static method hiding is a different concept.

Example:

```java
class Example {
    static int count = 10;
    int value = 20;

    static void display() {
        System.out.println(count); // Valid

        // System.out.println(value); // Compilation error
        // System.out.println(this.value); // Compilation error
    }
}
```

## 3. Static initialization block

A static block is used to initialize static state or perform class-level initialization.

```java
class Configuration {
    static String environment;

    static {
        environment = "Production";
        System.out.println("Static block executed");
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(Configuration.environment);
    }
}
```

Output:

```text
Static block executed
Production
```

The static initializer executes when the class is initialized, before its first active use, such as accessing a non-constant static field or invoking a static method. The JVM initializes a class once per class loader, subject to the Java class-initialization rules.

If a class has multiple static initialization blocks, they execute in source-code order along with static field initializers.

## 4. Static nested classes

Java allows a nested class to be declared `static`.

```java
class Outer {
    static String message = "Hello";

    static class Nested {
        void display() {
            System.out.println(message);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Outer.Nested nested = new Outer.Nested();
        nested.display();
    }
}
```

Output:

```text
Hello
```

A static nested class does not require an instance of the outer class. Unlike a non-static inner class, it has no implicit reference to an enclosing `Outer` object.

Note that a top-level class cannot be declared `static`.

## 5. Static variable vs instance variable

| Feature | Static variable | Instance variable |
|---|---|---|
| Belongs to | Class | Individual object |
| Number of copies | One per loaded class definition | One per object |
| Initialization | When the class is initialized, subject to initialization rules | As part of object initialization |
| Access | `ClassName.variable` | `object.variable` |
| Memory concept | Class-level shared state | Object-specific state |
| Typical use | Shared configuration, common counters | Name, age, salary of an employee |

### Example

```java
class Student {
    static String college = "ABC College";
    String name;

    Student(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        Student s1 = new Student("Ali");
        Student s2 = new Student("Sara");

        System.out.println(s1.name);
        System.out.println(s2.name);

        System.out.println(Student.college);
    }
}
```

Output:

```text
Ali
Sara
ABC College
```

Here, `name` is different for each student, while `college` is shared.

## 6. Static vs instance methods

| Feature | Static method | Instance method |
|---|---|---|
| Belongs to | Class | Object |
| Requires object to call? | No | Usually yes |
| Direct access to static members | Yes | Yes |
| Direct access to instance members | No | Yes |
| Can use `this`? | No | Yes |
| Supports runtime polymorphism through overriding? | No; static methods can be hidden | Yes, when overridden appropriately |

## 7. Can static variables be thread-safe?

No. Declaring a variable `static` does not make it thread-safe.

```java
class Counter {
    static int count = 0;

    static void increment() {
        count++;
    }
}
```

If multiple threads call `increment()` concurrently, updates can be lost because `count++` is not atomic.

Possible solutions include:

**Using synchronized:**

```java
class Counter {
    private static int count = 0;

    public static synchronized void increment() {
        count++;
    }

    public static synchronized int getCount() {
        return count;
    }
}
```

**Using AtomicInteger:**

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
    private static final AtomicInteger count =
            new AtomicInteger();

    public static void increment() {
        count.incrementAndGet();
    }

    public static int getCount() {
        return count.get();
    }
}
```

Choose the mechanism based on the required concurrency guarantees.

## 8. Common interview questions

### Q1. Can we access a static variable using an object?

Yes, Java permits it, but accessing it through the class name is clearer and recommended.

```java
Employee.company; // Preferred
```

### Q2. Can a static method access a non-static variable?

Not directly. It needs an explicit object reference.

```java
class Example {
    int value = 10;

    static void printValue() {
        Example obj = new Example();
        System.out.println(obj.value);
    }
}
```

### Q3. Can we override a static method?

Static methods are hidden, not overridden. The method selected depends on the class reference used at compile time rather than runtime polymorphism on the object.

### Q4. Can a constructor be static?

No. Constructors initialize objects and cannot be declared `static`.

### Q5. Can we declare a static variable inside a method?

No. Local variables cannot be declared `static` in Java.

### Q6. Can an abstract method be static?

No. An abstract method requires subclass implementation through instance-method overriding, whereas static methods are associated with a class and cannot be overridden polymorphically.

### Q7. Is the main() method static?

Yes. The traditional Java entry point is declared `public static void main(String[] args)`, allowing the launcher to invoke it without first creating an instance of the class.

### Q8. Are static variables initialized only once?

They are initialized as part of class initialization, once per loaded class definition. Different class loaders can load separate definitions of the same class, each with its own static state.

## 9. Quick revision

- `static` means a member belongs to the class, not an individual instance.
- Static variables are shared; instance variables belong to each object.
- Static methods can be called without creating an object.
- Static blocks initialize class-level state during class initialization.
- Static nested classes do not require an enclosing-class instance.
- `static` does not guarantee thread safety.
- Static methods are hidden, not overridden.


---


# Private vs Protected Access Modifiers in Java

Java access modifiers control where classes, variables, methods, and constructors can be accessed.

The main difference is:

- **`private`:** Accessible only within the declaring top-level class's enclosing class context.
- **`protected`:** Accessible within the same package and, subject to Java's cross-package subclass access rules, from subclasses in other packages.

## 1. Difference between private and protected

| Feature | `private` | `protected` |
|---|---|---|
| Same class | Yes | Yes |
| Same package, unrelated class | No | Yes |
| Subclass in the same package | No direct access to the private member | Yes |
| Subclass in a different package | No direct access to the private member | Yes, subject to cross-package access rules |
| Direct access by unrelated class in another package | No | No |
| Method overriding | Private methods are not overridden | Protected instance methods can be overridden |
| Encapsulation | Stronger restriction | Allows controlled inheritance access |

## 2. Example of private access

```java
class Employee {
    private double salary = 50000;

    public double getSalary() {
        return salary;
    }
}

public class Main {
    public static void main(String[] args) {
        Employee employee = new Employee();

        // System.out.println(employee.salary);
        // Compilation error: salary has private access

        System.out.println(employee.getSalary());
    }
}
```

Output:

```text
50000.0
```

The `salary` field is accessible directly only within `Employee`. The public getter provides controlled access to its value.

**Why use private?**

- Protect internal implementation details.
- Prevent uncontrolled modification of fields.
- Support encapsulation and validation.
- Allow implementation changes without exposing internal state.

## 3. Example of protected access

```java
class Employee {
    protected String name = "Danish";

    protected void displayName() {
        System.out.println(name);
    }
}

class Developer extends Employee {
    public void showDetails() {
        System.out.println(name);
        displayName();
    }
}

public class Main {
    public static void main(String[] args) {
        Developer developer = new Developer();
        developer.showDetails();
    }
}
```

Output:

```text
Danish
Danish
```

`Developer` inherits from `Employee` and can access the protected field and method.

## 4. Important rule: protected across packages

This is a common Java interview question.

A subclass in a different package can access an inherited protected instance member through inheritance, but it cannot freely access that member through an arbitrary superclass object reference.

Consider these classes in separate files.

**File: `package1/Employee.java`**

```java
package package1;

public class Employee {
    protected String name = "Danish";
}
```

**File: `package2/Developer.java`**

```java
package package2;

import package1.Employee;

public class Developer extends Employee {

    public void display() {
        System.out.println(name); // Valid

        Employee employee = new Employee();

        // System.out.println(employee.name);
        // Compilation error
    }
}
```

The inherited access through `name` is valid. However, the subclass cannot access `protected` through an arbitrary `Employee` reference from another package.

This restriction applies to protected instance members; protected static members have different receiver considerations.

## 5. Can private methods be overridden?

No. Private methods are not inherited as accessible methods by subclasses, so a subclass cannot override them in the polymorphic sense. A subclass may declare another method with the same signature, but it is a separate method.

```java
class Parent {
    private void show() {
        System.out.println("Parent");
    }

    public void execute() {
        show();
    }
}

class Child extends Parent {
    public void show() {
        System.out.println("Child");
    }
}

public class Main {
    public static void main(String[] args) {
        Parent obj = new Child();
        obj.execute();
        ((Child) obj).show();
    }
}
```

Output:

```text
Parent
Child
```

`Parent.execute()` calls the private method declared inside `Parent`. It does not dispatch to `Child.show()`.

By contrast, a protected instance method can be overridden:

```java
class Parent {
    protected void show() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    @Override
    protected void show() {
        System.out.println("Child");
    }
}
```

The overriding method cannot reduce the visibility of the inherited method. For example, overriding a protected method with a private method is not allowed.

## 6. Java access modifiers at a glance

| Modifier | Same class | Same package | Subclass in another package | Unrelated class in another package |
|---|---|---|---|---|
| `private` | Yes | No | No | No |
| *(default)* | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes, subject to access rules | No |
| `public` | Yes | Yes | Yes | Yes |

*Default access* means no access modifier is specified. It is also called package-private access.

## 7. What is the use of protected members?

Use `protected` when a class is designed to provide certain implementation details or extension points to subclasses, including subclasses in other packages.

For example, a framework might provide a protected method for subclasses to customize behavior:

```java
abstract class PaymentProcessor {

    public final void processPayment() {
        validate();
        executePayment();
    }

    protected void validate() {
        System.out.println("Validating payment");
    }

    protected abstract void executePayment();
}

class CardPaymentProcessor extends PaymentProcessor {

    @Override
    protected void executePayment() {
        System.out.println("Processing card payment");
    }
}
```

Here, subclasses can customize the payment execution while callers use the public `processPayment()` method.

However, prefer `private` by default and use `protected` deliberately when inheritance access is part of the class design.

## 8. Common interview questions

### Q1. Is a private member inherited?

Private members are part of the superclass's implementation and exist as part of the superclass portion of a subclass object, but subclasses cannot access them directly by name.

### Q2. Can a protected variable be accessed from the same package?

Yes. Any class in the same package can access a protected member, subject to normal restrictions such as the member's static or instance nature.

### Q3. Can a subclass override a protected method with a public method?

Yes. Increasing visibility is allowed.

```java
class Parent {
    protected void display() {}
}

class Child extends Parent {
    @Override
    public void display() {}
}
```

### Q4. Can a top-level class be private or protected?

No. A top-level class can be `public` or package-private. Nested classes can use `private`, `protected`, or `public`, as well as package-private access.

### Q5. Which access modifier provides the strongest restriction?

`private` provides the most restrictive member access among Java's four access levels.

## 9. Quick revision

- `private` restricts direct member access to the declaring class's enclosing context.
- `protected` allows access within the same package and controlled access from subclasses in other packages.
- Private methods are not overridden; protected instance methods can be overridden.
- A cross-package subclass cannot access a protected instance member through an arbitrary superclass reference.
- Prefer `private` for encapsulation and `protected` when subclass access is intentionally part of the design.


---

# hashCode() Method in Java: Implementation and Return Type

## 1. What Is the Return Type of hashCode()?

The return type of `hashCode()` is **`int`**.

The `hashCode()` method is defined in Java's `Object` class. It returns an integer hash code used by hash-based collections such as `HashMap`, `HashSet`, and `Hashtable` to help locate objects efficiently.

Method signature:

```java
public native int hashCode();
```

The `native` keyword describes the method in `Object`; your own overridden implementation is generally an ordinary Java method.

## 2. How to Implement hashCode()

Consider a `Person` class with three fields: `name`, `age`, and `address`.

```java
import java.util.Objects;

class Person {
    private final String name;
    private final int age;
    private final String address;

    public Person(String name, int age, String address) {
        this.name = name;
        this.age = age;
        this.address = address;
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age, address);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Person)) {
            return false;
        }

        Person other = (Person) obj;

        return age == other.age
                && Objects.equals(name, other.name)
                && Objects.equals(address, other.address);
    }
}
```

Here, `Objects.hash()` calculates a hash code using all three fields and handles null values safely.

## 3. How to Implement hashCode() Manually

You can also calculate the hash code manually using a prime number such as `31`.

```java
@Override
public int hashCode() {
    int result = 1;

    result = 31 * result + age;
    result = 31 * result + Objects.hashCode(name);
    result = 31 * result + Objects.hashCode(address);

    return result;
}
```

Explanation:

1. Initialize `result` to `1`.
2. Multiply the current result by `31`.
3. Add the hash code of each field.
4. Return the final integer.

Using `31` is a common convention, not a mandatory rule. A good implementation should distribute objects reasonably well across hash codes.

## 4. Rules of hashCode() and equals()

| Rule | Explanation |
|---|---|
| Equal objects | If `a.equals(b)` is `true`, both objects must have the same hash code |
| Unequal objects | They may have the same hash code; this is called a collision |
| Consistency | Repeated calls should return the same hash code while relevant equality information remains unchanged |
| Return type | Always `int` |
| Uniqueness | A hash code is not guaranteed to be unique |

**Important:** Overriding `hashCode()` alone does not make two objects equal. If you need value-based equality, implement `equals()` consistently as well.

## 5. Example: Using hashCode() in HashMap

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

class Employee {
    private final int id;
    private final String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Employee)) {
            return false;
        }

        Employee other = (Employee) obj;

        return id == other.id
                && Objects.equals(name, other.name);
    }
}

public class HashCodeExample {
    public static void main(String[] args) {
        Map<Employee, String> employees = new HashMap<>();

        employees.put(new Employee(101, "Ali"), "Developer");

        System.out.println(
            employees.get(new Employee(101, "Ali"))
        );
    }
}
```

Output:

```text
Developer
```

Although two different `Employee` objects are created, they are equal according to `equals()` and have the same hash code. Therefore, the second object can retrieve the value associated with the first object.

## 6. One-Line Interview Answer

**`hashCode()` returns an `int` and helps hash-based collections locate objects efficiently; if two objects are equal according to `equals()`, they must return the same hash code, although collisions between unequal objects are allowed.**


---

# Difference Between `==` and `equals()` in Java

## 1. What Is the Difference Between `==` and `equals()`?

In Java, `==` and `equals()` are used to compare values or objects, but they serve different purposes.

- **`==` operator:** Compares primitive values for equality. For reference types, it checks whether both references point to the same object.
- **`equals()` method:** Checks logical equality between objects according to the method's implementation. The default implementation inherited from `Object` compares object identity, but classes can override it to compare their contents.

### Comparison Table

| Feature | `==` | `equals()` |
|---|---|---|
| Type | Operator | Method |
| Primitive types | Compares values | Not available directly on primitives |
| Reference types | Checks reference identity | Checks logical equality as implemented by the class |
| Can be customized? | No | Yes, by overriding the method |
| Default behavior for ordinary objects | Reference identity | Reference identity, inherited from `Object` |
| Common use | Compare primitive values or object identity | Compare object contents or logical equality |

## 2. Example Using Strings

```java
public class Main {
    public static void main(String[] args) {
        String s1 = new String("Java");
        String s2 = new String("Java");

        System.out.println(s1 == s2);
        System.out.println(s1.equals(s2));
    }
}
```

**Output:**

```text
false
true
```

### Explanation

- `s1 == s2` returns `false` because the references point to two distinct `String` objects.
- `s1.equals(s2)` returns `true` because `String` overrides `equals()` to compare character sequences.

Even though the contents are identical, the objects are different.

## 3. Comparing String Literals

Java maintains a pool of string literals, allowing identical literals to share the same pooled `String` object.

```java
public class Main {
    public static void main(String[] args) {
        String s1 = "Java";
        String s2 = "Java";

        System.out.println(s1 == s2);
        System.out.println(s1.equals(s2));
    }
}
```

**Output:**

```text
true
true
```

Both variables refer to the same pooled string object, so `==` returns `true`. Their contents are also equal.

**Interview tip:** Do not rely on `==` to compare string contents. Use `equals()`.

## 4. What Happens When We Compare Primitive Types?

For primitive types such as `int`, `double`, and `char`, `==` compares their values.

```java
public class Main {
    public static void main(String[] args) {
        int a = 10;
        int b = 10;

        System.out.println(a == b);
    }
}
```

**Output:**

```text
true
```

Primitive types do not have instance methods, so you cannot call `a.equals(b)` when `a` and `b` are primitive `int` variables.

For floating-point values, note that `NaN == NaN` is `false`, and positive and negative zero compare equal with `==`.

## 5. How Does `equals()` Work Internally?

Every Java class inherits `equals()` from `java.lang.Object` unless it overrides the method.

The default implementation checks whether both references identify the same object.

Conceptually, its behavior is equivalent to:

```java
public boolean equals(Object obj) {
    return this == obj;
}
```

This is a conceptual illustration of the default identity behavior, not a requirement that every JVM implement the method with this exact source code.

Classes such as `String`, `Integer`, and many Java value classes override `equals()` to compare logical values.

## 6. Example: Custom Class With and Without `equals()`

### Without Overriding `equals()`

```java
class Employee {
    String name;

    Employee(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        Employee e1 = new Employee("Ali");
        Employee e2 = new Employee("Ali");

        System.out.println(e1 == e2);
        System.out.println(e1.equals(e2));
    }
}
```

**Output:**

```text
false
false
```

Although both objects have the same `name`, the default `equals()` implementation compares their identities.

### With Overridden `equals()`

```java
import java.util.Objects;

class Employee {
    private final String name;

    Employee(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Employee other)) {
            return false;
        }

        return Objects.equals(name, other.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}

public class Main {
    public static void main(String[] args) {
        Employee e1 = new Employee("Ali");
        Employee e2 = new Employee("Ali");

        System.out.println(e1 == e2);
        System.out.println(e1.equals(e2));
    }
}
```

**Output:**

```text
false
true
```

The objects remain distinct, but `equals()` considers them logically equal because their names match.

The matching `hashCode()` implementation is important when objects are used as keys in `HashMap` or elements in `HashSet`.

## 7. Important Rules of `equals()`

A correctly implemented `equals()` method should satisfy these properties:

1. **Reflexive:** `x.equals(x)` returns `true`.
2. **Symmetric:** If `x.equals(y)` is `true`, then `y.equals(x)` must also be `true`.
3. **Transitive:** If `x.equals(y)` and `y.equals(z)` are `true`, then `x.equals(z)` must be `true`.
4. **Consistent:** Repeated calls return the same result while the relevant comparison state remains unchanged.
5. **Non-null:** For any non-null `x`, `x.equals(null)` returns `false`.

When overriding `equals()`, you should generally override `hashCode()` as well.

## 8. Common Interview Questions

### Q1. Can `==` return `true` while `equals()` returns `false`?

Yes, if a class has an incorrectly implemented or unusual custom `equals()` method. However, a well-behaved implementation should always return `true` when comparing an object with itself.

### Q2. Can `==` return `false` while `equals()` returns `true`?

Yes. Two distinct `String` objects can have equal contents:

```java
String a = new String("Java");
String b = new String("Java");

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
```

### Q3. Does `equals()` always compare values?

No. The default `Object.equals()` compares identity. A class must override it to define logical equality.

### Q4. Why should we override `hashCode()` when overriding `equals()`?

The Java contract requires equal objects to have equal hash codes. Otherwise, hash-based collections such as `HashMap` and `HashSet` may behave incorrectly when handling logically equal objects.

### Q5. How should we compare strings in Java?

Use `equals()` for case-sensitive content comparison:

```java
String a = "Java";
String b = "java";

System.out.println(a.equals(b));           // false
System.out.println(a.equalsIgnoreCase(b)); // true
```

Use `Objects.equals(a, b)` when null-safe equality comparison is needed.

## 9. One-Line Interview Answer

`==` compares primitive values or reference identity, whereas `equals()` compares objects according to their equality implementation; use `equals()` to compare logical contents when the class supports it.


---

