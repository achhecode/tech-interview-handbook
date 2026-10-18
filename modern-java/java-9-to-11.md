---
title: Java 9 to 11
parent: Modern Java (8 to 25)
nav_order: 2
---

# Java 9 to Java 11 Features

Java 9, Java 10, and Java 11 introduced important improvements to the Java language, collections, streams, JVM, and standard libraries. These versions are especially important for interviews because they introduced the **Java Platform Module System, collection factory methods, `var`, the standardized HTTP Client, and new String APIs**.

## 1. Overview of Java 9, 10, and 11

| Version | Key features |
|---|---|
| Java 9 | Module System, JShell, factory methods for collections, private interface methods, Stream API improvements, try-with-resources improvements |
| Java 10 | Local-variable type inference with `var`, `List.copyOf()` and related APIs, application class-data sharing improvements |
| Java 11 | Standard HTTP Client, new String methods, `Files.readString()` and `writeString()`, `Optional.isEmpty()`, single-file source execution, standard Flight Recorder |

**Important:** Java 9 introduced modules, Java 10 introduced local-variable `var`, and Java 11 standardized the HTTP Client API.

---

## 2. Java 9 Features

### 2.1 Java Platform Module System (JPMS)

Before Java 9, Java applications commonly organized code using packages and JAR files. Java 9 introduced the **Java Platform Module System**, which lets developers group related packages into modules and explicitly declare dependencies.

A module can specify:
- Which other modules it requires.
- Which packages it exports for use by other modules.
- Which services it provides or consumes.

The module descriptor is named `module-info.java`.

**Example:**

```java
module com.example.app {
    requires java.sql;
    exports com.example.app.api;
}
```

- `module com.example.app` declares the module.
- `requires java.sql` declares a dependency on the `java.sql` module.
- `exports com.example.app.api` makes that package accessible to other modules.

**Benefits:**
- Stronger encapsulation.
- Explicit dependencies.
- More reliable application configuration.
- Support for creating smaller custom runtime images using tools such as `jlink`.

**Interview point:** A Java package groups classes, while a Java module groups packages and declares dependencies and access boundaries.

### 2.2 JShell

JShell is an interactive Java REPL (Read-Eval-Print Loop). It allows developers to execute Java expressions and statements without creating a complete class and `main()` method.

```text
jshell> int a = 10;
a ==> 10

jshell> int b = 20;
b ==> 20

jshell> System.out.println(a + b);
30
```

**Use cases:** Quickly testing expressions, exploring APIs, and demonstrating Java concepts.

### 2.3 Factory Methods for Immutable Collections

Java 9 introduced convenient factory methods for creating unmodifiable collections.

```java
import java.util.List;
import java.util.Set;
import java.util.Map;

public class CollectionFactoryExample {
    public static void main(String[] args) {
        List<String> names = List.of("Aman", "Danish", "Ravi");
        Set<Integer> numbers = Set.of(10, 20, 30);
        Map<Integer, String> employees = Map.of(
            101, "Aman",
            102, "Danish"
        );

        System.out.println(names);
        System.out.println(numbers);
        System.out.println(employees);
    }
}
```

**Output:**

```text
[Aman, Danish, Ravi]
[10, 20, 30]
{101=Aman, 102=Danish}
```

Important properties:

- The returned collections are unmodifiable.
- `List.of()`, `Set.of()`, and `Map.of()` reject `null` elements, keys, and values.
- `Set.of()` rejects duplicate elements.
- `Map.of()` rejects duplicate keys.
- Calling mutation methods such as `add()`, `remove()`, or `put()` throws `UnsupportedOperationException`.

For larger maps, use `Map.ofEntries()`:

```java
Map<Integer, String> employees = Map.ofEntries(
    Map.entry(101, "Aman"),
    Map.entry(102, "Danish"),
    Map.entry(103, "Ravi")
);
```

**Interview point:** These factory methods return unmodifiable collections, not deeply immutable object graphs. If an element is mutable, that element can still be changed.

### 2.4 Private Methods in Interfaces

Before Java 9, interfaces could contain abstract methods, default methods, and static methods, but not private methods.

Java 9 introduced private interface methods to share implementation code between default or static methods.

