---
title: Exception Handling
parent: Core Java
nav_order: 6
---

# Exception Handling
{: .no_toc }

Checked vs unchecked exceptions and resource handling.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

---
title: Classpath, Exception Hierarchy, Throw vs Throws vs Throwable
parent: Core Java
nav_order: 27
---

# Classpath, Exception Hierarchy, Throw vs Throws vs Throwable

This guide covers three common Java interview topics:

1. Classpath errors and how to troubleshoot them.
2. The Java exception hierarchy.
3. The difference between `throw`, `throws`, and `Throwable`.

## 1. What is the classpath in Java?

The **classpath** tells Java tools and the JVM where to find application classes and libraries.

It can include:
- Directories containing compiled `.class` files.
- JAR files containing compiled classes.
- Other locations configured through the relevant Java launch or build tooling.

The classpath can be configured using the `-cp` or `-classpath` command-line option, or through the `CLASSPATH` environment variable.

### Example: Setting the classpath

Suppose your project has this structure:

```text
project/
├── src/
│   └── com/
│       └── example/
│           └── Main.java
└── out/
    └── com/
        └── example/
            └── Main.class
```

Compile the Java source:

```bash
javac -d out src/com/example/Main.java
```

Run the compiled class:

```bash
java -cp out com.example.Main
```

Here:
- `out` is the classpath entry.
- `com.example.Main` is the fully qualified class name.
- Java searches for `out/com/example/Main.class`.

For a JAR dependency, you can include the JAR in the classpath.

macOS/Linux:

```bash
java -cp "out:lib/example.jar" com.example.Main
```

Windows:

```bash
java -cp "out;lib/example.jar" com.example.Main
```

The classpath separator is `:` on macOS/Linux and `;` on Windows.

## 2. What is a classpath exception?

"Classpath exception" is an informal expression, not a specific Java exception class. It generally refers to errors caused by missing classes or incorrectly configured class-loading locations.

Two common errors are `ClassNotFoundException` and `NoClassDefFoundError`.

### ClassNotFoundException

A checked exception that can occur when code explicitly attempts to load a class by name and the class loader cannot find it.

Example:

```java
public class Main {
    public static void main(String[] args)
            throws ClassNotFoundException {

        Class.forName("com.example.MissingClass");
    }
}
```

If that class cannot be found by the relevant class loader, `Class.forName()` throws `ClassNotFoundException`.

Common causes:
- A required JAR is missing.
- The class name or package name is incorrect.
- A dependency is absent from the runtime classpath.
- The class is unavailable to the selected class loader.

### NoClassDefFoundError

An unchecked `Error` that occurs when the JVM cannot find a class definition that was available or expected when the code was compiled or linked.

Example scenario:

- The application compiles successfully because a dependency is present.
- The dependency is omitted when the application is launched.
- The JVM later needs a class from that dependency and cannot find its definition.

Another cause can be a class that failed initialization earlier. In that situation, the underlying problem may be an exception thrown by a static initializer.

### ClassNotFoundException vs NoClassDefFoundError

| Feature | `ClassNotFoundException` | `NoClassDefFoundError` |
|---|---|---|
| Type | Checked exception | Unchecked `Error` |
| Common scenario | Explicit class loading fails | JVM cannot resolve a required class definition |
| Typical example | `Class.forName()` cannot locate a class | Runtime dependency missing |
| Handling | Catch it or declare it | Diagnose the underlying class-loading or initialization problem |

### How to troubleshoot classpath errors

1. **Check the class name:** Verify the package declaration and fully qualified class name.
2. **Check the runtime classpath:** Ensure the required directory and JAR files are included.
3. **Check build dependencies:** In Maven or Gradle, verify that the dependency is available at runtime, not only during compilation.
4. **Check Java versions:** An incompatible class-file version usually produces `UnsupportedClassVersionError`, rather than a generic classpath exception.
5. **Check permissions and paths:** Ensure the application can access the required files.
6. **Check initialization errors:** If you see `NoClassDefFoundError: Could not initialize class ...`, inspect the earlier exception thrown during static initialization.

**Interview tip:** A classpath problem does not always mean `ClassNotFoundException`. Identify the exact exception or error message before troubleshooting.

## 3. Explain the exception hierarchy in Java

All Java exceptions and errors derive from `java.lang.Throwable`.

