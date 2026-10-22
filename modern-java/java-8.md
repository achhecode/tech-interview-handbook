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

#### 1. Predicate<T> - `test(T)`

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

#### 2. Function<T, R> - `apply(T)`

```java
import java.util.function.Function;

Function<String, Integer> stringLength = text -> text.length();

System.out.println(stringLength.apply("Hello"));
System.out.println(stringLength.apply("Java Streams"));
```

Output:

```text
5
12
```

#### 3. Consumer<T> - `accept(T)`

```java
import java.util.function.Consumer;

Consumer<String> printer = message -> System.out.println("Message: " + message);

printer.accept("Hello, World!");
printer.accept("Learning Java");
```

Output:

```text
Message: Hello, World!
Message: Learning Java
```

#### 4. Supplier<T> - `get()`

```java
import java.util.function.Supplier;

Supplier<Double> randomValue = () -> Math.random();
Supplier<String> greeting = () -> "Welcome to Java!";

System.out.println(greeting.get());
System.out.println(randomValue.get() < 1.0);
```

Output:

```text
Welcome to Java!
true
```

#### 5. UnaryOperator<T> - `apply(T)`

```java
import java.util.function.UnaryOperator;

UnaryOperator<String> toUpper = text -> text.toUpperCase();
UnaryOperator<Integer> square = number -> number * number;

System.out.println(toUpper.apply("java"));
System.out.println(square.apply(6));
```

Output:

```text
JAVA
36
```

#### 6. BinaryOperator<T> - `apply(T, T)`

```java
import java.util.function.BinaryOperator;

BinaryOperator<Integer> add = (a, b) -> a + b;
BinaryOperator<String> concat = (a, b) -> a + " " + b;

System.out.println(add.apply(10, 20));
System.out.println(concat.apply("Hello", "World"));
```

Output:

```text
30
Hello World
```

#### Complete Runnable Program

```java
import java.util.function.*;

public class FunctionalInterfacesDemo {
    public static void main(String[] args) {

        // Predicate
        Predicate<Integer> isEven = number -> number % 2 == 0;
        System.out.println("Predicate: " + isEven.test(10));

        // Function
        Function<String, Integer> stringLength = text -> text.length();
        System.out.println("Function: " + stringLength.apply("Hello"));

        // Consumer
        Consumer<String> printer = message -> System.out.println("Consumer: " + message);
        printer.accept("Hello, World!");

        // Supplier
        Supplier<String> greeting = () -> "Welcome to Java!";
        System.out.println("Supplier: " + greeting.get());

        // UnaryOperator
        UnaryOperator<Integer> square = number -> number * number;
        System.out.println("UnaryOperator: " + square.apply(6));

        // BinaryOperator
        BinaryOperator<Integer> add = (a, b) -> a + b;
        System.out.println("BinaryOperator: " + add.apply(10, 20));
    }
}
```

Output:

```text
Predicate: true
Function: 5
Consumer: Hello, World!
Supplier: Welcome to Java!
UnaryOperator: 36
BinaryOperator: 30
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

### Sample Data Used in the Examples

```java
import java.util.Arrays;
import java.util.List;

List<Integer> numbers = Arrays.asList(5, 3, 8, 3, 1, 8, 9);
List<String> names = Arrays.asList("Danish", "Anna", "John", "Alex");
```

### Intermediate Operations

#### 1. `filter()` - keeps elements that match a condition

```java
List<Integer> evens = numbers.stream()
        .filter(n -> n % 2 == 0)
        .collect(Collectors.toList());

System.out.println(evens);
```

Output:

```text
[8, 8]
```

#### 2. `map()` - transforms each element

```java
List<Integer> lengths = names.stream()
        .map(String::length)
        .collect(Collectors.toList());

System.out.println(lengths);
```

Output:

```text
[6, 4, 4, 4]
```

#### 3. `flatMap()` - flattens nested collections into one stream

```java
List<List<Integer>> nested = Arrays.asList(
        Arrays.asList(1, 2),
        Arrays.asList(3, 4),
        Arrays.asList(5)
);

List<Integer> flat = nested.stream()
        .flatMap(List::stream)
        .collect(Collectors.toList());

System.out.println(flat);
```

Output:

```text
[1, 2, 3, 4, 5]
```

#### 4. `sorted()` - sorts elements

```java
import java.util.Comparator;

List<Integer> ascending = numbers.stream()
        .sorted()
        .collect(Collectors.toList());

List<Integer> descending = numbers.stream()
        .sorted(Comparator.reverseOrder())
        .collect(Collectors.toList());

System.out.println(ascending);
System.out.println(descending);
```

Output:

```text
[1, 3, 3, 5, 8, 8, 9]
[9, 8, 8, 5, 3, 3, 1]
```

#### 5. `distinct()` - removes duplicates

```java
List<Integer> unique = numbers.stream()
        .distinct()
        .collect(Collectors.toList());

System.out.println(unique);
```

Output:

```text
[5, 3, 8, 1, 9]
```

#### 6. `limit()` - keeps only the first N elements

```java
List<Integer> firstThree = numbers.stream()
        .limit(3)
        .collect(Collectors.toList());

System.out.println(firstThree);
```

Output:

```text
[5, 3, 8]
```

### Terminal Operations

#### 7. `collect()` - gathers elements into a collection or other result

```java
List<String> list = names.stream()
        .collect(Collectors.toList());

String joined = names.stream()
        .collect(Collectors.joining(", "));

