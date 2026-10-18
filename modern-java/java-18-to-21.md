---
title: Java 18 to 21
parent: Modern Java (8 to 25)
nav_order: 4
---

# Java 18 to Java 21 Features

Java 18 through Java 21 introduced improvements to character encoding, concurrency, language syntax, and JVM performance. The most important release for modern enterprise development is **Java 21**, which finalized virtual threads, record patterns, and pattern matching for `switch`.

## 1. Major Features at a Glance

| Java version | Major features |
|---|---|
| Java 18 | UTF-8 by default, Simple Web Server, Code Snippets in Javadoc |
| Java 19 | Virtual threads (preview), record patterns (preview), pattern matching for `switch` (preview) |
| Java 20 | Second preview of virtual threads, improvements to record patterns and pattern matching |
| Java 21 | Virtual threads, record patterns, and pattern matching for `switch` became permanent features |

Java 18, 19, and 20 are feature releases rather than LTS releases. Java 21 is an LTS release.

## 2. Java 18 Features

### 2.1 UTF-8 by Default

Java 18 made UTF-8 the default charset for standard Java APIs that depend on the default charset.

Before Java 18, the default charset could vary depending on the operating system, locale, and configuration. UTF-8 by default improves consistency across environments.

```java
import java.nio.charset.Charset;

public class CharsetExample {
    public static void main(String[] args) {
        System.out.println(Charset.defaultCharset());
    }
}
```

**Typical output in Java 18 and later:**

```text
UTF-8
```

For predictable file handling, explicitly specify the charset when it matters:

```java
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;

public class FileExample {
    public static void main(String[] args) throws Exception {
        Path path = Path.of("message.txt");

        Files.writeString(
            path,
            "Hello, Java",
            StandardCharsets.UTF_8
        );

        String content = Files.readString(
            path,
            StandardCharsets.UTF_8
        );

        System.out.println(content);
    }
}
```

**Interview point:** Java 18 standardized UTF-8 as the default charset; explicitly specifying a charset remains good practice when reading or writing files.

### 2.2 Simple Web Server

Java 18 introduced a minimal static web server through the `jwebserver` command.

Start a server from the current directory:

```bash
jwebserver
```

By default, it serves files from the current directory, typically on port `8000` and bound to the loopback address.

You can configure the binding address, port, and document root using command-line options. Use `jwebserver --help` to see the options supported by your JDK.

This server is useful for:

- Local testing of static HTML, CSS, and JavaScript.
- Quickly sharing static files on a development machine.
- Prototyping without configuring a full web server.

**Important:** It is not intended to replace a production application server or a Spring Boot REST API.

### 2.3 Code Snippets in Javadoc

Java 18 added support for the `@snippet` tag in Javadoc. It helps include readable code examples in API documentation.

```java
/**
 * Calculates the square of a number.
 *
 * {@snippet :
 * int result = 5 * 5;
 * System.out.println(result);
 * }
 */
public class MathExample {
}
```

This makes documentation examples easier to maintain and present.

## 3. Java 19 Features

### 3.1 Virtual Threads (Preview)

Java 19 introduced virtual threads as a preview feature.

Virtual threads are lightweight threads managed by the Java runtime. They are designed to make high-concurrency applications easier to write, especially applications that spend substantial time waiting for network, database, or other blocking I/O operations.

Traditional platform threads are associated with operating-system threads. Creating a very large number of platform threads can consume significant resources.

Virtual threads allow applications to create many more concurrent tasks with lower per-thread overhead.

Java 19 example using preview features:

```java
Thread thread = Thread.ofVirtual().start(() -> {
    System.out.println("Running in a virtual thread");
});

thread.join();
```

This API was still a preview feature in Java 19, so preview-feature compilation and runtime flags were required.

**Important:** Virtual threads became permanent in Java 21.

### 3.2 Record Patterns (Preview)

Java 19 introduced record patterns as a preview feature. Record patterns allow developers to extract components from a record during pattern matching.

```java
record Employee(String name, int age) {}

public class RecordPatternExample {
    public static void main(String[] args) {
        Object value = new Employee("Ali", 30);

        if (value instanceof Employee(String name, int age)) {
            System.out.println(name);
            System.out.println(age);
        }
    }
}
```

The example illustrates the feature's intent, but this syntax was preview-only in Java 19 and evolved in subsequent releases. In Java 21, record patterns became a permanent feature.

### 3.3 Pattern Matching for `switch` (Preview)

Java 19 continued the preview evolution of pattern matching for `switch`.

The feature allows a `switch` to select branches based on an object's type rather than only traditional constants.

```java
Object value = "Java";

String result = switch (value) {
    case Integer number -> "Integer: " + number;
    case String text -> "String: " + text;
    default -> "Other type";
};

System.out.println(result);
```

This example shows the syntax finalized in Java 21. In Java 19, pattern matching for `switch` was a preview feature, and the exact supported syntax changed across preview releases.