The main hierarchy is:

```text
Object
  └── Throwable
      ├── Error
      │   ├── AssertionError
      │   ├── OutOfMemoryError
      │   ├── StackOverflowError
      │   └── LinkageError
      │       └── NoClassDefFoundError
      │
      └── Exception
          ├── IOException
          ├── SQLException
          ├── ClassNotFoundException
          │
          └── RuntimeException
              ├── NullPointerException
              ├── ArithmeticException
              ├── IllegalArgumentException
              ├── IllegalStateException
              └── IndexOutOfBoundsException
```

### Throwable

`Throwable` is the root class for objects that can be thrown and caught.

It provides methods such as:

```java
getMessage()
getCause()
printStackTrace()
getSuppressed()
```

### Error

`Error` generally represents serious JVM, linkage, or resource problems that applications normally should not try to recover from through ordinary exception handling.

Examples:
- `OutOfMemoryError`
- `StackOverflowError`
- `NoClassDefFoundError`

### Exception

`Exception` represents conditions an application may need to handle or report.

Exceptions are divided into checked exceptions and unchecked exceptions.

### Checked exceptions

Checked exceptions are exceptions other than `RuntimeException` and its subclasses. They must generally be caught or declared in a method's `throws` clause.

Examples:
- `IOException`
- `SQLException`
- `ClassNotFoundException`

```java
import java.io.IOException;

public void readFile() throws IOException {
    // File-reading logic
}
```

### Unchecked exceptions

`RuntimeException` and its subclasses are unchecked. Java does not require them to be caught or declared.

```java
public class Main {
    public static void main(String[] args) {
        String name = null;

        System.out.println(name.length());
    }
}
```

This code throws `NullPointerException` at runtime.

### Checked vs unchecked exceptions

| Feature | Checked exception | Unchecked exception |
|---|---|---|
| Hierarchy | `Exception`, excluding `RuntimeException` subclasses | `RuntimeException` subclasses; `Error` is also unchecked |
| Must catch or declare? | Yes, generally | No |
| Typical example | `IOException` | `NullPointerException` |
| Common use | Recoverable or expected external failures | Programming errors or invalid runtime states |

**Important:** `Error` is unchecked, but it is not a subclass of `RuntimeException`. Both `Error` and `Exception` extend `Throwable`.

## 4. Explain throw, throws, and Throwable in Java

These terms are related but serve different purposes.

### A. `throw` keyword

The `throw` statement explicitly throws a particular exception object.

```java
public class Main {
    static void validateAge(int age) {
        if (age < 18) {
            throw new IllegalArgumentException(
                    "Age must be at least 18");
        }

        System.out.println("Eligible");
    }

    public static void main(String[] args) {
        validateAge(15);
    }
}
```

Output:

```text
Exception in thread "main" java.lang.IllegalArgumentException:
Age must be at least 18
```

Key points:
- `throw` is used inside a method, constructor, or block.
- It is followed by a throwable object.
- It transfers control to exception handling.
- You can throw checked exceptions or unchecked exceptions.

For a checked exception, the surrounding code must handle it or declare it appropriately.

### B. `throws` keyword

The `throws` clause declares exception types that a method or constructor may propagate to its caller.

```java
import java.io.IOException;

public class FileService {

    public void readFile() throws IOException {
        throw new IOException("Unable to read file");
    }
}
```

The caller must handle or declare the checked exception:

```java
public class Main {
    public static void main(String[] args) {
        FileService service = new FileService();

        try {
            service.readFile();
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

Output:

```text
Unable to read file
```

Key points:
- `throws` appears in the method or constructor declaration.
- It is followed by one or more exception types.
- It declares the exceptions that may propagate; it does not itself throw an exception.
- Unchecked exceptions do not need to be declared.

### C. `Throwable` class

`Throwable` is a class, not a keyword. It is the superclass of both `Exception` and `Error`.

```java
public class Main {
    public static void main(String[] args) {
        try {
            throw new IllegalArgumentException("Invalid input");
        } catch (Throwable t) {
            System.out.println(t.getMessage());
        }
    }
}
```

Output:

```text
Invalid input
```

Although `catch (Throwable t)` is legal, catching `Throwable` broadly is generally discouraged because it also catches serious `Error` instances. Prefer catching the specific exception types your code can meaningfully handle.

### Comparison table

| Feature | `throw` | `throws` | `Throwable` |
|---|---|---|---|
| What is it? | Keyword | Keyword | Class |
| Purpose | Throws an exception object | Declares possible propagated exceptions | Base type for exceptions and errors |
| Location | Statement body | Method or constructor signature | Type hierarchy |
| Followed by | An object | Exception class names | Not applicable |
| Example | `throw new IOException();` | `throws IOException` | `catch (Throwable t)` |

## 5. Can we create custom exceptions?

Yes. Extend `Exception` for a checked custom exception, or `RuntimeException` for an unchecked one.

### Checked custom exception

```java
class InsufficientBalanceException extends Exception {
    public InsufficientBalanceException(String message) {
        super(message);
    }
}