System.out.println(list);
System.out.println(joined);
```

Output:

```text
[Danish, Anna, John, Alex]
Danish, Anna, John, Alex
```

#### 8. `forEach()` - performs an action on each element

```java
names.stream()
        .forEach(name -> System.out.println("Hello, " + name));
```

Output:

```text
Hello, Danish
Hello, Anna
Hello, John
Hello, Alex
```

#### 9. `count()` - counts elements

```java
long count = names.stream()
        .filter(name -> name.startsWith("A"))
        .count();

System.out.println(count);
```

Output:

```text
2
```

#### 10. `reduce()` - combines elements into a single value

```java
int sum = numbers.stream()
        .reduce(0, Integer::sum);

System.out.println(sum);
```

Output:

```text
37
```

#### 11. `min()` - finds the smallest element

```java
import java.util.Optional;

Optional<Integer> min = numbers.stream()
        .min(Integer::compare);

System.out.println(min.get());
```

Output:

```text
1
```

#### 12. `max()` - finds the largest element

```java
Optional<Integer> max = numbers.stream()
        .max(Integer::compare);

System.out.println(max.get());
```

Output:

```text
9
```

### Complete Runnable Program

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

public class StreamOperationsDemo {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(5, 3, 8, 3, 1, 8, 9);
        List<String> names = Arrays.asList("Danish", "Anna", "John", "Alex");

        // Intermediate operations
        System.out.println("filter:   " + numbers.stream()
                .filter(n -> n % 2 == 0)
                .collect(Collectors.toList()));

        System.out.println("map:      " + names.stream()
                .map(String::length)
                .collect(Collectors.toList()));

        System.out.println("flatMap:  " + Arrays.asList(
                        Arrays.asList(1, 2),
                        Arrays.asList(3, 4),
                        Arrays.asList(5))
                .stream()
                .flatMap(List::stream)
                .collect(Collectors.toList()));

        System.out.println("sorted:   " + numbers.stream()
                .sorted(Comparator.reverseOrder())
                .collect(Collectors.toList()));

        System.out.println("distinct: " + numbers.stream()
                .distinct()
                .collect(Collectors.toList()));

        System.out.println("limit:    " + numbers.stream()
                .limit(3)
                .collect(Collectors.toList()));

        // Terminal operations
        System.out.println("collect:  " + names.stream()
                .filter(name -> name.startsWith("A"))
                .map(String::toUpperCase)
                .collect(Collectors.toList()));

        System.out.print("forEach:  ");
        names.stream().forEach(name -> System.out.print(name + " "));
        System.out.println();

        System.out.println("count:    " + names.stream()
                .filter(name -> name.startsWith("A"))
                .count());

        System.out.println("reduce:   " + numbers.stream()
                .reduce(0, Integer::sum));

        Optional<Integer> min = numbers.stream().min(Integer::compare);
        Optional<Integer> max = numbers.stream().max(Integer::compare);

        System.out.println("min:      " + min.get());
        System.out.println("max:      " + max.get());
    }
}
```

Output:

```text
filter:   [8, 8]
map:      [6, 4, 4, 4]
flatMap:  [1, 2, 3, 4, 5]
sorted:   [9, 8, 8, 5, 3, 3, 1]
distinct: [5, 3, 8, 1, 9]
limit:    [5, 3, 8]
collect:  [ANNA, ALEX]
forEach:  Danish Anna John Alex 
count:    2
reduce:   37
min:      1
max:      9
```

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

### Examples of Each Method

#### 1. `Optional.of(value)` - creates an Optional from a non-null value

```java
import java.util.Optional;

Optional<String> name = Optional.of("Danish");

System.out.println(name);
System.out.println(name.get());
```

Output:

```text
Optional[Danish]
Danish
```

Note: `Optional.of(null)` throws a `NullPointerException`.

```java
try {
    Optional<String> bad = Optional.of(null);
} catch (NullPointerException e) {
    System.out.println("NullPointerException thrown");
}
```

Output:

```text
NullPointerException thrown
```

#### 2. `Optional.ofNullable(value)` - creates an Optional that may be empty

```java
Optional<String> present = Optional.ofNullable("Java");
Optional<String> absent = Optional.ofNullable(null);

System.out.println(present);
System.out.println(absent);
```

Output:

```text
Optional[Java]
Optional.empty
```

#### 3. `Optional.empty()` - creates an empty Optional

```java
Optional<String> empty = Optional.empty();

System.out.println(empty);
System.out.println(empty.isPresent());
```

Output:

```text
Optional.empty
false
```

#### 4. `isPresent()` - checks whether a value exists

```java
Optional<String> present = Optional.of("Hello");
Optional<String> absent = Optional.empty();

System.out.println(present.isPresent());
System.out.println(absent.isPresent());
```

Output:

```text
true
false
```

#### 5. `ifPresent(action)` - runs an action only when a value exists

```java
Optional<String> present = Optional.of("Danish");
Optional<String> absent = Optional.empty();

present.ifPresent(value -> System.out.println("Hello, " + value));
absent.ifPresent(value -> System.out.println("This will not print"));
```

Output:

```text
Hello, Danish
```

#### 6. `map(function)` - transforms a present value

```java
Optional<String> name = Optional.of("danish");

Optional<String> upper = name.map(String::toUpperCase);
Optional<Integer> length = name.map(String::length);

System.out.println(upper);
System.out.println(length);
System.out.println(Optional.<String>empty().map(String::toUpperCase));
```

Output:

```text
Optional[DANISH]
Optional[6]
Optional.empty
```

#### 7. `flatMap(function)` - transforms into another Optional without nesting

```java
Optional<String> name = Optional.of("Danish");

// map() would produce Optional<Optional<String>>
Optional<Optional<String>> nested =
        name.map(n -> Optional.of(n.toUpperCase()));

// flatMap() flattens it into Optional<String>
Optional<String> flat =
        name.flatMap(n -> Optional.of(n.toUpperCase()));

System.out.println(nested);
System.out.println(flat);
```

Output:

```text
Optional[Optional[DANISH]]
Optional[DANISH]
```

#### 8. `orElse(value)` - returns the value or a fallback

```java
Optional<String> present = Optional.of("Danish");
Optional<String> absent = Optional.empty();

System.out.println(present.orElse("Unknown"));
System.out.println(absent.orElse("Unknown"));
```

Output:

```text
Danish
Unknown
```

#### 9. `orElseGet(supplier)` - computes a fallback only when empty

```java
Optional<String> present = Optional.of("Danish");
Optional<String> absent = Optional.empty();

System.out.println(present.orElseGet(() -> "Generated Default"));
System.out.println(absent.orElseGet(() -> "Generated Default"));
```

Output:

```text
Danish
Generated Default
```

#### 10. `orElseThrow()` - throws if empty

```java
import java.util.NoSuchElementException;

Optional<String> present = Optional.of("Danish");
Optional<String> absent = Optional.empty();

System.out.println(present.orElseThrow());

try {
    absent.orElseThrow();
} catch (NoSuchElementException e) {
    System.out.println("NoSuchElementException thrown");
}

// With a custom exception
try {
    absent.orElseThrow(() -> new IllegalStateException("Name is missing"));
} catch (IllegalStateException e) {
    System.out.println(e.getMessage());
}
```

Output:

```text
Danish
NoSuchElementException thrown
Name is missing
```

### `orElse()` vs `orElseGet()` Demonstration

```java
import java.util.Optional;

public class OrElseVsOrElseGet {
    static String expensiveDefault() {
        System.out.println("expensiveDefault() called");
        return "Default";
    }

    public static void main(String[] args) {
        Optional<String> value = Optional.of("Present");

        System.out.println("--- orElse ---");
        System.out.println(value.orElse(expensiveDefault()));

        System.out.println("--- orElseGet ---");
        System.out.println(value.orElseGet(() -> expensiveDefault()));
    }
}
```

Output:

```text
--- orElse ---
expensiveDefault() called
Present
--- orElseGet ---
Present
```

### Complete Runnable Program

```java
import java.util.NoSuchElementException;
import java.util.Optional;

public class OptionalMethodsDemo {
    public static void main(String[] args) {

        // of
        Optional<String> of = Optional.of("Danish");
        System.out.println("of:           " + of);

        // ofNullable
        Optional<String> nullable = Optional.ofNullable(null);
        System.out.println("ofNullable:   " + nullable);

        // empty
        Optional<String> empty = Optional.empty();
        System.out.println("empty:        " + empty);

        // isPresent
        System.out.println("isPresent:    " + of.isPresent() + ", " + empty.isPresent());

        // ifPresent
        System.out.print("ifPresent:    ");
        of.ifPresent(v -> System.out.println("Hello, " + v));

        // map
        System.out.println("map:          " + of.map(String::toUpperCase));

        // flatMap
        System.out.println("flatMap:      " + of.flatMap(v -> Optional.of(v.length())));

        // orElse
        System.out.println("orElse:       " + empty.orElse("Unknown"));

        // orElseGet
        System.out.println("orElseGet:    " + empty.orElseGet(() -> "Generated"));

        // orElseThrow
        try {
            empty.orElseThrow();
        } catch (NoSuchElementException e) {
            System.out.println("orElseThrow:  NoSuchElementException thrown");
        }
    }
}
```

Output:

```text
of:           Optional[Danish]
ofNullable:   Optional.empty
empty:        Optional.empty
isPresent:    true, false
ifPresent:    Hello, Danish
map:          Optional[DANISH]
flatMap:      Optional[6]
orElse:       Unknown
orElseGet:    Generated
orElseThrow:  NoSuchElementException thrown
```

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

### Examples of Each Class

#### 1. `LocalDate` - date without time or time zone

```java
import java.time.DayOfWeek;
import java.time.LocalDate;

LocalDate date = LocalDate.of(2026, 10, 22);

System.out.println(date);
System.out.println(date.getYear());
System.out.println(date.getMonth());
System.out.println(date.getDayOfWeek());
System.out.println(date.plusDays(10));
System.out.println(date.minusMonths(1));
System.out.println(date.isLeapYear());
System.out.println(date.isBefore(LocalDate.of(2027, 1, 1)));
```

Output:

```text
2026-10-22
2026
OCTOBER
THURSDAY
2026-11-01
2026-09-22
false
true
```

#### 2. `LocalTime` - time without date or time zone

```java
import java.time.LocalTime;

LocalTime time = LocalTime.of(14, 30, 15);

System.out.println(time);
System.out.println(time.getHour());
System.out.println(time.getMinute());
System.out.println(time.plusHours(3));
System.out.println(time.minusMinutes(45));
System.out.println(time.isAfter(LocalTime.NOON));
```

Output:

```text
14:30:15
14
30
17:30:15
13:45:15
true
```

#### 3. `LocalDateTime` - date and time without a time zone

