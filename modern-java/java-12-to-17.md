---
title: Java 12 to 17
parent: Modern Java (8 to 25)
nav_order: 3
---

# Java 12 to Java 17 Features

Java 12 through Java 17 introduced several important improvements to the Java language, standard APIs, garbage collection, and JVM. These releases made Java code more concise and expressive, particularly through **switch expressions, text blocks, records, pattern matching, and sealed classes**.

Java 17 is an LTS (Long-Term Support) release, making it particularly important for enterprise Java and Spring Boot interviews.

## 1. Java 12 Features

Java 12 was released in March 2019. Its notable changes included switch expressions as a preview feature, the Shenandoah garbage collector in certain builds, and improvements to the `String` API.

### 1.1 Switch Expressions (Preview)

Traditional switch statements primarily execute statements. Java 12 introduced a preview of switch expressions, allowing a switch to return a value.

```java
public class SwitchExample {
    public static void main(String[] args) {
        int day = 2;

        String result = switch (day) {
            case 1 -> "Monday";
            case 2 -> "Tuesday";
            case 3 -> "Wednesday";
            default -> "Other day";
        };

        System.out.println(result);
    }
}
```

**Output:**

```text
Tuesday
```

The arrow syntax avoids accidental fall-through between cases.

**Important:** Switch expressions were a preview feature in Java 12 and became a permanent feature in Java 14.

### 1.2 New String Methods

Java 12 added two useful methods:

- `indent(int n)` — adjusts indentation of lines.
- `transform(Function)` — applies a function to a string.

```java
public class StringFeatures {
    public static void main(String[] args) {
        String text = "Java";

        System.out.println(text.indent(4));
        System.out.println(text.transform(String::toUpperCase));
    }
}
```

**Output:**

```text
    Java

JAVA
```

`indent(4)` adds four spaces and also ensures the result ends with a line terminator. The exact whitespace in output therefore matters.

### 1.3 Shenandoah Garbage Collector

Shenandoah is a low-pause garbage collector designed to perform much of its work concurrently with application threads.

Its availability in Java 12 depended on the JDK distribution and build configuration; it was not universally enabled in every Java 12 distribution.

---

## 2. Java 13 Features

Java 13 was released in September 2019. Important features included text blocks as a preview feature and improvements to switch expressions.

### 2.1 Text Blocks (Preview)

Text blocks allow multiline strings to be written without repeatedly escaping quotes and line breaks.

```java
public class TextBlockExample {
    public static void main(String[] args) {
        String json = """
                {
                    "name": "Danish",
                    "language": "Java"
                }
                """;

        System.out.println(json);
    }
}
```

**Output:**

```json
{
    "name": "Danish",
    "language": "Java"
}
```

Text blocks are useful for JSON, SQL, HTML, test data, and other multiline content.

**Important:** Text blocks were previewed in Java 13, previewed again in Java 14, and became permanent in Java 15.

### 2.2 Switch Expression Improvements

Java 13 refined switch expressions and introduced the `yield` keyword for returning a value from a multi-statement switch-expression case.

```java
public class SwitchYieldExample {
    public static void main(String[] args) {
        int number = 2;

        String result = switch (number) {
            case 1 -> "One";
            case 2 -> {
                System.out.println("Processing...");
                yield "Two";
            }
            default -> "Other";
        };

        System.out.println(result);
    }
}
```

**Output:**

```text
Processing...
Two
```

Use `yield` when a switch-expression branch contains multiple statements and must produce a result.

---

## 3. Java 14 Features

Java 14 was released in March 2020. Its major features included permanent switch expressions, records as a preview feature, and helpful `NullPointerException` messages.

### 3.1 Switch Expressions Became Permanent

Java 14 finalized switch expressions, making them available without preview flags.