class BankAccount {
    private double balance = 1000;

    public void withdraw(double amount)
            throws InsufficientBalanceException {

        if (amount > balance) {
            throw new InsufficientBalanceException(
                    "Insufficient balance");
        }

        balance -= amount;
    }
}
```

The caller must catch or declare `InsufficientBalanceException`.

### Unchecked custom exception

```java
class InvalidOrderException extends RuntimeException {
    public InvalidOrderException(String message) {
        super(message);
    }
}
```

This exception does not have to be caught or declared by the caller.

## 6. Quick interview revision

- **Classpath:** Locations used to find classes and libraries.
- **`ClassNotFoundException`:** A requested class cannot be found during explicit class loading.
- **`NoClassDefFoundError`:** A required class definition cannot be resolved, or class initialization has previously failed.
- **`Throwable`:** The common superclass of `Exception` and `Error`.
- **`Error`:** Usually represents serious JVM or linkage problems.
- **Checked exception:** Must generally be caught or declared.
- **Unchecked exception:** Does not have to be caught or declared.
- **`throw`:** Throws an exception object.
- **`throws`:** Declares exceptions a method may propagate.
- **Custom exceptions:** Extend `Exception` or `RuntimeException` according to the intended handling contract.


---

# Checked vs Unchecked Exceptions in Java

## 1. What is a checked exception?

A **checked exception** is an exception that the Java compiler checks at compile time. If a method can throw a checked exception, the caller must generally either handle it using `try-catch` or declare it using `throws`.

Checked exceptions typically represent conditions such as file access failures, database errors, or problems communicating with external systems.

### Example

```java
import java.io.FileReader;
import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        try {
            FileReader reader = new FileReader("data.txt");
            reader.close();
        } catch (IOException e) {
            System.out.println("File operation failed");
        }
    }
}
```

If you remove the `try-catch` block and do not declare `throws IOException`, the compiler reports an error because `FileReader` can throw a checked `IOException`.

Other examples include:
- `IOException`
- `SQLException`
- `ClassNotFoundException`
- `InterruptedException`

## 2. What is an unchecked exception?

An **unchecked exception** is an exception that the compiler does not require you to catch or declare.

In Java, unchecked exceptions include `RuntimeException` and its subclasses. `Error` and its subclasses are also unchecked, although they are not exceptions in the `RuntimeException` hierarchy.

Unchecked exceptions often indicate programming mistakes, invalid arguments, or invalid object states.

### Example

```java
public class Main {
    public static void main(String[] args) {
        String name = null;

        System.out.println(name.length());
    }
}
```

Output:

```text
Exception in thread "main" java.lang.NullPointerException
```

This code compiles successfully, but throws a `NullPointerException` at runtime.

Other examples include:
- `NullPointerException`
- `ArithmeticException`
- `IllegalArgumentException`
- `IllegalStateException`
- `IndexOutOfBoundsException`

## 3. Key differences

| Feature | Checked exception | Unchecked exception |
|---|---|---|
| Checked by compiler | Yes | No |
| Must catch or declare | Generally yes | No |
| Main hierarchy | `Exception`, excluding `RuntimeException` and its subclasses | `RuntimeException` and its subclasses; `Error` is also unchecked |
| Typical causes | External failures and recoverable conditions | Programming errors or invalid states |
| Example | `IOException` | `NullPointerException` |
| Custom exception base class | Usually `Exception` | Usually `RuntimeException` |

## 4. Difference in the exception hierarchy

```text
Throwable
├── Error                         (Unchecked)
│   ├── OutOfMemoryError
│   └── StackOverflowError
│
└── Exception
    ├── IOException               (Checked)
    ├── SQLException              (Checked)
    ├── ClassNotFoundException    (Checked)
    │
    └── RuntimeException          (Unchecked)
        ├── NullPointerException
        ├── ArithmeticException
        └── IllegalArgumentException
