---
title: Java 22 to 25
parent: Modern Java (8 to 25)
nav_order: 5
---

# Java 22 to Java 25 Features

Java 22 through Java 25 introduced improvements to the Java language, concurrency, collections, cryptography, and JVM performance.

For interviews, pay particular attention to **Stream Gatherers, the Foreign Function and Memory API, module import declarations, compact source files, flexible constructor bodies, and scoped values**.

Java 25 was released in September 2025 as an LTS release.

## 1. Major Features at a Glance

| Version | Important features |
|---|---|
| Java 22 | Unnamed variables and patterns finalized, Foreign Function and Memory API finalized, statements before `super()` previewed |
| Java 23 | Primitive types in patterns previewed, module import declarations previewed, Markdown documentation comments previewed |
| Java 24 | Stream Gatherers finalized, Class-File API finalized, virtual-thread synchronization improvements, compact object headers experimental |
| Java 25 | Module import declarations finalized, compact source files and instance `main` methods finalized, flexible constructor bodies finalized, scoped values finalized |

**Important:** A feature's release version does not always mean it became permanent in that release. Preview and incubator features may change before becoming final.

## 2. Java 22 Features

Java 22 was released in March 2024. Important features included the Foreign Function and Memory API becoming permanent and unnamed variables and patterns becoming standard.

### 2.1 Unnamed Variables and Patterns

Java 22 finalized unnamed variables and patterns. The underscore (`_`) can be used where a variable or pattern is required but its value is not needed.

This reduces unnecessary variable names and makes code easier to read.

```java
public class UnnamedVariableExample {
    public static void main(String[] args) {
        try {
            int number = Integer.parseInt("123");
            System.out.println(number);
        } catch (NumberFormatException _) {
            System.out.println("Invalid number");
        }
    }
}
```

**Output:**

```text
123
```

The exception variable is unused, so it can be named `_`.

Another example uses an unnamed pattern:

```java
record Employee(String name, int age) {}

public class UnnamedPatternExample {
    public static void main(String[] args) {
        Object value = new Employee("Ali", 30);

        if (value instanceof Employee(String name, _)) {
            System.out.println(name);
        }
    }
}
```

**Output:**

```text
Ali
```

Here, the age component is not needed.

**Interview point:** Unnamed variables and patterns became permanent in Java 22. The underscore is a special syntax in these contexts, not a general-purpose variable name.

### 2.2 Foreign Function and Memory API

Java 22 finalized the Foreign Function and Memory (FFM) API.

It provides a supported way for Java applications to interact with native code and memory outside the Java heap, without relying on traditional JNI for every use case.

Common applications include:

- Calling native libraries written in C.
- Interacting with native functions.
- Accessing off-heap memory.
- Integrating Java applications with operating-system capabilities.

The API uses concepts such as `Linker`, `Arena`, `MemorySegment`, and `FunctionDescriptor`.

For most Spring Boot developers, FFM is a specialized feature rather than a routine application-development requirement.

### 2.3 Statements Before `super()` in Constructors (Preview)

Traditionally, an explicit constructor invocation such as `super(...)` or `this(...)` must be the first statement in a constructor.

Java 22 previewed a change allowing certain statements before an explicit constructor invocation, provided they do not improperly access the instance being constructed.

This enables validation and preparation before calling the superclass constructor.

The feature evolved in later releases and became permanent in Java 25 under the name **flexible constructor bodies**.

### 2.4 Stream Gatherers (Preview)

Java 22 introduced Stream Gatherers as a preview feature.

Gatherers extend the Stream API by supporting custom intermediate operations, including operations that maintain state or process groups of elements.

They became permanent in Java 24.

See the Java 24 section below for an example.

## 3. Java 23 Features

Java 23 was released in September 2024. Its changes included continued work on pattern matching, module imports, and documentation.

### 3.1 Primitive Types in Patterns (Preview)

Java 23 previewed extending pattern matching to primitive types.

