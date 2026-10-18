---
title: Java 8
parent: Modern Java (8 to 25)
nav_order: 1
---

# Java 8
{: .no_toc }

Lambdas, streams and the start of modern Java.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

# Features of Java 8

Java 8, released in March 2014, was a major milestone in Java's evolution. It introduced functional programming features, a modern Date and Time API, improved collection processing, and enhancements to asynchronous programming.

## 1. Major Features of Java 8

| Feature | Purpose |
|---|---|
| Lambda expressions | Write concise implementations of functional interfaces |
| Functional interfaces | Represent a single abstract operation |
| Stream API | Process sequences of data declaratively |
| Default methods in interfaces | Add implementations to interfaces without requiring every existing implementation to change |
| Method references | Refer to existing methods using `::` |
| Constructor references | Refer to constructors using `ClassName::new` |
| `Optional` | Represent a value that may be absent |
| Date and Time API | Provide modern date, time, duration, and time-zone classes |
| `Collectors` | Collect and aggregate stream results |
| Static methods in interfaces | Define utility methods associated with an interface |
| `forEach()` | Perform an action for each element |
| Parallel array sorting | Sort arrays using `Arrays.parallelSort()` |
| Concurrency enhancements | Introduce `CompletableFuture` and other concurrency improvements |
| JDBC enhancements | Improve SQL date/time integration and add interface default methods |

---

## 2. Lambda Expressions

A **lambda expression** is a concise way to provide an implementation of a functional interface. It can represent behavior that is passed to another method as an argument.

### Syntax

```java
(parameters) -> expression
```

Or:

```java
(parameters) -> {
    // Multiple statements
}
```

### Example 1: Adding Two Numbers

```java
interface MathOperation {
    int operate(int a, int b);
}

public class LambdaDemo {
    public static void main(String[] args) {
        MathOperation addition = (a, b) -> a + b;

        System.out.println(addition.operate(5, 3));
    }
}
```

Output:

```text
8
```

### Example 2: Printing a Greeting

```java
java.util.function.Consumer<String> greeting =
        name -> System.out.println("Hello, " + name);

greeting.accept("Danish");
```

Output:

```text
Hello, Danish
```

### Advantages

- Reduces boilerplate code.
- Makes behavior easier to pass as an argument.
- Works naturally with the Stream API.
- Supports functional programming patterns.

---

## 3. Functional Interfaces

A **functional interface** is an interface with exactly one abstract method, ignoring methods that do not count as abstract interface methods, such as public methods corresponding to those declared in `Object`.

It can have multiple default and static methods.

### Example

```java
@FunctionalInterface
interface Greeting {
    void sayHello(String name);
}

public class FunctionalInterfaceDemo {
    public static void main(String[] args) {
        Greeting greeting =
                name -> System.out.println("Hello, " + name);

        greeting.sayHello("Danish");
    }
}
```

Output:

```text
Hello, Danish
```

The `@FunctionalInterface` annotation is optional, but it helps the compiler verify that the interface satisfies the functional-interface rules.

### Common Built-in Functional Interfaces

| Interface | Main method | Purpose |
|---|---|---|
| `Predicate<T>` | `test(T)` | Tests a condition and returns `boolean` |
| `Function<T, R>` | `apply(T)` | Transforms a value |
| `Consumer<T>` | `accept(T)` | Consumes a value without returning a result |
| `Supplier<T>` | `get()` | Supplies a value |
| `UnaryOperator<T>` | `apply(T)` | Transforms a value into another value of the same type |
| `BinaryOperator<T>` | `apply(T, T)` | Combines two values of the same type |

Example:

```java
import java.util.function.Predicate;

Predicate<Integer> isEven = number -> number % 2 == 0;

System.out.println(isEven.test(10));
System.out.println(isEven.test(7));
```

Output:

```text
true
false
```

---

## 4. Stream API

The Stream API processes data declaratively through a pipeline of operations.

A stream does not store its own collection of data. Instead, it processes elements from a source such as a collection or array.