```

Remember: `Error` is unchecked, but it does not extend `RuntimeException`.

## 5. How to create custom exceptions

### Checked custom exception

Extend `Exception`:

```java
class InsufficientBalanceException extends Exception {
    public InsufficientBalanceException(String message) {
        super(message);
    }
}
```

Usage:

```java
static void withdraw(double balance, double amount)
        throws InsufficientBalanceException {

    if (amount > balance) {
        throw new InsufficientBalanceException(
                "Insufficient balance");
    }
}
```

The caller must catch or declare `InsufficientBalanceException`.

### Unchecked custom exception

Extend `RuntimeException`:

```java
class InvalidAgeException extends RuntimeException {
    public InvalidAgeException(String message) {
        super(message);
    }
}
```

Usage:

```java
static void validateAge(int age) {
    if (age < 18) {
        throw new InvalidAgeException(
                "Age must be at least 18");
    }
}
```

The caller is not required to catch or declare `InvalidAgeException`.

## 6. When should you use each?

**Use checked exceptions when:**
- The caller can reasonably recover from the failure.
- The caller is expected to make an explicit decision about handling it.
- The API's contract benefits from requiring handling or declaration.

**Use unchecked exceptions when:**
- An argument violates a method's contract.
- An operation is invoked in an invalid state.
- The failure is a programming error or mandatory handling by every caller would add little value.

The choice depends on the API design; not every external failure must be represented by a checked exception.

## 7. Common interview questions

**Q1. Does a checked exception occur at compile time?**

Not necessarily. The compiler checks whether the exception is handled or declared. The exception itself usually occurs at runtime when the relevant operation fails.

**Q2. Is `NullPointerException` checked or unchecked?**

Unchecked, because it extends `RuntimeException`.

**Q3. Is `Error` a checked exception?**

No. `Error` is unchecked, but it is a separate branch under `Throwable`.

**Q4. Can we throw an unchecked exception without declaring it using `throws`?**

Yes. For example:

```java
throw new IllegalArgumentException("Invalid input");
```

**Q5. Can a method declare an unchecked exception using `throws`?**

Yes, but the declaration is not required by the compiler.

```java
public void validate() throws IllegalArgumentException {
    throw new IllegalArgumentException("Invalid input");
}
```

**Q6. Which type should a custom business exception extend?**

It depends on the intended API contract. Extend `Exception` if callers should be required to catch or declare it; extend `RuntimeException` if that requirement is not appropriate.

## Quick revision

- Checked exceptions are checked by the compiler and must generally be caught or declared.
- Unchecked exceptions do not require catching or declaration.
- `IOException` is checked.
- `NullPointerException` is unchecked.
- `Error` is unchecked but is not a subclass of `Exception`.
- Extend `Exception` for a checked custom exception and `RuntimeException` for an unchecked custom exception.


---

# Try-with-Resources in Java

## 1. What is try-with-resources?

Try-with-resources was introduced in **Java 7** to automatically close resources when execution leaves a `try` statement, whether it completes normally or an exception occurs.

A resource must implement `java.lang.AutoCloseable` to be eligible for a try-with-resources statement.

This feature is commonly used with files, streams, database connections, statements, and readers.

### Syntax

```java
try (ResourceType resource = createResource()) {
    // Use the resource
} catch (Exception e) {
    // Handle exception
}
```

Java automatically calls the resource's `close()` method when control exits the `try` block.

## 2. Which classes are eligible for the resource block?

Any class that implements `AutoCloseable` can be used as a resource, provided the resource declaration meets Java's syntax rules.

Common examples include:

| Class or interface | Package | Typical use |
|---|---|---|
| `FileInputStream` | `java.io` | Reading file bytes |
| `FileOutputStream` | `java.io` | Writing file bytes |
| `BufferedReader` | `java.io` | Buffered text reading |
| `BufferedWriter` | `java.io` | Buffered text writing |
| `Scanner` | `java.util` | Reading and parsing input |
| `PrintWriter` | `java.io` | Writing formatted text |
| `Connection` | `java.sql` | Database connection |
| `Statement` | `java.sql` | Executing SQL statements |
| `PreparedStatement` | `java.sql` | Executing parameterized SQL |
| `ResultSet` | `java.sql` | Reading query results |
| `Socket` | `java.net` | Network communication |
| `ZipFile` | `java.util.zip` | Reading ZIP archives |

The important interview point is that **implementing `AutoCloseable` is the general eligibility requirement**, not membership in a particular package.

## 3. Example using FileInputStream

```java
import java.io.FileInputStream;
import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        try (FileInputStream input =
                     new FileInputStream("data.txt")) {

            int data;

            while ((data = input.read()) != -1) {
                System.out.print((char) data);
            }

        } catch (IOException e) {
            System.err.println("File operation failed: "
                    + e.getMessage());
        }
    }
}
```

The `FileInputStream` is automatically closed when the `try` block exits, even if an `IOException` occurs while reading.

There is no need to explicitly call `input.close()` in a `finally` block.

## 4. Example using multiple resources

You can declare multiple resources separated by semicolons. Java closes them in the **reverse order of declaration**.

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        try (
            FileReader file = new FileReader("data.txt");
            BufferedReader reader = new BufferedReader(file)
        ) {
            System.out.println(reader.readLine());

        } catch (IOException e) {
            System.err.println(e.getMessage());
        }
    }
}
```

