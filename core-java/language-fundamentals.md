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

---
title: How Java's equals() and hashCode() Methods Work Together
parent: Core Java
nav_order: 6
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