This work aimed to make `switch` and pattern matching more consistent when dealing with primitive values. It evolved through later releases and was still a preview feature in Java 25.

For ordinary application development, continue using standard primitive `switch` expressions where appropriate:

```java
public class PrimitiveSwitchExample {
    public static void main(String[] args) {
        int score = 85;

        String result = switch (score / 10) {
            case 10, 9 -> "Excellent";
            case 8 -> "Very good";
            case 7 -> "Good";
            default -> "Keep practicing";
        };

        System.out.println(result);
    }
}
```

**Output:**

```text
Very good
```

This example uses standard Java syntax and does not require preview features.

### 3.2 Module Import Declarations (Preview)

Java 23 introduced module import declarations as a preview feature.

They allow source files to import the packages exported by a module without explicitly importing each package individually.

The feature was finalized in Java 25.

This can simplify examples and reduce import boilerplate, particularly when working with modular libraries.

### 3.3 Markdown Documentation Comments (Preview)

Java 23 introduced preview support for writing API documentation comments using Markdown.

This allows developers to document APIs with familiar Markdown formatting rather than relying exclusively on traditional HTML-style Javadoc markup.

The feature was preview-only in Java 23 and was not finalized in Java 25.

### 3.4 Other Improvements

Java 23 also continued improvements to:

- Garbage collection and runtime performance.
- The Vector API incubator.
- The Class-File API.
- Pattern matching and other language features under development.

## 4. Java 24 Features

Java 24 was released in March 2025. Its notable changes include permanent Stream Gatherers, a permanent Class-File API, and improvements to virtual-thread synchronization.

### 4.1 Stream Gatherers Became Permanent

Java 24 finalized Stream Gatherers through JEP 485.

The Stream API already provides operations such as `filter()`, `map()`, and `flatMap()`. Gatherers allow custom intermediate stream operations that are more expressive than ordinary mapping and filtering.

For example, Java 24's `Gatherers.windowFixed()` can group stream elements into fixed-size windows.

```java
import java.util.List;
import java.util.stream.Gatherers;

public class GathererExample {
    public static void main(String[] args) {
        List<Integer> numbers =
                List.of(1, 2, 3, 4, 5, 6, 7);

        List<List<Integer>> windows = numbers.stream()
                .gather(Gatherers.windowFixed(3))
                .toList();

        System.out.println(windows);
    }
}
```

**Output:**

```text
[[1, 2, 3], [4, 5, 6], [7]]
```

The final window can contain fewer elements than the requested window size.

**When is this useful?**

- Processing batches of records.
- Grouping sequential elements.
- Building custom stream transformations.
- Processing events and time-series data.

**Interview distinction:** Stream Gatherers became a permanent feature in Java 24. They are different from the `Collector` API, which primarily collects stream results into a final container or summary.

### 4.2 Class-File API Became Permanent

Java 24 finalized the Class-File API.

This API provides a standard way to parse, generate, and transform Java class files.

It can be useful for:

- Bytecode instrumentation.
- Framework development.
- Compiler tooling.
- Static analysis.
- Code-generation tools.

Before this API, tools often relied on third-party libraries or internal JDK APIs to manipulate bytecode.

Most application developers do not need to use the Class-File API directly, but it is important for developers working on frameworks and tooling.

### 4.3 Virtual-Thread Synchronization Improvements

Java 24 introduced improvements through JEP 491, allowing virtual threads to unmount from carrier threads when blocked inside synchronized methods or blocks in situations that previously caused pinning.

This can improve scalability for applications using virtual threads and synchronization.

**Important:** This does not mean synchronization should be removed from application code. `synchronized` remains a valid mechanism for protecting shared mutable state.

### 4.4 Compact Object Headers (Experimental)

Java 24 introduced compact object headers as an experimental feature.

Object headers store JVM metadata associated with objects. Reducing their size can reduce memory usage, particularly for applications with large numbers of small objects.