### Example: Filter and Transform Names

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class StreamDemo {
    public static void main(String[] args) {
        List<String> names =
                Arrays.asList("Danish", "Anna", "John", "Alex");

        List<String> result = names.stream()
                .filter(name -> name.startsWith("A"))
                .map(String::toUpperCase)
                .collect(Collectors.toList());

        System.out.println(result);
    }
}
```

Output:

```text
[ANNA, ALEX]
```

### Types of Stream Operations

**Intermediate operations** return another stream:

- `filter()`
- `map()`
- `flatMap()`
- `sorted()`
- `distinct()`
- `limit()`

**Terminal operations** produce a result or side effect:

- `collect()`
- `forEach()`
- `count()`
- `reduce()`
- `min()`
- `max()`

### Stream vs Collection

| Collection | Stream |
|---|---|
| Stores or represents data | Processes elements from a source |
| Can usually be traversed repeatedly | A stream is consumed by a terminal operation and cannot be reused |
| Often uses external iteration with loops | Supports internal iteration through operations |
| Does not generally provide lazy pipeline evaluation | Intermediate operations are generally lazy |

### Important Interview Point

Streams do not automatically make code faster. Parallel streams can help with suitable workloads, but overhead, ordering requirements, shared state, and task costs can make them slower.

---

## 5. Default Methods in Interfaces

Java 8 introduced **default methods**, which allow interfaces to provide method implementations.

They help evolve existing interfaces without forcing every existing implementing class to add an implementation of a newly introduced method.

### Example

```java
interface Vehicle {
    void start();

    default void horn() {
        System.out.println("Beep beep!");
    }
}

class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car started");
    }
}

public class DefaultMethodDemo {
    public static void main(String[] args) {
        Car car = new Car();

        car.start();
        car.horn();
    }
}
```

Output:

```text
Car started
Beep beep!
```

A class can override a default method.

### What Happens if Two Interfaces Have the Same Default Method?

The implementing class must resolve a conflict between unrelated interfaces providing the same default method signature.

```java
interface A {
    default void greet() {
        System.out.println("Hello from A");
    }
}

interface B {
    default void greet() {
        System.out.println("Hello from B");
    }
}

class C implements A, B {
    @Override
    public void greet() {
        A.super.greet();
    }
}
```

Here, `C` explicitly selects the implementation from `A`.

---

## 6. Method and Constructor References

Method references use the `::` operator to refer to an existing method. They are commonly used when a lambda expression simply calls an existing method.

### Four Common Forms

| Form | Example |
|---|---|
| Static method reference | `Integer::parseInt` |
| Instance method of a particular object | `System.out::println` |
| Instance method of an arbitrary object of a type | `String::toUpperCase` |
| Constructor reference | `ArrayList::new` |

### Example: Method Reference

```java
import java.util.Arrays;
import java.util.List;

public class MethodReferenceDemo {
    public static void main(String[] args) {
        List<String> names =
                Arrays.asList("Danish", "Anna", "John");

        names.forEach(System.out::println);
    }
}
```

### Example: Constructor Reference

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Supplier;

Supplier<List<String>> supplier = ArrayList::new;

List<String> names = supplier.get();
names.add("Danish");

System.out.println(names);
```

Output:

```text
[Danish]
```

---

## 7. Optional Class

`Optional<T>` represents a value that may or may not be present. It is primarily useful for making potentially absent return values explicit.

### Example

```java
import java.util.Optional;

public class OptionalDemo {
    public static void main(String[] args) {
        Optional<String> name =
                Optional.ofNullable(null);

        System.out.println(name.orElse("Unknown"));
    }
}
```

Output:

```text
Unknown
```

### Important Methods

| Method | Purpose |
|---|---|
| `Optional.of(value)` | Creates an Optional from a non-null value |
| `Optional.ofNullable(value)` | Creates an Optional that may be empty |
| `Optional.empty()` | Creates an empty Optional |
| `isPresent()` | Checks whether a value exists |
| `ifPresent(action)` | Executes an action when a value exists |
| `map(function)` | Transforms a present value |
| `flatMap(function)` | Transforms a value into another Optional without nesting |
| `orElse(value)` | Returns the value or a fallback |
| `orElseGet(supplier)` | Computes a fallback when empty |
| `orElseThrow()` | Throws `NoSuchElementException` if empty |