Here, `reader` is closed first, followed by `file`. Closing the `BufferedReader` also closes its underlying reader, so declaring both resources is usually redundant. A simpler version is:

```java
try (BufferedReader reader =
         new BufferedReader(new FileReader("data.txt"))) {
    System.out.println(reader.readLine());
}
```

## 5. What is AutoCloseable?

`AutoCloseable` is an interface in the `java.lang` package.

Its essential method is:

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

A class can implement this interface to define how its resources are released.

### Custom AutoCloseable example

```java
class MyResource implements AutoCloseable {

    public void use() {
        System.out.println("Using resource");
    }

    @Override
    public void close() {
        System.out.println("Resource closed");
    }
}

public class Main {
    public static void main(String[] args) {
        try (MyResource resource = new MyResource()) {
            resource.use();
        }
    }
}
```

Output:

```text
Using resource
Resource closed
```

The `close()` method runs automatically after the body of the `try` statement.

### AutoCloseable vs Closeable

| Feature | `AutoCloseable` | `Closeable` |
|---|---|---|
| Introduced | Java 7 | Java 5 |
| Package | `java.lang` | `java.io` |
| Close method | `close() throws Exception` | `close() throws IOException` |
| Purpose | General resource cleanup | I/O resource cleanup |
| Used by try-with-resources | Yes | Yes, because `Closeable` extends `AutoCloseable` |

## 6. Can we use an existing variable in the resource block?

Yes. Starting with **Java 9**, an existing final or effectively final variable can be used as a resource.

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader reader =
                new BufferedReader(new FileReader("data.txt"));

        try (reader) {
            System.out.println(reader.readLine());
        }
    }
}
```

Before Java 9, the resource generally had to be declared inside the resource specification.

An effectively final variable is one that is not reassigned after initialization.

## 7. What happens if both the try block and close() throw exceptions?

If the `try` body throws an exception and `close()` also throws an exception, the exception from the `try` body remains the primary exception. The exception thrown by `close()` is attached as a **suppressed exception**.

Java provides `Throwable.getSuppressed()` to inspect suppressed exceptions.

```java
class MyResource implements AutoCloseable {
    @Override
    public void close() {
        throw new RuntimeException("Close failed");
    }
}