```java
public class SwitchExpression {
    public static void main(String[] args) {
        int month = 4;

        String season = switch (month) {
            case 12, 1, 2 -> "Winter";
            case 3, 4, 5 -> "Spring";
            case 6, 7, 8 -> "Summer";
            case 9, 10, 11 -> "Autumn";
            default -> "Invalid month";
        };

        System.out.println(season);
    }
}
```

**Output:**

```text
Spring
```

### 3.2 Records (Preview)

Records provide a concise way to declare classes primarily intended to carry data.

```java
record Employee(int id, String name) {}

public class RecordExample {
    public static void main(String[] args) {
        Employee employee = new Employee(101, "Ali");

        System.out.println(employee.id());
        System.out.println(employee.name());
        System.out.println(employee);
    }
}
```

**Output:**

```text
101
Ali
Employee[id=101, name=Ali]
```

The compiler generates common members such as:

- A canonical constructor.
- Accessor methods named after the components, such as `id()` and `name()`.
- `equals()`, `hashCode()`, and `toString()`.

Records are implicitly final and cannot extend another class. They can implement interfaces.

**Important:** Records were previewed in Java 14 and 15, then finalized in Java 16.

### 3.3 Helpful NullPointerException Messages

Java 14 improved `NullPointerException` messages to help identify which expression was null.

```java
public class NullPointerExample {
    public static void main(String[] args) {
        Employee employee = null;
        System.out.println(employee.name());
    }
}

class Employee {
    String name() {
        return "Ali";
    }
}
```

On a suitable Java 14+ runtime, the exception message can identify that `employee` was null when the method was invoked. Exact message wording depends on the runtime and expression.

This improvement helps developers debug failures faster without changing the fundamental behavior of `NullPointerException`.

---

## 4. Java 15 Features

Java 15 was released in September 2020. It finalized text blocks and introduced several other features, including hidden classes and preview support for sealed classes.

### 4.1 Text Blocks Became Permanent

Text blocks became a standard Java language feature in Java 15.

```java
public class SqlExample {
    public static void main(String[] args) {
        String sql = """
                SELECT id, name
                FROM employees
                WHERE active = true
                ORDER BY name
                """;

        System.out.println(sql);
    }
}
```

Text blocks improve readability, but they do not automatically make embedded SQL safe. Use prepared statements and parameter binding for user-supplied values.

### 4.2 Sealed Classes (Preview)

Sealed classes restrict which classes can extend a class or implement an interface.

```java
sealed interface Payment
        permits CardPayment, CashPayment {}

final class CardPayment implements Payment {}

final class CashPayment implements Payment {}
```

This restricts implementations to the explicitly permitted types.

The permitted subclasses must satisfy the sealed hierarchy rules: they must be declared `final`, `sealed`, or `non-sealed`.

**Important:** Sealed classes were previewed in Java 15 and 16 and finalized in Java 17.

### 4.3 Hidden Classes

Java 15 introduced hidden classes, primarily to support frameworks and runtime-generated classes.

They are useful for implementation details that should not be discoverable through ordinary class loading by name. They are not a replacement for regular application classes.

---

## 5. Java 16 Features

Java 16 was released in March 2021. It finalized records and pattern matching for `instanceof`, and added the convenient `Stream.toList()` method.

### 5.1 Records Became Permanent

Records became a standard feature in Java 16.

```java
record Product(int id, String name, double price) {}

public class ProductExample {
    public static void main(String[] args) {
        Product product = new Product(1, "Keyboard", 1500.0);

        System.out.println(product.name());
        System.out.println(product.price());
    }
}
```

Records reduce boilerplate when representing data.

However, a record is only shallowly immutable. If a component refers to a mutable object, that object can still be modified unless defensive copying is implemented.

### 5.2 Pattern Matching for `instanceof`

Before Java 16, code often required an explicit cast after checking the type.

```java
Object value = "Java";

if (value instanceof String) {
    String text = (String) value;
    System.out.println(text.length());
}
```

Java 16 allows the type check and variable declaration in one expression:

```java
public class InstanceofExample {
    public static void main(String[] args) {
        Object value = "Java";

        if (value instanceof String text) {
            System.out.println(text.length());
        }
    }
}
```