**Interview point:** `orElse()` evaluates its fallback argument before the call, while `orElseGet()` invokes its supplier only when the Optional is empty.

Optional does not eliminate every `NullPointerException`, and it is generally intended for return values rather than as a replacement for every nullable field or method parameter.

---

## 8. Date and Time API

Java 8 introduced the modern `java.time` package, improving upon many design and thread-safety limitations of legacy date/time classes.

### Important Classes

| Class | Purpose |
|---|---|
| `LocalDate` | Date without time or time zone |
| `LocalTime` | Time without date or time zone |
| `LocalDateTime` | Date and time without a time zone |
| `ZonedDateTime` | Date and time with a time zone |
| `Instant` | Point on the UTC timeline |
| `Period` | Date-based amount of time |
| `Duration` | Time-based amount of time |
| `DateTimeFormatter` | Formats and parses date/time values |

### Example

```java
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class DateTimeDemo {
    public static void main(String[] args) {
        LocalDate birthday = LocalDate.of(1995, 5, 20);

        System.out.println(birthday);

        LocalDateTime now = LocalDateTime.now();

        DateTimeFormatter formatter =
                DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm");

        System.out.println(now.format(formatter));
    }
}
```

Example output:

```text
1995-05-20
18-10-2026 16:30
```

The current date and time output depends on the execution environment.

Most core `java.time` date/time types are immutable and thread-safe. `LocalDateTime` does not itself contain a time zone; use `ZonedDateTime` or `Instant` when time-zone or global timeline semantics are needed.

---

## 9. Stream for File I/O

Java 8 added stream-based file-processing options through methods such as `Files.lines()`.

### Read a File Using a Stream

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class FileStreamDemo {
    public static void main(String[] args) throws IOException {
        try (var lines = Files.lines(Path.of("file.txt"))) {
            lines.filter(line -> line.contains("Java"))
                 .forEach(System.out::println);
        }
    }
}
```

This example uses local variable type inference (`var`), which was introduced in Java 10. For Java 8 source compatibility, declare the stream type explicitly:

```java
try (java.util.stream.Stream<String> lines =
         Files.lines(java.nio.file.Paths.get("file.txt"))) {
    lines.filter(line -> line.contains("Java"))
         .forEach(System.out::println);
}
```

`Files.lines()` processes lines lazily and the stream should be closed, which is why try-with-resources is used.

### Write Lines to a File

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Arrays;

public class FileWriteDemo {
    public static void main(String[] args) throws IOException {
        Files.write(
                Paths.get("output.txt"),
                Arrays.asList("Java 8", "Streams")
        );
    }
}
```

---

## 10. Collectors Utility

`Collectors` provides factory methods for collecting and aggregating elements in a stream.

### Common Collectors

| Collector | Purpose |
|---|---|
| `toList()` | Collects elements into a list |
| `toSet()` | Collects elements into a set |
| `toMap()` | Collects elements into a map |
| `joining()` | Joins strings |
| `groupingBy()` | Groups elements by a classification function |
| `partitioningBy()` | Splits elements into `true` and `false` groups |
| `counting()` | Counts elements |
| `summarizingInt()` | Computes integer summary statistics |
| `averagingInt()` | Computes an average |

### Example: Joining Names

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class CollectorsDemo {
    public static void main(String[] args) {
        List<String> names =
                Arrays.asList("John", "Sham", "Danish");

        String result = names.stream()
                .collect(Collectors.joining(", "));

        System.out.println(result);
    }
}
```

Output:

```text
John, Sham, Danish
```

### Example: Grouping Employees by Department

```java
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

class Employee {
    private final String name;
    private final String department;

    Employee(String name, String department) {
        this.name = name;
        this.department = department;
    }

    public String getDepartment() {
        return department;
    }

    public String getName() {
        return name;
    }
}

public class GroupingDemo {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
                new Employee("A", "IT"),
                new Employee("B", "HR"),
                new Employee("C", "IT")
        );

        Map<String, List<Employee>> grouped =
                employees.stream()
                        .collect(Collectors.groupingBy(
                                Employee::getDepartment));

        System.out.println(grouped.keySet());
    }
}
```

The map groups employees under their department names. The order of keys is not guaranteed by `groupingBy()` without an explicitly supplied map factory.

---

## 11. Static Methods in Interfaces

Java 8 allows interfaces to declare static methods. These methods belong to the interface itself and are called using its name.

```java
interface Calculator {
    static int add(int a, int b) {
        return a + b;
    }
}