```java
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;

LocalDateTime dateTime = LocalDateTime.of(2026, 10, 22, 16, 30);

System.out.println(dateTime);
System.out.println(dateTime.plusDays(1).plusHours(2));
System.out.println(dateTime.toLocalDate());
System.out.println(dateTime.toLocalTime());

// Combining a LocalDate and LocalTime
LocalDateTime combined = LocalDateTime.of(
        LocalDate.of(2027, 1, 1),
        LocalTime.of(9, 0));
System.out.println(combined);
```

Output:

```text
2026-10-22T16:30
2026-10-23T18:30
2026-10-22
16:30
2027-01-01T09:00
```

#### 4. `ZonedDateTime` - date and time with a time zone

```java
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;

ZonedDateTime kolkata = ZonedDateTime.of(
        LocalDateTime.of(2026, 10, 22, 16, 30),
        ZoneId.of("Asia/Kolkata"));

ZonedDateTime newYork = kolkata.withZoneSameInstant(ZoneId.of("America/New_York"));
ZonedDateTime tokyo = kolkata.withZoneSameInstant(ZoneId.of("Asia/Tokyo"));

System.out.println(kolkata);
System.out.println(newYork);
System.out.println(tokyo);
```

Output:

```text
2026-10-22T16:30+05:30[Asia/Kolkata]
2026-10-22T07:00-04:00[America/New_York]
2026-10-22T20:00+09:00[Asia/Tokyo]
```

#### 5. `Instant` - point on the UTC timeline

```java
import java.time.Instant;

Instant instant = Instant.parse("2026-10-22T11:00:00Z");

System.out.println(instant);
System.out.println(instant.getEpochSecond());
System.out.println(instant.plusSeconds(3600));
System.out.println(Instant.ofEpochSecond(0));

// Current instant (varies on each run)
Instant now = Instant.now();
System.out.println(now.isAfter(instant));
```

Output:

```text
2026-10-22T11:00:00Z
1792666800
2026-10-22T12:00:00Z
1970-01-01T00:00:00Z
true
```

The last line depends on when you run it; it prints `true` once the current time is later than the parsed instant.

#### 6. `Period` - date-based amount of time

```java
import java.time.LocalDate;
import java.time.Period;

LocalDate start = LocalDate.of(1995, 5, 20);
LocalDate end = LocalDate.of(2026, 10, 22);

Period age = Period.between(start, end);

System.out.println(age);
System.out.println(age.getYears() + " years, "
        + age.getMonths() + " months, "
        + age.getDays() + " days");

Period custom = Period.of(1, 2, 3);
System.out.println(LocalDate.of(2026, 1, 1).plus(custom));
```

Output:

```text
P31Y5M2D
31 years, 5 months, 2 days
2027-03-04
```

#### 7. `Duration` - time-based amount of time

```java
import java.time.Duration;
import java.time.LocalTime;

LocalTime start = LocalTime.of(9, 0);
LocalTime end = LocalTime.of(17, 45);

Duration duration = Duration.between(start, end);

System.out.println(duration);
System.out.println(duration.toHours());
System.out.println(duration.toMinutes());

Duration custom = Duration.ofMinutes(90);
System.out.println(custom);
System.out.println(LocalTime.of(10, 0).plus(custom));
```

Output:

```text
PT8H45M
8
525
PT1H30M
11:30
```

#### 8. `DateTimeFormatter` - formats and parses date/time values

```java
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

LocalDateTime dateTime = LocalDateTime.of(2026, 10, 22, 16, 30);

DateTimeFormatter custom = DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm");
DateTimeFormatter readable = DateTimeFormatter.ofPattern("EEEE, MMM d, yyyy hh:mm a");

// Formatting
System.out.println(dateTime.format(custom));
System.out.println(dateTime.format(readable));
System.out.println(dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));

// Parsing
LocalDate parsed = LocalDate.parse("22/10/2026",
        DateTimeFormatter.ofPattern("dd/MM/yyyy"));
System.out.println(parsed);
```

Output:

```text
22-10-2026 16:30
Thursday, Oct 22, 2026 04:30 PM
2026-10-22T16:30:00
2026-10-22
```

The `a` marker output (`PM`) can vary by locale; pass `Locale.ENGLISH` to `ofPattern` for consistent results.

### Complete Runnable Program

```java
import java.time.Duration;
import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.Period;
import java.time.ZoneId;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;

public class DateTimeMethodsDemo {
    public static void main(String[] args) {

        // LocalDate
        LocalDate date = LocalDate.of(2026, 10, 22);
        System.out.println("LocalDate:         " + date.plusDays(10));

        // LocalTime
        LocalTime time = LocalTime.of(14, 30);
        System.out.println("LocalTime:         " + time.plusHours(3));

        // LocalDateTime
        LocalDateTime dateTime = LocalDateTime.of(2026, 10, 22, 16, 30);
        System.out.println("LocalDateTime:     " + dateTime.plusDays(1));

        // ZonedDateTime
        ZonedDateTime kolkata = ZonedDateTime.of(dateTime, ZoneId.of("Asia/Kolkata"));
        System.out.println("ZonedDateTime:     "
                + kolkata.withZoneSameInstant(ZoneId.of("America/New_York")));

        // Instant
        Instant instant = Instant.parse("2026-10-22T11:00:00Z");
        System.out.println("Instant:           " + instant.plusSeconds(3600));

        // Period
        Period period = Period.between(LocalDate.of(1995, 5, 20), date);
        System.out.println("Period:            " + period);

        // Duration
        Duration duration = Duration.between(LocalTime.of(9, 0), LocalTime.of(17, 45));
        System.out.println("Duration:          " + duration);

        // DateTimeFormatter
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm");
        System.out.println("DateTimeFormatter: " + dateTime.format(formatter));
    }
}
```