```java
interface Notification {
    default void sendEmail() {
        validate();
        System.out.println("Sending email");
    }

    default void sendSms() {
        validate();
        System.out.println("Sending SMS");
    }

    private void validate() {
        System.out.println("Validating notification");
    }
}
```

**Why use private interface methods?**

They prevent duplicated implementation inside an interface. They are internal helpers and cannot be called directly by implementing classes.

### 2.5 Stream API Improvements

Java 9 added methods such as `takeWhile()`, `dropWhile()`, and `ofNullable()`.

#### `takeWhile()`

Takes elements from the beginning of an ordered stream while the predicate remains true.

```java
import java.util.List;

public class TakeWhileExample {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(2, 4, 6, 7, 8, 10);

        numbers.stream()
               .takeWhile(n -> n % 2 == 0)
               .forEach(System.out::println);
    }
}
```

**Output:**

```text
2
4
6
```

It stops when it encounters `7`; it does not resume at `8`.

#### `dropWhile()`

Drops elements from the beginning while the predicate is true, then emits the remaining elements.

```java
List<Integer> numbers = List.of(2, 4, 6, 7, 8, 10);

numbers.stream()
       .dropWhile(n -> n % 2 == 0)
       .forEach(System.out::println);
```

**Output:**

```text
7
8
10
```

#### `ofNullable()`

Creates a stream containing one element if the value is non-null; otherwise, it creates an empty stream.

```java
import java.util.stream.Stream;

Stream<String> stream = Stream.ofNullable(null);
System.out.println(stream.count());
```

**Output:**

```text
0
```

This is useful when converting potentially null values into streams without manually checking for `null`.

### 2.6 Improved Try-with-Resources

Java 7 introduced try-with-resources. Java 9 enhanced it to allow an already-declared resource variable to be used directly if it is final or effectively final.

**Before Java 9:**

```java
BufferedReader reader = Files.newBufferedReader(path);

try (BufferedReader br = reader) {
    System.out.println(br.readLine());
}
```

**Java 9 and later:**

```java
BufferedReader reader = Files.newBufferedReader(path);

try (reader) {
    System.out.println(reader.readLine());
}
```

The resource is automatically closed when the `try` block exits.

### 2.7 Other Java 9 Improvements

- **Private interface methods:** Reuse logic within interfaces.
- **`Optional.stream()`:** Convert an `Optional` into a stream of zero or one element.
- **`Stream.iterate()` overload:** Generate a stream using an initial value, a predicate, and a next-value function.
- **Diamond operator with anonymous classes:** Improved type inference in certain anonymous-class declarations.
- **Multi-release JARs:** Package version-specific class implementations for different Java releases.

Example of `Optional.stream()`:

```java
import java.util.Optional;

Optional<String> name = Optional.of("Danish");

name.stream().forEach(System.out::println);
```

Output:

```text
Danish
```

---

## 3. Java 10 Features

### 3.1 Local-Variable Type Inference: `var`

Java 10 introduced `var`, which lets the compiler infer the type of a local variable from its initializer.

```java
var name = "Danish";       // String
var age = 28;              // int
var salary = 25000.50;     // double
var active = true;         // boolean
```

The compiler determines the type at compile time. Java remains statically typed.

**Before Java 10:**

```java
String name = "Danish";
List<String> names = new ArrayList<>();
```

**Java 10 and later:**

```java
var name = "Danish";
var names = new ArrayList<String>();
```

The second declaration infers the concrete type `ArrayList<String>`, not `List<String>`.

#### Restrictions of `var`

```java
// Invalid: no initializer
// var name;

// Invalid: cannot initialize with null alone
// var value = null;

// Invalid: var cannot be used for a field
// class Employee { var name = "Danish"; }

// Invalid: var cannot be used as a method return type
// var getName() { return "Danish"; }
```

`var` is supported for local variables with initializers, including suitable loop variables and lambda parameters in later versions (lambda parameter support was added in Java 11).

**Interview point:** `var` is not dynamic typing. Once inferred, the variable has a fixed compile-time type.

### 3.2 Example: `var` with Collections