public class StaticInterfaceDemo {
    public static void main(String[] args) {
        System.out.println(Calculator.add(5, 3));
    }
}
```

Output:

```text
8
```

Important points:

- Call the method using `InterfaceName.methodName()`.
- Interface static methods are not inherited as instance methods by implementing classes.
- They cannot be overridden as interface instance methods.
- They are useful for interface-related utility operations.

---

## 12. forEach() Method

Java 8 introduced `Iterable.forEach()`, which enables concise iteration over collection elements. Streams also provide a terminal `forEach()` operation.

### Example

```java
import java.util.Arrays;
import java.util.List;

public class ForEachDemo {
    public static void main(String[] args) {
        List<String> names =
                Arrays.asList("John", "Anna", "Danish");

        names.forEach(name -> System.out.println(name));
    }
}
```

Equivalent method-reference syntax:

```java
names.forEach(System.out::println);
```

The `forEach()` method accepts a `Consumer` action.

**Important:** Do not rely on execution order for parallel streams. If encounter order matters, consider `forEachOrdered()` or a sequential stream.

---

## 13. Parallel Array Sorting

Java 8 introduced `Arrays.parallelSort()` for sorting arrays using parallel processing where appropriate.

### Example

```java
import java.util.Arrays;

public class ParallelSortDemo {
    public static void main(String[] args) {
        int[] numbers = {3, 1, 4, 1, 5, 9, 2, 6};

        Arrays.parallelSort(numbers);

        System.out.println(Arrays.toString(numbers));
    }
}
```

Output:

```text
[1, 1, 2, 3, 4, 5, 6, 9]
```

### `Arrays.sort()` vs `Arrays.parallelSort()`

| `Arrays.sort()` | `Arrays.parallelSort()` |
|---|---|
| Standard array sorting method | Parallel-capable array sorting method |
| Often preferable for smaller arrays | May help for sufficiently large arrays |
| Does not expose parallel execution as its purpose | Can use multiple worker threads |

Parallel sorting is not guaranteed to be faster for every array. Performance depends on array size, data type, hardware, and available resources.

---

## 14. Concurrency Enhancements

Java 8 improved asynchronous programming and parallel data processing.

### CompletableFuture

`CompletableFuture<T>` supports asynchronous computation, chaining, combining results, and exception handling.

```java
import java.util.concurrent.CompletableFuture;

public class CompletableFutureDemo {
    public static void main(String[] args) {
        CompletableFuture.supplyAsync(() -> 10)
                .thenApply(number -> number * 2)
                .thenAccept(System.out::println)
                .join();
    }
}
```

Output:

```text
20
```

Common methods include:

- `supplyAsync()` — starts an asynchronous computation that produces a value.
- `runAsync()` — starts an asynchronous computation without a result.
- `thenApply()` — transforms a completed result.
- `thenCompose()` — chains a computation that itself returns a completion stage.
- `thenCombine()` — combines two independent results.
- `exceptionally()` — provides a recovery value after an exceptional completion.
- `allOf()` — creates a future that completes when all supplied futures complete.

Async methods without an explicit executor commonly use the common `ForkJoinPool`. Non-async continuation methods do not guarantee execution on a separate thread.

### Parallel Streams

```java
import java.util.Arrays;