**Output:**

```text
4
```

The variable `text` is available where the compiler can prove the pattern matched.

### 5.3 `Stream.toList()`

Java 16 introduced `Stream.toList()`, which returns an unmodifiable list containing the stream elements.

```java
import java.util.List;

public class StreamToListExample {
    public static void main(String[] args) {
        List<String> names = List.of("Ali", "Sara", "John");

        List<String> result = names.stream()
                .filter(name -> name.length() > 3)
                .toList();

        System.out.println(result);
    }
}
```

**Output:**

```text
[Sara, John]
```

**Interview distinction:**

```java
stream.toList();
```

returns an unmodifiable list, whereas:

```java
stream.collect(Collectors.toList());
```

does not guarantee a particular mutability or implementation. If you specifically need a mutable `ArrayList`, use:

```java
List<String> result = stream.collect(
    java.util.stream.Collectors.toCollection(
        java.util.ArrayList::new
    )
);
```

### 5.4 Other Java 16 Improvements

- Stronger encapsulation of JDK internals continued.
- The Vector API was available as an incubator API.
- Improvements were made to the garbage collector and runtime performance.
- The `jpackage` tool, introduced earlier as an incubator feature, became permanent.

---

## 6. Java 17 Features

Java 17 was released in September 2021 and is an LTS release.

Its major changes include permanent sealed classes, enhanced random-number generation APIs, stronger encapsulation of JDK internals, and a preview of pattern matching for `switch`.

### 6.1 Sealed Classes Became Permanent

Java 17 finalized sealed classes and interfaces.

```java
sealed interface Shape
        permits Circle, Rectangle {}

record Circle(double radius) implements Shape {}

record Rectangle(double width, double height)
        implements Shape {}
```

The permitted types explicitly define the allowed implementations of `Shape`.

**Benefits:**

- Restricts inheritance to known types.
- Makes domain models easier to reason about.
- Helps represent controlled hierarchies, such as payment types or geometric shapes.

### 6.2 Pattern Matching for `switch` (Preview)

Java 17 introduced pattern matching for `switch` as a preview feature. It allows type patterns in switch cases.

```java
public class PatternSwitchExample {
    public static void main(String[] args) {
        Object value = "Java";

        String result = switch (value) {
            case Integer i -> "Integer: " + i;
            case String s -> "String: " + s;
            default -> "Unknown type";
        };

        System.out.println(result);
    }
}
```

**Output:**

```text
String: Java
```

To compile preview-feature code with JDK 17:

```bash
javac --enable-preview --release 17 PatternSwitchExample.java
java --enable-preview PatternSwitchExample
```

Pattern matching for `switch` was not a permanent feature in Java 17. It became permanent in Java 21 after further preview iterations.

### 6.3 Enhanced Pseudo-Random Number Generators

Java 17 added new interfaces and implementations for random-number generation, including the `RandomGenerator` API.

```java
import java.util.random.RandomGenerator;

public class RandomGeneratorExample {
    public static void main(String[] args) {
        RandomGenerator random = RandomGenerator.getDefault();

        System.out.println(random.nextInt(100));
    }
}
```

**Output:** A random integer from `0` through `99`.

This API makes it easier to work with different random-number generator algorithms. Ordinary pseudo-random generators should not be used for cryptographic secrets; use `SecureRandom` for security-sensitive random values.

### 6.4 Strong Encapsulation of JDK Internals

Java 17 strongly encapsulated many internal JDK APIs, continuing the migration work begun in earlier releases.

Applications that depended on unsupported internal APIs may encounter access errors.

**Interview tip:** Prefer supported Java APIs over internal packages such as `sun.*`. When migrating older applications to Java 17, check dependencies that rely on reflective access to JDK internals.

### 6.5 Security Manager Deprecated for Removal

Java 17 deprecated the Security Manager for removal. This signaled that the feature was moving toward removal in a future release.