```java
import java.util.ArrayList;
import java.util.List;

public class VarExample {
    public static void main(String[] args) {
        var names = new ArrayList<String>();

        names.add("Aman");
        names.add("Danish");

        for (var name : names) {
            System.out.println(name);
        }
    }
}
```

**Output:**

```text
Aman
Danish
```

Use `var` when the initializer makes the type clear. Avoid it when it makes code harder to understand.

### 3.3 Collection Copying Methods

Java 10 added `copyOf()` methods to common collection interfaces, including `List`, `Set`, and `Map`.

```java
import java.util.ArrayList;
import java.util.List;

public class CopyOfExample {
    public static void main(String[] args) {
        var original = new ArrayList<>(List.of("Java", "Spring"));

        List<String> copy = List.copyOf(original);

        original.add("Kafka");

        System.out.println(original);
        System.out.println(copy);
    }
}
```

**Output:**

```text
[Java, Spring, Kafka]
[Java, Spring]
```

The result of `List.copyOf()` is unmodifiable and does not reflect later structural changes to the source collection.

Important:
- `List.copyOf()`, `Set.copyOf()`, and `Map.copyOf()` reject null elements, keys, or values.
- They provide unmodifiable collections, not necessarily a new physical collection in every case.
- Copying is shallow: mutable elements are not automatically copied.

### 3.4 Other Java 10 Improvements

- Application Class-Data Sharing (AppCDS) improvements to help startup and memory use.
- Garbage-collector interface improvements.
- Experimental parallel full GC for G1.
- Root certificates in the JDK distribution were updated as part of the release.

For interviews, **`var` is the main Java 10 language feature to remember**.

---

## 4. Java 11 Features

Java 11 is an important Long-Term Support (LTS) release. It introduced useful APIs for HTTP communication, String processing, file operations, and `Optional`.

### 4.1 Standard HTTP Client API

The HTTP Client API was introduced as an incubating API in Java 9 and standardized in Java 11.

It supports:
- HTTP/1.1 and HTTP/2.
- Synchronous requests.
- Asynchronous requests using `CompletableFuture`.
- Request and response body handlers.
- Configurable timeouts and redirect policies.

**Example: synchronous GET request**

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class HttpClientExample {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://example.com"))
                .GET()
                .build();

        HttpResponse<String> response = client.send(
                request,
                HttpResponse.BodyHandlers.ofString()
        );

        System.out.println(response.statusCode());
        System.out.println(response.body());
    }
}
```

The example makes a real network request, so its output depends on the server and network connection.

**How it works:**

1. Create an `HttpClient`.
2. Build an `HttpRequest` with a URI and method.
3. Call `send()` for a synchronous request.
4. Choose a body handler, such as `BodyHandlers.ofString()`.
5. Read the status code and response body.

#### Asynchronous HTTP request

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.concurrent.CompletableFuture;

public class AsyncHttpClientExample {
    public static void main(String[] args) {
        HttpClient client = HttpClient.newHttpClient();

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://example.com"))
                .GET()
                .build();

        CompletableFuture<HttpResponse<String>> future =
                client.sendAsync(
                        request,
                        HttpResponse.BodyHandlers.ofString()
                );

        future.thenApply(HttpResponse::statusCode)
              .thenAccept(System.out::println);
    }
}
```

`sendAsync()` returns a `CompletableFuture`, allowing the application to compose work without blocking the calling thread on the HTTP operation. The example's main method does not wait for completion, so a short-lived program may exit before the result is printed.

**Interview point:** Use `send()` when synchronous blocking behavior is appropriate and `sendAsync()` when composing asynchronous operations is useful.

### 4.2 New String Methods

Java 11 introduced several useful methods in `String`.

| Method | Purpose | Example result |
|---|---|---|
| `isBlank()` | Checks whether a string is empty or contains only whitespace | `"   ".isBlank()` → `true` |
| `strip()` | Removes leading and trailing Unicode whitespace | `"  Java  ".strip()` → `"Java"` |
| `stripLeading()` | Removes leading Unicode whitespace | `"  Java".stripLeading()` → `"Java"` |
| `stripTrailing()` | Removes trailing Unicode whitespace | `"Java  ".stripTrailing()` → `"Java"` |
| `lines()` | Returns a stream of lines | `"A\nB".lines()` → `A`, `B` |
| `repeat(int)` | Repeats a string the specified number of times | `"Hi".repeat(3)` → `"HiHiHi"` |