public class ParallelStreamDemo {
    public static void main(String[] args) {
        Arrays.asList(1, 2, 3, 4, 5)
                .parallelStream()
                .map(number -> number * number)
                .forEach(System.out::println);
    }
}
```

Output order is not guaranteed because the stream is parallel.

Java 8 also introduced enhancements such as `LongAdder`, `LongAccumulator`, and additional `CompletableFuture` capabilities in its concurrency libraries.

---

## 15. JDBC Enhancements in Java 8

Java 8 made several improvements to JDBC and SQL-related APIs.

### Important Enhancements

- Support for `java.time` types in JDBC, including `LocalDate`, `LocalTime`, `LocalDateTime`, and `OffsetDateTime`, through suitable JDBC 4.2 drivers.
- Default methods added to certain JDBC interfaces, including methods that simplify working with large update counts and SQL types.
- Better interoperability between Java's modern date/time API and database date/time columns.

### Example: Reading a SQL DATE

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.time.LocalDate;

public class JdbcDateDemo {
    public static void readDate(Connection connection)
            throws Exception {

        String sql = "SELECT birth_date FROM employee WHERE id = ?";

        try (PreparedStatement statement =
                     connection.prepareStatement(sql)) {

            statement.setInt(1, 101);

            try (ResultSet resultSet = statement.executeQuery()) {
                if (resultSet.next()) {
                    LocalDate birthDate =
                            resultSet.getObject(
                                    "birth_date", LocalDate.class);

                    System.out.println(birthDate);
                }
            }
        }
    }
}
```

This example assumes a compatible JDBC driver and a SQL `DATE` column named `birth_date`.

**Correction to a common misconception:** Java 8 did not introduce a general Stream API directly on JDBC `ResultSet`. A `ResultSet` is still typically processed using `next()` in a loop, or wrapped in a stream by application code or a library.

---

## 16. Other Java 8 Enhancements

In addition to the main features above, Java 8 included:

- **Base64 API:** `java.util.Base64` for encoding and decoding Base64 data.
- **Repeating annotations:** Support for applying certain annotations multiple times to the same declaration.
- **Type annotations:** Annotations can be used in more type-use locations.
- **Nashorn JavaScript engine:** Introduced in Java 8, though deprecated in Java 11 and removed from the JDK in Java 15.
- **StringJoiner:** Convenient construction of delimited strings.
- **Map enhancements:** Methods such as `getOrDefault()`, `forEach()`, `computeIfAbsent()`, `computeIfPresent()`, and `merge()`.
- **Collection enhancements:** Methods such as `removeIf()`, `replaceAll()`, and `sort()`.

### Example: Base64

```java
import java.util.Base64;

public class Base64Demo {
    public static void main(String[] args) {
        String original = "Java 8";

        String encoded = Base64.getEncoder()
                .encodeToString(original.getBytes(
                        java.nio.charset.StandardCharsets.UTF_8));

        System.out.println(encoded);
    }
}
```

Output:

```text
SmF2YSA4
```

Base64 is an encoding format, not encryption.

---

## 17. Frequently Asked Java 8 Interview Questions

### Q1. What is the biggest feature of Java 8?

Lambda expressions and the Stream API are among Java 8's most significant features because they enable concise functional-style programming and declarative data processing.

### Q2. Can a functional interface contain default and static methods?

Yes. It must have exactly one abstract method, but it may also have default and static methods.

### Q3. What is the difference between `map()` and `flatMap()`?

- `map()` transforms each element into a result.
- `flatMap()` transforms each element into a stream or similar container and flattens the results into one stream.

### Q4. What is the difference between `orElse()` and `orElseGet()`?

`orElse()` evaluates its fallback expression before the method call. `orElseGet()` evaluates its supplier only when the Optional is empty.

### Q5. Are Java 8 streams reusable?

No. Once a terminal operation consumes a stream, it cannot be reused. Create a new stream from the source for another traversal.

### Q6. Does Java 8 support multiple inheritance?

Java does not support multiple inheritance of classes. However, a class can implement multiple interfaces, including interfaces with default methods, subject to method-conflict resolution rules.

### Q7. Are Java 8 date/time classes thread-safe?

The main classes in `java.time` are immutable and thread-safe. Legacy `Date` and `Calendar` have different design characteristics.

### Q8. Does a parallel stream always improve performance?

No. Parallel processing adds coordination overhead and can perform worse for small datasets, inexpensive operations, or workloads with contention.

---

## 18. One-Line Interview Answer

Java 8 introduced lambda expressions, functional interfaces, the Stream API, default and static interface methods, method references, Optional, the modern Date and Time API, collectors, parallel array sorting, CompletableFuture, and JDBC improvements, making Java more expressive and improving functional-style and concurrent programming.