Do not confuse deprecation with removal: the Security Manager still existed in Java 17, although its use was discouraged.

### 6.6 Other Java 17 Improvements

- Foreign Function and Memory API available as an incubator API.
- Vector API available as an incubator API.
- Context-specific deserialization filters.
- Additional runtime and performance improvements.

---

## 7. Java 12 to Java 17 Feature Comparison

| Java version | Major features | Status of notable language features |
|---|---|---|
| Java 12 | Switch expressions, `String.indent()`, `String.transform()` | Switch expressions preview |
| Java 13 | Text blocks, switch-expression improvements, `yield` | Text blocks preview |
| Java 14 | Switch expressions, records, helpful NPE messages | Switch expressions final; records preview |
| Java 15 | Text blocks, hidden classes, sealed classes | Text blocks final; sealed classes preview |
| Java 16 | Records, pattern matching for `instanceof`, `Stream.toList()` | Records and `instanceof` pattern matching final |
| Java 17 | Sealed classes, enhanced random generators, stronger encapsulation | Sealed classes final; pattern matching for `switch` preview |

## 8. Important Java 8, 11, and 17 Differences

| Feature | Java 8 | Java 11 | Java 17 |
|---|---|---|---|
| Lambda expressions | Available | Available | Available |
| `var` local variables | Not available | Available | Available |
| `var` in lambda parameters | Not available | Available | Available |
| Standard HTTP Client API | Not available | Available | Available |
| `String.isBlank()` and `strip()` | Not available | Available | Available |
| Text blocks | Not available | Not available | Available |
| Switch expressions | Not available | Not available | Available |
| Records | Not available | Not available | Available |
| Pattern matching for `instanceof` | Not available | Not available | Available |
| Sealed classes | Not available | Not available | Available |
| `Stream.toList()` | Not available | Not available | Available |

## 9. Common Java 12–17 Interview Questions

### Q1. Which Java version introduced switch expressions?

Switch expressions were previewed in Java 12 and became permanent in Java 14.

### Q2. Which Java version introduced text blocks?

Text blocks were previewed in Java 13 and became permanent in Java 15.

### Q3. What is a record in Java?

A record is a concise class declaration for data-oriented types. The compiler generates component accessors, a canonical constructor, and standard implementations of `equals()`, `hashCode()`, and `toString()`.

Records became permanent in Java 16.

### Q4. What are sealed classes?

Sealed classes and interfaces restrict which classes can extend or implement them. They became permanent in Java 17.

### Q5. What is pattern matching for `instanceof`?

It combines a type check with a pattern variable, avoiding an explicit cast when the condition matches.

```java
if (obj instanceof String text) {
    System.out.println(text.length());
}
```

This feature became permanent in Java 16.

### Q6. What is the difference between `Stream.toList()` and `Collectors.toList()`?

`Stream.toList()` returns an unmodifiable list. `Collectors.toList()` does not promise a specific list implementation or mutability.

### Q7. What is the `yield` keyword in switch expressions?

`yield` returns a value from a block-style switch-expression branch.

### Q8. Is pattern matching for `switch` final in Java 17?

No. It was a preview feature in Java 17 and became permanent in Java 21.

### Q9. Which Java versions are LTS among Java 8, 11, and 17?

All three are LTS releases. Java 11 and Java 17 are particularly important when discussing modern enterprise Java features.

### Q10. Which Java version should a developer focus on for interviews?

For Core Java interviews, understand Java 8 thoroughly, Java 11 API improvements, and Java 17 language features. Be able to explain lambdas, streams, `Optional`, records, sealed classes, switch expressions, pattern matching, collections, concurrency, and JVM fundamentals.

## Java 12 to Java 17: One-Line Interview Answer

Java 12–17 progressively introduced switch expressions, text blocks, records, pattern matching for `instanceof`, sealed classes, `Stream.toList()`, enhanced random generators, and JVM improvements, with Java 17 serving as an important LTS release.