Output:

```text
LocalDate:         2026-11-01
LocalTime:         17:30
LocalDateTime:     2026-10-23T16:30
ZonedDateTime:     2026-10-22T07:00-04:00[America/New_York]
Instant:           2026-10-22T12:00:00Z
Period:            P31Y5M2D
Duration:          PT8H45M
DateTimeFormatter: 22-10-2026 16:30
```

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

### Sample Data Used in the Examples

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

class Employee {
    private final String name;
    private final String department;
    private final int salary;

    Employee(String name, String department, int salary) {
        this.name = name;
        this.department = department;
        this.salary = salary;
    }

    public String getName()       { return name; }
    public String getDepartment() { return department; }
    public int getSalary()        { return salary; }
}

List<String> names = Arrays.asList("John", "Sham", "Danish");
List<Integer> numbers = Arrays.asList(5, 3, 8, 3, 1, 8, 9);

List<Employee> employees = Arrays.asList(
        new Employee("A", "IT", 50000),
        new Employee("B", "HR", 40000),
        new Employee("C", "IT", 60000)
);
```

### Examples of Each Collector

#### 1. `toList()` - collects elements into a list

```java
List<String> list = names.stream()
        .collect(Collectors.toList());

System.out.println(list);
```

Output:

```text
[John, Sham, Danish]
```

#### 2. `toSet()` - collects elements into a set (duplicates removed)

```java
import java.util.Set;

Set<Integer> set = numbers.stream()
        .collect(Collectors.toSet());

System.out.println(set);
```

Output:

```text
[1, 3, 5, 8, 9]
```

A `Set` does not guarantee order in general. The sorted output here is just how `HashSet` happens to arrange these small integers.

#### 3. `toMap()` - collects elements into a map

```java
import java.util.Map;
import java.util.TreeMap;

// Basic: key function and value function
Map<String, Integer> salaryByName = employees.stream()
        .collect(Collectors.toMap(
                Employee::getName,
                Employee::getSalary));

System.out.println(salaryByName);

// With a merge function (handles duplicate keys) and a map factory
List<String> words = Arrays.asList("apple", "avocado", "banana");

Map<Character, String> byFirstLetter = words.stream()
        .collect(Collectors.toMap(
                word -> word.charAt(0),
                word -> word,
                (first, second) -> first + "|" + second,
                TreeMap::new));

System.out.println(byFirstLetter);

// Without a merge function, duplicate keys throw an exception
try {
    words.stream()
            .collect(Collectors.toMap(word -> word.charAt(0), word -> word));
} catch (IllegalStateException e) {
    System.out.println("IllegalStateException: duplicate key");
}
```

Output:

```text
{A=50000, B=40000, C=60000}
{a=apple|avocado, b=banana}
IllegalStateException: duplicate key
```

#### 4. `joining()` - joins strings

```java
// No delimiter
System.out.println(names.stream().collect(Collectors.joining()));

// With delimiter
System.out.println(names.stream().collect(Collectors.joining(", ")));

// With delimiter, prefix, and suffix
System.out.println(names.stream().collect(Collectors.joining(", ", "[", "]")));
```

Output:

```text
JohnShamDanish
John, Sham, Danish
[John, Sham, Danish]
```

`joining()` works only on streams of `CharSequence` (such as `String`).

#### 5. `groupingBy()` - groups elements by a classification function

```java
import java.util.Map;

// Basic grouping: names by length
Map<Integer, List<String>> byLength = names.stream()
        .collect(Collectors.groupingBy(String::length));

System.out.println(byLength);

// With a downstream collector: count employees per department
Map<String, Long> countByDepartment = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()));

System.out.println(countByDepartment);

// With a downstream collector: employee names per department
Map<String, List<String>> namesByDepartment = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.mapping(Employee::getName, Collectors.toList())));

System.out.println(namesByDepartment);
```

Output:

```text
{4=[John, Sham], 6=[Danish]}
{HR=1, IT=2}
{HR=[B], IT=[A, C]}
```

#### 6. `partitioningBy()` - splits elements into `true` and `false` groups

```java
Map<Boolean, List<Integer>> evenOdd = numbers.stream()
        .collect(Collectors.partitioningBy(n -> n % 2 == 0));

System.out.println(evenOdd);
System.out.println("Even: " + evenOdd.get(true));
System.out.println("Odd:  " + evenOdd.get(false));

// With a downstream collector
Map<Boolean, Long> counts = numbers.stream()
        .collect(Collectors.partitioningBy(
                n -> n > 4,
                Collectors.counting()));

System.out.println(counts);
```

Output:

```text
{false=[5, 3, 3, 1, 9], true=[8, 8]}
Even: [8, 8]
Odd:  [5, 3, 3, 1, 9]
{false=3, true=4}
```

`partitioningBy()` always produces both the `false` and `true` keys, even if one group is empty.

#### 7. `counting()` - counts elements

```java
long total = names.stream()
        .collect(Collectors.counting());

System.out.println(total);

long longNames = names.stream()
        .filter(name -> name.length() > 4)
        .collect(Collectors.counting());