public class Main {
    public static void main(String[] args) {
        try (MyResource resource = new MyResource()) {
            throw new RuntimeException("Work failed");
        } catch (RuntimeException e) {
            System.out.println(e.getMessage());

            for (Throwable suppressed : e.getSuppressed()) {
                System.out.println(suppressed.getMessage());
            }
        }
    }
}
```

Output:

```text
Work failed
Close failed
```

This behavior helps preserve the original failure while still making cleanup failures available for debugging.

## 8. Interview questions and quick revision

**Q1. Which classes can be used in try-with-resources?**

Any class implementing `AutoCloseable`, including classes that implement `Closeable`.

**Q2. Is a `finally` block required?**

No. Resources declared in try-with-resources are closed automatically. You can still use `catch` and `finally` when needed for other cleanup or error handling.

**Q3. In what order are multiple resources closed?**

In reverse declaration order: the last declared resource closes first.

**Q4. Can we create our own resource class?**

Yes. Implement `AutoCloseable` and provide the required `close()` method.

**Q5. What is the difference between `AutoCloseable` and `Closeable`?**

`AutoCloseable` is the general cleanup contract. `Closeable` specializes it for I/O and declares `IOException` from `close()`.

**Q6. What if closing a resource throws an exception?**

The exception propagates unless handled. If another exception is already in flight from the `try` body, the closing exception is suppressed.

**Remember:** Try-with-resources improves reliability by ensuring that eligible resources are closed automatically, reducing boilerplate and preventing many resource leaks.


---


# Runtime Exceptions in Java and How They Work Internally

## 1. What Is a Runtime Exception?

A **runtime exception** is an exception that can occur while a Java program is executing. Runtime exceptions are unchecked exceptions because the Java compiler does not require methods to catch them or declare them using a `throws` clause.

Runtime exceptions are subclasses of `java.lang.RuntimeException`, which extends `Exception`.

Examples include:

- `NullPointerException`
- `ArithmeticException`
- `ArrayIndexOutOfBoundsException`
- `IllegalArgumentException`
- `NumberFormatException`

```java
public class RuntimeExceptionExample {
    public static void main(String[] args) {
        String name = null;

        System.out.println(name.length());
    }
}
```

Output (representative):

```text
Exception in thread "main" java.lang.NullPointerException
```

The program attempts to call `length()` on a `null` reference, so a `NullPointerException` is thrown.

## 2. Exception Hierarchy in Java

The exception hierarchy starts with `Throwable`.

```text
Object
  └── Throwable
       ├── Error
       │    ├── OutOfMemoryError
       │    └── StackOverflowError
       │
       └── Exception
            ├── IOException                 (checked)
            ├── SQLException                (checked)
            │
            └── RuntimeException            (unchecked)
                 ├── NullPointerException
                 ├── ArithmeticException
                 ├── IllegalArgumentException
                 ├── NumberFormatException
                 └── IndexOutOfBoundsException
                      └── ArrayIndexOutOfBoundsException
```

**Important distinction:**

- Checked exceptions must generally be caught or declared.
- Runtime exceptions do not have this compiler-enforced requirement.
- Errors represent serious problems, often involving the JVM or execution environment, and are generally not intended to be handled as ordinary application exceptions.

`RuntimeException` is itself a subclass of `Exception`, but not every subclass of `Exception` is unchecked.

## 3. How Are Runtime Exceptions Implemented Internally?

Runtime exceptions use Java's `Throwable` mechanism. They are ordinary objects that represent exceptional conditions, but the JVM and Java runtime cooperate to detect certain conditions, create exceptions, and transfer control to an appropriate handler.

### Step 1: An exceptional condition occurs

Consider integer division by zero:

```java
int a = 10;
int b = 0;