## 4. Java 20 Features

Java 20 continued refining features that became permanent in Java 21.

### 4.1 Virtual Threads (Second Preview)

Virtual threads received a second preview in Java 20. The goal remained the same: make highly concurrent applications easier to build without requiring developers to manage large numbers of platform threads manually.

### 4.2 Record Patterns (Second Preview)

Record patterns were refined to improve type inference and their integration with other pattern-matching constructs.

### 4.3 Pattern Matching for `switch` (Fourth Preview)

Java 20 delivered another preview of pattern matching for `switch`, incorporating changes based on developer feedback.

### 4.4 Structured Concurrency (Incubator)

Structured concurrency was available as an incubator API in Java 20. It treats related tasks running in different threads as one unit of work, helping coordinate cancellation, error handling, and task completion.

**Interview distinction:** Structured concurrency was not finalized in Java 20 or Java 21. It remains distinct from the permanent virtual-thread feature.

## 5. Java 21 Features

Java 21 was released in September 2023 and is an LTS release.

The most important features for interviews are virtual threads, record patterns, pattern matching for `switch`, and sequenced collections.

### 5.1 Virtual Threads Became Permanent

Java 21 finalized virtual threads through JEP 444.

```java
public class VirtualThreadExample {
    public static void main(String[] args)
            throws InterruptedException {

        Thread thread = Thread.ofVirtual().start(() -> {
            System.out.println(
                "Hello from " + Thread.currentThread()
            );
        });

        thread.join();
    }
}
```

Virtual threads can also be created using an executor:

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class VirtualThreadExecutorExample {
    public static void main(String[] args) throws Exception {
        try (ExecutorService executor =
                 Executors.newVirtualThreadPerTaskExecutor()) {

            for (int i = 1; i <= 5; i++) {
                int taskId = i;

                executor.submit(() ->
                    System.out.println("Task " + taskId)
                );
            }
        }
    }
}
```

The executor creates a virtual thread for each submitted task. Closing the executor initiates shutdown and waits for submitted tasks to finish.

**Benefits:**

- Supports high numbers of concurrent tasks.
- Simplifies blocking I/O code.
- Reduces the overhead associated with creating many platform threads.
- Works with much existing thread-per-request Java code.

**Important limitations:**

- Virtual threads do not make CPU-intensive work execute faster automatically.
- They do not eliminate database connection limits, API rate limits, or other resource bottlenecks.
- They should generally be created per task rather than pooled like scarce platform threads.
- Applications still need sensible limits on external resources and workload admission.

### 5.2 Record Patterns Became Permanent

Java 21 finalized record patterns through JEP 440.

Record patterns let you test a record's type and extract its components in one expression.

```java
record Point(int x, int y) {}

public class RecordPatternDemo {
    public static void main(String[] args) {
        Object value = new Point(10, 20);

        if (value instanceof Point(int x, int y)) {
            System.out.println("X = " + x);
            System.out.println("Y = " + y);
        }
    }
}
```

**Output:**

```text
X = 10
Y = 20
```

Nested record patterns are also supported:

```java
record Address(String city) {}
record Employee(String name, Address address) {}

public class NestedRecordPatternDemo {
    public static void main(String[] args) {
        Object value =
            new Employee("Ali", new Address("Delhi"));

        if (value instanceof Employee(
                String name,
                Address(String city))) {

            System.out.println(name + " lives in " + city);
        }
    }
}
```

**Output:**

```text
Ali lives in Delhi
```

Record patterns reduce repetitive accessor calls and make data-oriented code more concise.

### 5.3 Pattern Matching for `switch` Became Permanent

Java 21 finalized pattern matching for `switch` through JEP 441.

```java
public class PatternSwitchDemo {
    static String describe(Object value) {
        return switch (value) {
            case Integer number -> "Integer: " + number;
            case String text -> "String: " + text;
            case null -> "Null value";
            default -> "Other type";
        };
    }

    public static void main(String[] args) {
        System.out.println(describe(100));
        System.out.println(describe("Java"));
        System.out.println(describe(null));
    }
}
```

**Output:**

```text
Integer: 100
String: Java
Null value
```

**Advantages:**

- Removes repetitive `instanceof` checks and casts.
- Allows type-based branching.
- Supports more expressive handling of sealed hierarchies.
- Can handle `null` explicitly with a `case null` label.

**Interview point:** Pattern matching for `switch` was a preview feature in Java 17–20 and became permanent in Java 21.

### 5.4 Sequenced Collections

Java 21 introduced new interfaces to represent collections with a defined encounter order and consistent operations at both ends.

Important interfaces include:

- `SequencedCollection`
- `SequencedSet`
- `SequencedMap`

They provide operations such as:

- `getFirst()`
- `getLast()`
- `addFirst()` and `addLast()` where supported
- `reversed()`

Example:

```java
import java.util.List;