System.out.println(longNames);
```

Output:

```text
3
1
```

`counting()` returns a `Long`. It is most useful as a downstream collector, as in `groupingBy()` above. For a plain count, `stream.count()` is simpler.

#### 8. `summarizingInt()` - computes integer summary statistics

```java
import java.util.IntSummaryStatistics;

IntSummaryStatistics stats = employees.stream()
        .collect(Collectors.summarizingInt(Employee::getSalary));

System.out.println(stats);
System.out.println("Count:   " + stats.getCount());
System.out.println("Sum:     " + stats.getSum());
System.out.println("Min:     " + stats.getMin());
System.out.println("Max:     " + stats.getMax());
System.out.println("Average: " + stats.getAverage());
```

Output:

```text
IntSummaryStatistics{count=3, sum=150000, min=40000, average=50000.000000, max=60000}
Count:   3
Sum:     150000
Min:     40000
Max:     60000
Average: 50000.0
```

#### 9. `averagingInt()` - computes an average

```java
double averageSalary = employees.stream()
        .collect(Collectors.averagingInt(Employee::getSalary));

System.out.println(averageSalary);

// As a downstream collector: average salary per department
Map<String, Double> averageByDepartment = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingInt(Employee::getSalary)));

System.out.println(averageByDepartment);
```

Output:

```text
50000.0
{HR=40000.0, IT=55000.0}
```

### Complete Runnable Program

```java
import java.util.Arrays;
import java.util.IntSummaryStatistics;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.TreeMap;
import java.util.stream.Collectors;

class Employee {
    private final String name;
    private final String department;
    private final int salary;

    Employee(String name, String department, int salary) {
        this.name = name;
        this.department = department;
        this.salary = salary;
    }

    public String getName()       { return name; }
    public String getDepartment() { return department; }
    public int getSalary()        { return salary; }
}

public class CollectorsMethodsDemo {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("John", "Sham", "Danish");
        List<Integer> numbers = Arrays.asList(5, 3, 8, 3, 1, 8, 9);
        List<Employee> employees = Arrays.asList(
                new Employee("A", "IT", 50000),
                new Employee("B", "HR", 40000),
                new Employee("C", "IT", 60000)
        );

        // toList
        List<String> list = names.stream().collect(Collectors.toList());
        System.out.println("toList:         " + list);

        // toSet
        Set<Integer> set = numbers.stream().collect(Collectors.toSet());
        System.out.println("toSet:          " + set);

        // toMap
        Map<String, Integer> salaryByName = employees.stream()
                .collect(Collectors.toMap(Employee::getName, Employee::getSalary));
        System.out.println("toMap:          " + salaryByName);

        List<String> words = Arrays.asList("apple", "avocado", "banana");
        Map<Character, String> byFirstLetter = words.stream()
                .collect(Collectors.toMap(
                        word -> word.charAt(0),
                        word -> word,
                        (first, second) -> first + "|" + second,
                        TreeMap::new));
        System.out.println("toMap (merge):  " + byFirstLetter);

        // joining
        System.out.println("joining:        " + names.stream()
                .collect(Collectors.joining(", ", "[", "]")));

        // groupingBy
        System.out.println("groupingBy:     " + names.stream()
                .collect(Collectors.groupingBy(String::length)));

        System.out.println("grouping+names: " + employees.stream()
                .collect(Collectors.groupingBy(
                        Employee::getDepartment,
                        Collectors.mapping(Employee::getName, Collectors.toList()))));

        // partitioningBy
        System.out.println("partitioningBy: " + numbers.stream()
                .collect(Collectors.partitioningBy(n -> n % 2 == 0)));

        // counting
        System.out.println("counting:       " + names.stream()
                .collect(Collectors.counting()));

        System.out.println("grouping+count: " + employees.stream()
                .collect(Collectors.groupingBy(
                        Employee::getDepartment,
                        Collectors.counting())));

        // summarizingInt
        IntSummaryStatistics stats = employees.stream()
                .collect(Collectors.summarizingInt(Employee::getSalary));
        System.out.println("summarizingInt: " + stats);

        // averagingInt
        System.out.println("averagingInt:   " + employees.stream()
                .collect(Collectors.averagingInt(Employee::getSalary)));