Potential benefits include:

- Lower memory overhead.
- Improved memory density.
- Better cache utilization in some workloads.

This was experimental in Java 24, so it should not be treated as a standard feature available without special configuration.

### 4.5 Other Java 24 Changes

- Improvements to garbage collection.
- Further work on flexible constructor bodies.
- Continued development of structured concurrency and scoped values.
- Additional security and runtime improvements.

## 5. Java 25 Features

Java 25 was released on September 16, 2025, as an LTS release.

Several language features that had been previewed in earlier releases became permanent.

### 5.1 Module Import Declarations Became Permanent

Java 25 finalized module import declarations through JEP 511.

They allow a source file to import all packages exported by a module without listing every package individually.

Conceptually, this can reduce repetitive imports when working with modular Java libraries.

This feature is especially useful for concise examples and exploratory code. In production applications, explicit imports can still improve clarity.

### 5.2 Compact Source Files and Instance Main Methods

Java 25 finalized compact source files and instance `main` methods through JEP 512.

Traditionally, a simple Java program requires a class declaration and a `static main` method:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

Java 25 allows a simpler form:

```java
void main() {
    System.out.println("Hello, Java 25!");
}
```

The second example is a compact source file using an instance `main` method. It can be compiled and executed with Java 25.

This feature is designed to make Java easier to learn and to simplify small programs. Traditional class-based programs remain fully supported.

**Interview point:** This feature became permanent in Java 25. Do not assume that the compact form compiles under Java 21.

### 5.3 Flexible Constructor Bodies Became Permanent

Java 25 finalized flexible constructor bodies through JEP 513.

This allows certain statements to run before an explicit superclass constructor invocation, as long as they follow the language's restrictions.

It is useful for validating constructor arguments or preparing local values before calling `super(...)`.

For example, a constructor can validate an argument before invoking the superclass constructor:

```java
class Person {
    private final String name;

    Person(String name) {
        this.name = name;
    }
}

class Employee extends Person {
    Employee(String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException(
                    "Name must not be blank"
            );
        }

        super(name);
    }
}
```

This example demonstrates the kind of validation the feature enables in Java 25. Compile it with a Java 25 compiler; older Java versions require the superclass constructor invocation to occur first.

**Why is it useful?**

- Validating constructor parameters before superclass initialization.
- Preparing local variables used in constructor delegation.
- Improving error handling during object construction.

The rules still prohibit accessing the object under construction in ways that could expose an incompletely initialized instance.

### 5.4 Scoped Values Became Permanent

Java 25 finalized scoped values through JEP 506.

Scoped values provide a way to share immutable data across a bounded execution scope and its child tasks. They are designed as an alternative to some uses of `ThreadLocal`, particularly in applications using structured concurrency and virtual threads.

Example:

```java
import java.lang.ScopedValue;

public class ScopedValueExample {
    private static final ScopedValue<String> USER =
            ScopedValue.newInstance();

    public static void main(String[] args) {
        ScopedValue.where(USER, "Danish").run(() -> {
            System.out.println(USER.get());
        });
    }
}
```

**Output:**

```text
Danish
```

The value is bound for the duration of the scoped operation.

**Benefits:**

- Makes contextual data available within a bounded execution.
- Avoids some lifecycle and memory-management concerns associated with `ThreadLocal`.
- Fits well with modern concurrency designs.

Scoped values are not a universal replacement for `ThreadLocal`. Choose the mechanism according to the data-sharing and lifecycle requirements.

### 5.5 Primitive Types in Patterns (Third Preview)

Java 25 continued preview support for primitive types in patterns, `instanceof`, and `switch`.

The goal is to extend pattern matching consistently to primitive values as well as reference types.

This feature was still a preview feature in Java 25, so applications require the appropriate preview-feature configuration to use it.

**Interview point:** Do not describe primitive patterns as a finalized Java 25 feature.

### 5.6 Other Java 25 Improvements