Example:

```java
public class StringMethodsExample {
    public static void main(String[] args) {
        System.out.println("   ".isBlank());
        System.out.println("  Java  ".strip());
        System.out.println("Hello\nJava".lines().count());
        System.out.println("Java ".repeat(3));
    }
}
```

**Output:**

```text
true
Java
2
Java Java Java 
```

**`strip()` vs `trim()`:** `strip()` uses Unicode-aware whitespace handling, while `trim()` removes leading and trailing characters whose code points are at most U+0020.

### 4.3 `Files.readString()` and `Files.writeString()`

Java 11 added convenient methods for reading and writing whole files as strings.

```java
import java.nio.file.Files;
import java.nio.file.Path;

public class FileStringExample {
    public static void main(String[] args) throws Exception {
        Path path = Path.of("message.txt");

        Files.writeString(path, "Hello Java 11");

        String content = Files.readString(path);

        System.out.println(content);
    }
}
```

**Output:**

```text
Hello Java 11
```

These methods are convenient for small or moderate-sized text files. For large files, consider streaming or buffered processing instead of loading the entire content into memory.

### 4.4 `Optional.isEmpty()`

Java 11 added `isEmpty()` to `Optional`.

```java
import java.util.Optional;

public class OptionalExample {
    public static void main(String[] args) {
        Optional<String> name = Optional.empty();

        System.out.println(name.isEmpty());
        System.out.println(name.isPresent());
    }
}
```

**Output:**

```text
true
false
```

`isEmpty()` is equivalent to `!isPresent()`.

### 4.5 `var` in Lambda Parameters

Java 11 allows `var` in lambda parameter declarations. This is especially useful when parameter annotations are needed.

```java
import java.util.List;

public class LambdaVarExample {
    public static void main(String[] args) {
        List<String> names = List.of("Aman", "Danish", "Ravi");

        names.forEach((var name) -> System.out.println(name));
    }
}
```

**Output:**

```text
Aman
Danish
Ravi
```

Do not mix `var` with explicit types in the same lambda parameter list.

```java
// Invalid: mixing inferred and explicit parameter types
// (var name, String city) -> ...
```

### 4.6 Run a Single-File Java Program

Java 11 allows a source file to be executed directly without explicitly running `javac` first.