public class SequencedCollectionExample {
    public static void main(String[] args) {
        List<String> names =
            new java.util.ArrayList<>(
                List.of("Ali", "Sara", "John")
            );

        System.out.println(names.getFirst());
        System.out.println(names.getLast());
        System.out.println(names.reversed());
    }
}
```

**Output:**

```text
Ali
John
[John, Sara, Ali]
```

The reversed view reflects the underlying list; it does not necessarily create an independent copy.

**Why is this useful?**

Before Java 21, working with the first or last element often required implementation-specific methods or manual indexing. Sequenced collections provide a more consistent API.

### 5.5 String Templates (Preview)

Java 21 introduced string templates as a preview feature. They were designed to combine literal text with embedded expressions through template processors.

The syntax used in Java 21 preview releases was experimental and is not a permanent Java 21 language feature.

For interview preparation, know that **string templates were previewed in Java 21**, but do not treat them as a finalized feature or assume the preview syntax is available in ordinary Java 21 compilation.

### 5.6 Unnamed Patterns and Variables (Preview)

Java 21 introduced unnamed patterns and variables as a preview feature. The underscore (`_`) can represent a value or pattern that does not need to be named or used.

These features reduce unused-variable clutter in code. They were preview features in Java 21, not permanent features in that release.

### 5.7 Other Java 21 Improvements

- Generational ZGC became available as an experimental feature.
- The Foreign Function and Memory API became permanent.
- The Vector API continued as an incubator API.
- Unnamed classes and instance main methods were introduced as a preview feature, making simple Java programs easier to write.

## 6. Java 18 vs Java 19 vs Java 20 vs Java 21

| Feature | Java 18 | Java 19 | Java 20 | Java 21 |
|---|---|---|---|---|
| UTF-8 by default | Permanent | Available | Available | Available |
| Virtual threads | Not yet introduced | Preview | Second preview | Permanent |
| Record patterns | Not yet introduced | Preview | Second preview | Permanent |
| Pattern matching for `switch` | Not yet introduced | Preview | Fourth preview | Permanent |
| Sequenced collections | Not available | Not available | Not available | Permanent |
| String templates | Not available | Not available | Not available | Preview |
| Simple Web Server | Permanent tool | Available | Available | Available |
| Foreign Function and Memory API | Preview | Preview | Third preview | Permanent |

## 7. Java 17 vs Java 21

| Feature | Java 17 | Java 21 |
|---|---|---|
| LTS release | Yes | Yes |
| Virtual threads | Not available as a permanent feature | Permanent |
| Record patterns | Not available | Permanent |
| Pattern matching for `switch` | Preview | Permanent |
| Sequenced collections | Not available | Available |
| Foreign Function and Memory API | Not available as a permanent feature | Permanent |
| String templates | Not available | Preview |
| Unnamed patterns and variables | Not available | Preview |

## 8. Common Java 18–21 Interview Questions

### Q1. What is the biggest feature introduced in Java 21?

Virtual threads are among Java 21's most significant features, particularly for high-concurrency, I/O-bound applications. Record patterns, pattern matching for `switch`, and sequenced collections are also important.

### Q2. What are virtual threads?

Virtual threads are lightweight Java threads managed by the JVM. They make it easier to handle many concurrent tasks, particularly those that spend time waiting for I/O.

### Q3. Are virtual threads faster than platform threads?

Not automatically. They primarily improve scalability and reduce per-thread overhead for high-concurrency workloads. CPU-intensive work does not inherently become faster.

### Q4. Which Java version made virtual threads permanent?

Java 21. They were previewed in Java 19 and Java 20.

### Q5. What are record patterns?

Record patterns allow a program to match a record and extract its components in a single pattern expression. They became permanent in Java 21.

### Q6. What is pattern matching for `switch`?

It extends switch expressions and statements to support patterns, including type patterns. It became permanent in Java 21.

### Q7. What are sequenced collections?

Sequenced collections provide consistent APIs for collections with a defined encounter order, including methods to access or manipulate elements at either end.

### Q8. What changed in Java 18 regarding character encoding?

UTF-8 became the default charset for standard Java APIs that rely on the default charset, improving consistency across platforms.

### Q9. Is string interpolation a permanent feature in Java 21?

No. String templates were a preview feature in Java 21 and should not be treated as permanent Java 21 syntax.

### Q10. What should a Java developer prioritize for Java 21 interviews?

Prioritize virtual threads, pattern matching for `switch`, record patterns, sequenced collections, records, sealed classes, streams, concurrency, and JVM fundamentals.

## Java 18 to Java 21: One-Line Interview Answer

Java 18–21 introduced UTF-8 by default, virtual threads, record patterns, pattern matching for `switch`, sequenced collections, and improvements to the Foreign Function and Memory API, with Java 21 finalizing several major language and concurrency features.