Java 25 also includes improvements to:

- Ahead-of-time class loading and linking.
- Garbage collection and runtime performance.
- Java Flight Recorder diagnostics.
- Cryptographic APIs.
- Structured concurrency, which remains a preview feature.
- Monitoring and profiling capabilities.

## 6. Java 22 vs Java 23 vs Java 24 vs Java 25

| Feature | Java 22 | Java 23 | Java 24 | Java 25 |
|---|---|---|---|---|
| Unnamed variables and patterns | Final | Available | Available | Available |
| Foreign Function and Memory API | Final | Available | Available | Available |
| Stream Gatherers | Preview | Preview | Final | Available |
| Class-File API | Preview | Preview | Final | Available |
| Primitive types in patterns | Not yet introduced | Preview | Second preview | Third preview |
| Module import declarations | Not yet introduced | Preview | Second preview | Final |
| Flexible constructor bodies | Preview | Second preview | Third preview | Final |
| Compact source files and instance `main` methods | Not yet introduced | Not yet introduced | Preview | Final |
| Scoped values | Preview | Preview | Fourth preview | Final |
| Structured concurrency | Preview | Preview | Fourth preview | Preview |
| Compact object headers | Not yet introduced | Not yet introduced | Experimental | Experimental |

Feature status is release-specific. Preview features may change, be withdrawn, or become permanent in a later Java release.

## 7. Java 21 vs Java 25

| Feature | Java 21 | Java 25 |
|---|---|---|
| LTS release | Yes | Yes |
| Virtual threads | Permanent | Permanent |
| Pattern matching for `switch` | Permanent | Permanent |
| Record patterns | Permanent | Permanent |
| Sequenced collections | Permanent | Permanent |
| Stream Gatherers | Not available | Permanent |
| Class-File API | Not available as a permanent API | Permanent |
| Scoped values | Not available as a permanent feature | Permanent |
| Compact source files | Not available | Permanent |
| Flexible constructor bodies | Not available | Permanent |
| Module import declarations | Not available | Permanent |

## 8. Common Java 22–25 Interview Questions

### Q1. What are Stream Gatherers?

Stream Gatherers allow developers to implement custom intermediate operations in the Stream API. Java 24 finalized this feature.

### Q2. What is the Foreign Function and Memory API?

It provides a standard way for Java applications to interact with native functions and memory outside the Java heap. The API became permanent in Java 22.

### Q3. What are unnamed variables?

Unnamed variables use `_` where a value must be represented syntactically but does not need to be referenced. This feature became permanent in Java 22.

### Q4. What is the Class-File API?

The Class-File API provides standard Java APIs for parsing, generating, and transforming class files. It became permanent in Java 24.

### Q5. What are compact source files in Java 25?

They allow simple programs to be written without explicitly declaring a surrounding class and a traditional `static main` method. The feature became permanent in Java 25.

### Q6. What are flexible constructor bodies?

They allow certain statements before an explicit `super(...)` or `this(...)` constructor invocation, subject to language restrictions. They became permanent in Java 25.

### Q7. What are scoped values?

Scoped values share data within a bounded execution scope. They became permanent in Java 25 and are useful in modern concurrent applications.

### Q8. Are structured concurrency and primitive patterns permanent in Java 25?

No. Both remain preview features in Java 25.

### Q9. Which Java versions are LTS among Java 21 and Java 25?

Both Java 21 and Java 25 are LTS releases.

### Q10. Which features should a senior Java developer prioritize?

Prioritize virtual threads, Stream Gatherers, scoped values, the Foreign Function and Memory API, Java 25 language changes, garbage collection, and the difference between permanent and preview features.

## Java 22 to Java 25: One-Line Interview Answer

Java 22–25 introduced the permanent Foreign Function and Memory API, unnamed variables, Stream Gatherers, the Class-File API, scoped values, flexible constructor bodies, module import declarations, and compact source files, with Java 25 serving as an LTS release.