Create `Hello.java`:

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello from Java 11");
    }
}
```

Run:

```bash
java Hello.java
```

**Output:**

```text
Hello from Java 11
```

This is useful for quick scripts and small demonstrations. It does not eliminate compilation; the Java launcher compiles the source in memory as part of execution.

### 4.7 Java Flight Recorder (JFR)

Java Flight Recorder became a standard feature in the JDK in Java 11.

It helps developers diagnose performance issues by recording events such as:
- Garbage-collection activity.
- Thread scheduling and contention.
- Allocation activity.
- JVM and application events.

JFR is useful for production troubleshooting because it can collect detailed diagnostic data with relatively low overhead when configured appropriately.

### 4.8 Other Java 11 Improvements

- **Nest-based access control:** Helps JVM access private members among related nested classes.
- **TLS 1.3:** Improved security protocol support.
- **ChaCha20 and Poly1305 cryptographic algorithms:** Additional cryptographic capabilities.
- **ZGC:** Introduced as an experimental low-latency garbage collector in Java 11.
- **Epsilon GC:** An experimental no-op garbage collector intended for specialized use cases.

Note that the garbage collectors' status changed in later Java releases. Do not describe every Java 11 experimental feature as a finalized standard feature.

---

## 5. Important Java 9 vs Java 10 vs Java 11 Differences

| Feature | Introduced |
|---|---|
| Java Platform Module System | Java 9 |
| JShell | Java 9 |
| `List.of()`, `Set.of()`, `Map.of()` | Java 9 |
| Private interface methods | Java 9 |
| `takeWhile()`, `dropWhile()`, `ofNullable()` | Java 9 |
| `var` for local variables | Java 10 |
| `List.copyOf()`, `Set.copyOf()`, `Map.copyOf()` | Java 10 |
| Standard HTTP Client | Java 11 |
| `String.isBlank()`, `strip()`, `lines()`, `repeat()` | Java 11 |
| `Files.readString()` and `Files.writeString()` | Java 11 |
| `Optional.isEmpty()` | Java 11 |
| `var` in lambda parameters | Java 11 |
| Single-file source execution | Java 11 |
| Standard Java Flight Recorder | Java 11 |

---

## 6. Common Java 9–11 Interview Questions

### Q1. What is the Java Module System?

The Java Platform Module System, introduced in Java 9, lets applications declare module dependencies and control package visibility using `module-info.java`.

### Q2. What is the difference between a package and a module?

A package groups related classes. A module groups packages and explicitly declares dependencies and access boundaries.

### Q3. What is JShell?

JShell is Java's interactive REPL. It lets developers execute expressions and statements without creating a full application class.

### Q4. Are collections created by `List.of()` mutable?

No. They are unmodifiable. Attempts to change their structure throw `UnsupportedOperationException`. Their contained objects can still be mutable.

### Q5. What is the difference between `List.of()` and `List.copyOf()`?

`List.of()` creates an unmodifiable list from the supplied elements. `List.copyOf()` creates an unmodifiable list from an existing collection. Both reject null elements, and neither guarantees a deep copy of mutable elements.

### Q6. What is `var` in Java 10?

`var` enables local-variable type inference. The compiler determines the static type from the initializer; it does not make Java dynamically typed.

### Q7. Can `var` be used for instance variables or method return types?

No. Local-variable `var` is not allowed for fields, method parameters, or method return types. Java 11's lambda-specific `var` support is a separate feature.

### Q8. What is the difference between `trim()` and `strip()`?

`trim()` removes leading and trailing characters with code points at most U+0020. `strip()` uses Unicode-aware whitespace handling.

### Q9. What is the difference between `isBlank()` and `isEmpty()`?

`isEmpty()` returns true only when the string has length zero. `isBlank()` also returns true when the string contains only whitespace.

```java
"".isEmpty();      // true
"   ".isEmpty();   // false
"   ".isBlank();   // true
```

### Q10. What is the difference between `send()` and `sendAsync()` in Java's HTTP Client?

`send()` performs a synchronous request and returns the response when available. `sendAsync()` returns a `CompletableFuture` representing the response that will become available asynchronously.

### Q11. What is the advantage of `Files.readString()`?

It simplifies reading an entire text file into a `String` without manually creating a reader and copying lines.

### Q12. What is Java Flight Recorder?

JFR is a JVM diagnostic and profiling facility used to capture runtime events for performance analysis and troubleshooting.

### Q13. Which Java version introduced `var`?

Java 10 introduced local-variable type inference using `var`. Java 11 extended `var` support to lambda parameters.

### Q14. Which Java version introduced the HTTP Client API?

The API was incubated in Java 9 and Java 10 and became a standard API in Java 11.

---

## 7. Quick Revision Checklist

- [ ] Java 9: Module System and `module-info.java`.
- [ ] Java 9: JShell for interactive execution.
- [ ] Java 9: `List.of()`, `Set.of()`, `Map.of()`.
- [ ] Java 9: Private methods in interfaces.
- [ ] Java 9: `takeWhile()`, `dropWhile()`, `ofNullable()`.
- [ ] Java 10: Local-variable type inference using `var`.
- [ ] Java 10: `List.copyOf()`, `Set.copyOf()`, `Map.copyOf()`.
- [ ] Java 11: Standard HTTP Client and `CompletableFuture`.
- [ ] Java 11: `String` methods such as `strip()`, `isBlank()`, `lines()`, and `repeat()`.
- [ ] Java 11: `Files.readString()`, `Files.writeString()`, and `Optional.isEmpty()`.
- [ ] Java 11: `var` in lambda parameters, single-file source execution, and JFR.

## One-Line Interview Answer

**Java 9 introduced modules, JShell, collection factory methods, and private interface methods; Java 10 introduced local-variable type inference with `var`; and Java 11 standardized the HTTP Client and added useful String, file, and Optional APIs.**