int result = a / b;
```

For integer division, Java does not permit a zero divisor. The operation throws `ArithmeticException`.

### Step 2: An exception object is created

Conceptually, the runtime produces an exception object of the appropriate class, such as:

```java
new ArithmeticException("/ by zero");
```

This is an illustration of the resulting exception object, not a claim that the JVM literally executes this exact Java constructor expression at every throw site.

### Step 3: Control transfers to an exception handler

The JVM looks for a matching exception handler in the current method. If one is found, execution transfers to that handler. Otherwise, the exception propagates up the call stack.

```java
public class ArithmeticExample {
    public static void main(String[] args) {
        try {
            int result = 10 / 0;
            System.out.println(result);
        } catch (ArithmeticException e) {
            System.out.println("Cannot divide an integer by zero.");
        }

        System.out.println("Program continues.");
    }
}
```

Output:

```text
Cannot divide an integer by zero.
Program continues.
```

### Step 4: The exception may propagate up the call stack

If no matching handler exists in the current method, the JVM searches for a suitable handler in the caller, continuing up the call stack.

If no handler is found before the exception reaches the thread's uncaught-exception handling mechanism, the thread terminates. In a typical application, the uncaught exception and stack trace are printed.

## 4. Are Runtime Exceptions Always Thrown by the JVM?

No. This is a common interview misconception.

Runtime exceptions can originate from different sources:

1. **JVM-detected conditions:** For example, integer division by zero causes `ArithmeticException`, and invalid array access causes `ArrayIndexOutOfBoundsException`.
2. **Java library code:** Methods may throw exceptions when their preconditions are violated.
3. **Application code:** Developers can explicitly throw runtime exceptions using `throw`.

Example:

```java
public class ValidationExample {
    static void validateAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException(
                "Age cannot be negative"
            );
        }
    }

    public static void main(String[] args) {
        validateAge(-5);
    }
}
```

The method deliberately throws an `IllegalArgumentException`. The JVM then handles the exception according to Java's exception mechanism.

## 5. How to Catch Runtime Exceptions

Use a `try-catch` block when the application can recover meaningfully from the exceptional condition.

```java
public class CatchRuntimeExceptions {
    public static void main(String[] args) {
        try {
            String value = null;
            System.out.println(value.length());

            int[] numbers = {10, 20, 30};
            System.out.println(numbers[5]);
        } catch (NullPointerException e) {
            System.out.println("The reference is null.");
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Invalid array index.");
        } catch (RuntimeException e) {
            System.out.println("Another runtime error occurred.");
        }
    }
}
```

Output:

```text
The reference is null.
```

The first exception interrupts execution of the `try` block. The remaining statements in that block are skipped, and the matching `catch` block executes.

**Catch order matters:** More specific exception types must appear before their broader superclass types. Placing `catch (RuntimeException e)` before `catch (NullPointerException e)` would make the latter unreachable.

Avoid catching `RuntimeException` indiscriminately unless you have a meaningful recovery or boundary-handling strategy. Broad catches can hide programming defects.

## 6. Checked vs Unchecked Exceptions

| Feature | Checked exception | Runtime exception |
|---|---|---|
| Parent class | Usually `Exception`, excluding `RuntimeException` subclasses | `RuntimeException` |
| Compiler requires catch or declaration? | Yes, generally | No |
| Must be declared in a method signature? | If not caught, generally yes | No |
| Typical examples | `IOException`, `SQLException` | `NullPointerException`, `IllegalArgumentException` |
| Common use | Recoverable or externally influenced failures | Programming errors, invalid arguments, violated preconditions |

The checked/unchecked distinction is about compile-time handling requirements, not whether an exception occurs at runtime. Both types can occur while a program executes.

## 7. How to Create a Custom Runtime Exception

Extend `RuntimeException` when you want to represent an application-specific unchecked failure.

```java
class InvalidOrderException extends RuntimeException {
    public InvalidOrderException(String message) {
        super(message);
    }

    public InvalidOrderException(
            String message, Throwable cause) {
        super(message, cause);
    }
}

public class OrderService {
    static void placeOrder(int quantity) {
        if (quantity <= 0) {
            throw new InvalidOrderException(
                "Order quantity must be positive"
            );
        }

        System.out.println("Order placed.");
    }