        System.out.println("avg by dept:    " + employees.stream()
                .collect(Collectors.groupingBy(
                        Employee::getDepartment,
                        Collectors.averagingInt(Employee::getSalary))));
    }
}
```

Output:

```text
toList:         [John, Sham, Danish]
toSet:          [1, 3, 5, 8, 9]
toMap:          {A=50000, B=40000, C=60000}
toMap (merge):  {a=apple|avocado, b=banana}
joining:        [John, Sham, Danish]
groupingBy:     {4=[John, Sham], 6=[Danish]}
grouping+names: {HR=[B], IT=[A, C]}
partitioningBy: {false=[5, 3, 3, 1, 9], true=[8, 8]}
counting:       3
grouping+count: {HR=1, IT=2}
summarizingInt: IntSummaryStatistics{count=3, sum=150000, min=40000, average=50000.000000, max=60000}
averagingInt:   50000.0
avg by dept:    {HR=40000.0, IT=55000.0}
```

**Interview point:** `toMap()` throws `IllegalStateException` on duplicate keys unless you supply a merge function, and `groupingBy()` and `toMap()` return a `HashMap` by default, so key order is not guaranteed. Pass a map factory such as `TreeMap::new` or `LinkedHashMap::new` when you need a predictable order.

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

### Q9. What is a lambda expression?

A lambda expression is a concise way to represent an anonymous function. It consists of parameters, an arrow (`->`), and a body. It can be used wherever a functional interface is expected, and it removes much of the boilerplate of anonymous inner classes.

### Q10. What is a functional interface?

A functional interface is an interface with exactly one abstract method. It can be the target type of a lambda expression or method reference. The `@FunctionalInterface` annotation is optional, but it makes the compiler enforce the single-abstract-method rule.

### Q11. What are the common built-in functional interfaces?

`Predicate<T>` tests a condition, `Function<T, R>` transforms a value, `Consumer<T>` consumes a value, `Supplier<T>` supplies a value, `UnaryOperator<T>` transforms a value into the same type, and `BinaryOperator<T>` combines two values of the same type. They are in the `java.util.function` package.

### Q12. What is a method reference, and what are its types?

A method reference is a shorthand for a lambda expression that calls an existing method. There are four types:

- Reference to a static method: `ClassName::staticMethod`
- Reference to an instance method of a particular object: `object::instanceMethod`
- Reference to an instance method of an arbitrary object of a particular type: `ClassName::instanceMethod`
- Reference to a constructor: `ClassName::new`

### Q13. What are default methods in interfaces, and why were they introduced?

Default methods are interface methods with a body, declared with the `default` keyword. They were introduced so that interfaces such as `Collection` could gain new methods (for example `stream()` and `forEach()`) without breaking existing implementing classes.

### Q14. What are static methods in interfaces?

Static methods in interfaces belong to the interface itself and are called using the interface name. They are not inherited by implementing classes. They are typically used for utility or factory methods related to the interface.

### Q15. What happens if two interfaces provide the same default method and a class implements both?

The code does not compile unless the class resolves the conflict. The class must override the method, and it can call a specific interface's version using `InterfaceName.super.methodName()`.

### Q16. What is the difference between intermediate and terminal operations in streams?

Intermediate operations (such as `filter()`, `map()`, `sorted()`) return another stream and are lazy, meaning they do not execute until a terminal operation is invoked. Terminal operations (such as `collect()`, `forEach()`, `count()`, `reduce()`) trigger the pipeline and produce a result or side effect.

### Q17. What does lazy evaluation mean in streams?

Intermediate operations are not executed when they are declared. The pipeline runs only when a terminal operation is called, which allows optimizations such as short-circuiting and fusing multiple operations into a single pass over the data.

### Q18. What is the difference between `Collection` and `Stream`?

A collection stores and manages data, and it can be traversed multiple times. A stream does not store data. It processes elements from a source through a pipeline, uses internal iteration, and is consumed after a terminal operation.

### Q19. What is the difference between `findFirst()` and `findAny()`?

`findFirst()` returns the first element in encounter order. `findAny()` may return any element, which gives it more freedom in parallel streams. For sequential streams, both usually return the first element.

### Q20. What is the difference between `forEach()` and `forEachOrdered()`?

In sequential streams they behave the same. In parallel streams, `forEach()` does not guarantee encounter order, while `forEachOrdered()` preserves it, usually at some performance cost.

### Q21. What is the difference between `peek()` and `map()`?

`map()` transforms elements and produces a new stream of the results. `peek()` performs an action on each element as it passes through and returns the same elements, mainly for debugging. It should not be relied on for core logic.

### Q22. What is the difference between `reduce()` and `collect()`?

`reduce()` combines elements into a single immutable-style result by repeatedly applying an operation. `collect()` is a mutable reduction that accumulates elements into a container such as a list, set, or map. For building collections, `collect()` is the appropriate choice.

### Q23. What is the difference between `groupingBy()` and `partitioningBy()`?

`groupingBy()` groups elements by a classifier function into any number of groups. `partitioningBy()` uses a predicate and always produces exactly two groups, `true` and `false`.

### Q24. What happens when `Collectors.toMap()` encounters duplicate keys?

It throws `IllegalStateException`. To handle duplicates, supply a merge function as the third argument, for example `(oldValue, newValue) -> oldValue`.

### Q25. What is `Optional`, and why was it introduced?

`Optional<T>` is a container that may or may not hold a non-null value. It makes the possible absence of a value explicit in a method's return type, which encourages callers to handle that case and reduces unexpected `NullPointerException`s.

### Q26. What is the difference between `Optional.of()` and `Optional.ofNullable()`?

`Optional.of(value)` throws `NullPointerException` if the value is null. `Optional.ofNullable(value)` returns an empty Optional when the value is null.

### Q27. Should `Optional` be used for fields and method parameters?

Generally no. It was designed mainly as a return type. It is not serializable, adds object overhead, and makes parameter lists harder to read. Overloaded methods or plain null checks are usually preferred for parameters.

### Q28. What is the difference between `Optional.map()` and `Optional.flatMap()`?

`map()` applies a function and wraps the result in an Optional. `flatMap()` is used when the function itself returns an Optional, and it avoids nested results such as `Optional<Optional<T>>`.

### Q29. What is the difference between `LocalDate`, `LocalDateTime`, and `ZonedDateTime`?

`LocalDate` holds only a date, `LocalDateTime` holds a date and time without a time zone, and `ZonedDateTime` holds a date, time, and time zone (with offset). Use `ZonedDateTime` or `Instant` when time-zone handling matters.

### Q30. What is the difference between `Period` and `Duration`?

`Period` measures date-based amounts in years, months, and days. `Duration` measures time-based amounts in hours, minutes, seconds, and nanoseconds.

### Q31. Why is `java.time` considered better than `java.util.Date` and `Calendar`?

The `java.time` classes are immutable and thread-safe, have a clearer and more consistent API, use sensible month numbering (1 to 12) and year handling, and offer separate types for dates, times, and time zones. `Date` and `Calendar` are mutable, not thread-safe, and have confusing design choices.

### Q32. What is `DateTimeFormatter`, and is it thread-safe?

`DateTimeFormatter` formats and parses date/time objects. Unlike the legacy `SimpleDateFormat`, it is immutable and thread-safe, so it can be shared across threads.

### Q33. What is the difference between `Instant` and `LocalDateTime`?

`Instant` represents a point on the UTC timeline, typically used for machine timestamps. `LocalDateTime` represents a date and time as people see it on a calendar and clock, without any time-zone context.

### Q34. What is `CompletableFuture`?

`CompletableFuture` is a class introduced in Java 8 for asynchronous programming. It supports non-blocking composition of tasks through methods such as `supplyAsync()`, `thenApply()`, `thenCompose()`, and `thenCombine()`, and it allows explicit completion and exception handling.

### Q35. What is the difference between `Future` and `CompletableFuture`?

`Future` only allows blocking retrieval with `get()` and basic cancellation checks. `CompletableFuture` adds callbacks, chaining, combining multiple futures, explicit completion, and built-in exception handling.

### Q36. What is a parallel stream, and when should it be used?

A parallel stream splits its source into parts that are processed concurrently, typically using the common `ForkJoinPool`. It is useful for large datasets with CPU-intensive, stateless, independent operations. It is a poor fit for small data, I/O-bound work, or operations that depend on shared mutable state.

### Q37. What are the requirements for safe use of parallel streams?

Operations should be stateless, non-interfering, and associative where applicable (such as in `reduce()`). Avoid modifying shared mutable state from within stream operations, because it causes race conditions and unpredictable results.

### Q38. Can you modify the source collection while a stream is processing it?

No. Modifying the source during a terminal operation can cause `ConcurrentModificationException` or undefined behavior. Stream operations should be non-interfering with the source.

### Q39. Which new methods were added to `Map` in Java 8?

Notable additions include `getOrDefault()`, `putIfAbsent()`, `computeIfAbsent()`, `computeIfPresent()`, `compute()`, `merge()`, `forEach()`, `replaceAll()`, and `remove(key, value)`.

### Q40. How did `HashMap` change in Java 8?

When a bucket accumulates many collisions (the default threshold is 8 entries, and the table must also be large enough), the linked list is converted to a balanced tree. This improves worst-case lookup time from O(n) to O(log n).

### Q41. What are `StringJoiner` and `String.join()`?

`StringJoiner` builds a delimited string with an optional prefix and suffix. `String.join()` is a convenient static method that joins elements with a delimiter.

### Q42. What are repeating annotations and type annotations?

Repeating annotations let the same annotation be applied multiple times to one declaration, using `@Repeatable`. Type annotations let annotations be applied wherever a type is used, such as `List<@NonNull String>`, which supports stronger static analysis tools.

### Q43. What is the Nashorn engine?

Nashorn was the JavaScript engine introduced in Java 8 to run JavaScript code on the JVM. It replaced the older Rhino engine. It was deprecated in Java 11 and removed in Java 15.

### Q44. What was removed from the JVM in Java 8 regarding memory?

The Permanent Generation (PermGen) was removed and replaced by Metaspace, which uses native memory and grows dynamically by default, reducing the chance of `OutOfMemoryError` from PermGen limits.

### Q45. Can a lambda expression modify local variables from its enclosing scope?

A lambda can use local variables only if they are final or effectively final. It can, however, modify instance fields and static fields.

### Q46. What is the difference between a lambda expression and an anonymous inner class?

- A lambda works only with functional interfaces, while an anonymous class can implement any interface or extend a class.
- In a lambda, `this` refers to the enclosing class instance. In an anonymous class, `this` refers to the anonymous class instance.
- Lambdas are generally more concise and are compiled using `invokedynamic`, while anonymous classes generate separate class files.

### Q47. Can an interface in Java 8 have a constructor or instance variables?

No. Interfaces cannot have constructors or instance state. Fields declared in an interface are implicitly `public static final` constants.

### Q48. What are `Comparator` enhancements in Java 8?

`Comparator` gained static and default methods such as `comparing()`, `comparingInt()`, `thenComparing()`, `reversed()`, `naturalOrder()`, `reverseOrder()`, `nullsFirst()`, and `nullsLast()`, which make multi-level sorting concise and readable.

### Q49. What is the difference between `Stream.of()` and `Arrays.stream()`?

`Stream.of(T... values)` creates a stream from the given elements and, for a single array argument of a reference type, treats it as the elements. `Arrays.stream()` creates a stream from an array and also supports ranges and primitive arrays, returning primitive streams such as `IntStream` where appropriate.

### Q50. What are primitive streams, and why are they useful?

`IntStream`, `LongStream`, and `DoubleStream` avoid boxing overhead and provide numeric operations such as `sum()`, `average()`, `range()`, and `summaryStatistics()`. They improve performance and convenience for numeric processing.

---

## 18. One-Line Interview Answer

Java 8 introduced lambda expressions, functional interfaces, the Stream API, default and static interface methods, method references, Optional, the modern Date and Time API, collectors, parallel array sorting, CompletableFuture, and JDBC improvements, making Java more expressive and improving functional-style and concurrent programming.