    public static void main(String[] args) {
        placeOrder(0);
    }
}
```

The first constructor passes the message to the superclass. The second also preserves the original cause, which is useful when translating one exception into a more meaningful application exception.

Choose a checked custom exception when callers should be required to acknowledge the failure; choose a runtime exception when that compiler-enforced requirement is not appropriate.

## 8. Best Practices

- Validate arguments at API boundaries.
- Prevent predictable failures instead of relying on exceptions for normal control flow.
- Catch exceptions only where recovery, translation, logging, or appropriate handling is possible.
- Preserve the original cause when wrapping exceptions.
- Avoid silently swallowing exceptions.
- Do not catch `Error` or broad runtime exceptions indiscriminately.
- Use meaningful exception types and messages.

## 9. Common Interview Questions

**Q1. Why are runtime exceptions called unchecked?**

Because the compiler does not require a method to catch them or declare them using `throws`.

**Q2. Is `NullPointerException` thrown only by the JVM?**

No. The JVM may detect a null dereference, but application or library code can also explicitly throw a `NullPointerException`.

**Q3. Can we throw a runtime exception manually?**

Yes. Use `throw`, for example:

```java
throw new IllegalArgumentException("Invalid input");
```

**Q4. Does declaring `throws RuntimeException` become mandatory?**

No. It is legal to declare it, but the declaration is not required by the compiler.

**Q5. What happens when a runtime exception is not caught?**

It propagates up the call stack. If no handler catches it, the uncaught-exception mechanism handles it and the affected thread terminates.

**Q6. Can a method throw multiple runtime exception types?**

Yes. A method can throw different runtime exceptions, and they do not need to be listed in its `throws` clause.

## 10. Quick Revision

- `RuntimeException` extends `Exception`, which extends `Throwable`.
- Runtime exceptions are unchecked.
- They may be thrown by the JVM, Java libraries, or application code.
- `throw` explicitly throws an exception object.
- `try-catch` can handle runtime exceptions.
- Uncaught exceptions propagate through the call stack.
- Custom unchecked exceptions extend `RuntimeException`.

**One-line interview answer:** A runtime exception is an unchecked subclass of `RuntimeException` that represents an exceptional condition during execution. The JVM's exception-handling mechanism transfers control to a matching handler, or propagates the exception up the call stack if no handler is found.

---

# What Happens When an Exception Declared with `throws` Occurs in Java?

## 1. What Does `throws` Mean?

In Java, the `throws` keyword declares that a method may propagate one or more exceptions to its caller.

If an exception occurs inside the method and is not handled there, it propagates up the call stack until a matching `catch` block handles it. If no handler is found, it reaches the thread's uncaught-exception handler.

**Important:** Declaring an exception with `throws` does not throw the exception by itself. It declares the method's exception contract.

```java
public void myMethod() throws MyException {
    // Code that may throw MyException
}
```

Here, `MyException` is declared as a possible exception that the method can propagate.

## 2. Example: Exception Handled by the Caller

```java
class MyException extends Exception {
    public MyException(String message) {
        super(message);
    }
}

public class Main {

    static void myMethod() throws MyException {
        throw new MyException("Something went wrong");
    }

    public static void main(String[] args) {
        try {
            myMethod();
        } catch (MyException e) {
            System.out.println("Exception handled: " + e.getMessage());
        }

        System.out.println("Program continues");
    }
}
```

**Output:**

```text
Exception handled: Something went wrong
Program continues
```

### Execution flow

1. `main()` calls `myMethod()`.
2. `myMethod()` throws a `MyException`.
3. The exception propagates to the caller because `myMethod()` does not handle it.
4. The `catch` block in `main()` handles the exception.
5. Execution continues after the `try-catch` statement.

## 3. What If the Caller Does Not Handle the Exception?

If the caller neither catches a checked exception nor declares it in its own `throws` clause, the code generally fails to compile.

For an unchecked exception, such as `NullPointerException`, the caller is not required to catch or declare it.

If an exception is allowed to propagate at runtime and no matching handler exists anywhere up the call stack, the thread's uncaught-exception handler is invoked. The default handler typically prints a stack trace, and that thread terminates.

## 4. Does `throws` Automatically Handle an Exception?

No. `throws` only declares the possibility of an exception escaping the method.

Compare the two keywords:

| Keyword | Purpose |
|---|---|
| `throw` | Explicitly throws an exception object |
| `throws` | Declares exception types a method or constructor may propagate |
| `try-catch` | Handles an exception |
| `finally` | Runs cleanup code when control leaves the `try` statement, subject to JVM termination and other exceptional conditions |

Example:

```java
static void divide() throws ArithmeticException {
    throw new ArithmeticException("Division error");
}
```

The method declares `ArithmeticException` and explicitly throws it. Since `ArithmeticException` is unchecked, the `throws` declaration is optional.

## 5. What Happens With Checked Exceptions?

For checked exceptions, Java's compiler requires a method to catch the exception or declare it for propagation, unless the exception is handled by another applicable mechanism.

```java
import java.io.IOException;

public class Main {

    static void readFile() throws IOException {
        throw new IOException("File could not be read");
    }

    public static void main(String[] args) {
        try {
            readFile();
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Output:**

```text
File could not be read
```

The compiler requires `IOException` to be caught or declared because it is a checked exception.

## 6. One-Line Interview Answer

When an exception occurs in a method that declares it with `throws`, the exception propagates to the caller unless it is handled inside the method; the caller must catch or further declare checked exceptions, while unchecked exceptions do not require this.

---

