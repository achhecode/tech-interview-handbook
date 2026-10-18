---
title: String
parent: Core Java
nav_order: 4
---

# String in Java
{: .no_toc }


<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

# String in Java

## 1. What is a String in Java?

In Java, `String` is a **class used to represent a sequence of characters**. It belongs to the `java.lang` package, which is automatically imported into every Java program.

```java
String name = "Danish";
System.out.println(name);
```

Output:

```text
Danish
```

Strings are commonly used to represent names, messages, user input, file paths, URLs, and other textual data.

**Key point:** Java `String` objects are immutable, meaning their values cannot be changed after they are created.

## 2. Ways to Create a String

There are two commonly discussed ways to create a string.

### A. Using a string literal

```java
String s1 = "Hello";
String s2 = "Hello";

System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
```

Output:

```text
true
true
```

Java reuses identical string literals from the **String Constant Pool**. Therefore, `s1` and `s2` refer to the same pooled string object in this example.

### B. Using the `new` keyword

```java
String s1 = new String("Hello");
String s2 = new String("Hello");

System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
```

Output:

```text
false
true
```

Each `new String(...)` expression creates a distinct `String` object. The string literal `"Hello"` used as the constructor argument can still refer to an object in the String Constant Pool.

### String literal vs `new String()`

| Feature | String literal | `new String()` |
|---|---|---|
| Example | `"Hello"` | `new String("Hello")` |
| Reuses pooled strings | Yes, for identical literals | The created object is distinct |
| Object creation | May reuse an existing pooled object | Creates a new `String` object |
| Recommended for normal use | Yes | Usually unnecessary |

## 3. What Is the String Constant Pool?

The String Constant Pool is a JVM-managed pool that allows identical string literals to share the same string object.

```java
String a = "Java";
String b = "Java";
String c = new String("Java");

System.out.println(a == b);
System.out.println(a == c);
System.out.println(a.equals(c));
```

Output:

```text
true
false
true
```

Explanation:

- `a == b`: Both references point to the same pooled object.
- `a == c`: The references point to different objects.
- `a.equals(c)`: Both objects contain the same character sequence.

You can use `intern()` to obtain the canonical pooled reference for a string:

```java
String a = new String("Java");
String b = a.intern();

System.out.println(b == "Java");
```

Output:

```text
true
```

## 4. Why Is String Immutable?

An immutable object cannot have its state changed after construction.

```java
String s = "Hello";
s.concat(" World");

System.out.println(s);
```

Output:

```text
Hello
```

The `concat()` method returns a new string, but the returned value was not assigned to `s`.

Correct usage:

```java
String s = "Hello";
s = s.concat(" World");

System.out.println(s);
```

Output:

```text
Hello World
```

The original string was not modified. The variable `s` now references the string returned by `concat()`.

### Why did Java make String immutable?

1. **Security:** Strings are commonly used for file paths, URLs, class names, and configuration values. Immutability prevents their contents from changing unexpectedly.
2. **String pool:** Shared string objects can safely be reused when their values cannot change.
3. **Thread safety:** Immutable strings can be shared across threads without synchronization to protect their internal character data.
4. **Hashing and collections:** A string's hash code can be cached, and its contents remain stable when used as a `HashMap` key.
5. **Predictability:** Methods can use strings without worrying that another part of the program will modify their contents.

## 5. Common String Methods

| Method | Purpose | Example |
|---|---|---|
| `length()` | Returns the number of UTF-16 code units | `"Java".length()` → `4` |
| `charAt(index)` | Returns the character at an index | `"Java".charAt(1)` → `'a'` |
| `substring(begin)` | Returns a substring from an index | `"Java".substring(2)` → `"va"` |
| `substring(begin, end)` | Returns a substring up to, but excluding, `end` | `"Java".substring(1, 3)` → `"av"` |
| `indexOf(value)` | Finds the first matching position | `"Java".indexOf('a')` → `1` |
| `contains(value)` | Checks whether a sequence exists | `"Java".contains("av")` → `true` |
| `toUpperCase()` | Converts text to uppercase | `"Java".toUpperCase()` → `"JAVA"` |
| `toLowerCase()` | Converts text to lowercase | `"JAVA".toLowerCase()` → `"java"` |
| `trim()` | Removes leading and trailing characters at or below U+0020 | `" Java ".trim()` → `"Java"` |
| `strip()` | Removes leading and trailing Unicode whitespace | `" Java ".strip()` → `"Java"` |
| `equals(value)` | Compares string contents, case-sensitively | `"Java".equals("Java")` → `true` |
| `equalsIgnoreCase(value)` | Compares contents ignoring case differences | `"Java".equalsIgnoreCase("JAVA")` → `true` |
| `replace(old, new)` | Replaces matching characters or sequences | `"Java".replace('a', 'o')` → `"Jovo"` |
| `startsWith(value)` | Checks the prefix | `"Java".startsWith("Ja")` → `true` |
| `endsWith(value)` | Checks the suffix | `"Java".endsWith("va")` → `true` |
| `isEmpty()` | Checks whether length is zero | `"".isEmpty()` → `true` |
| `isBlank()` | Checks whether the string is empty or contains only whitespace | `"   ".isBlank()` → `true` |

**Interview note:** `strip()` and `isBlank()` were introduced in Java 11. `trim()` and `isEmpty()` are available in earlier Java versions.

Also, `length()` counts UTF-16 code units, not necessarily user-perceived characters. Some Unicode characters, such as many emoji, use two UTF-16 code units.

## 6. `==` vs `equals()` in String

This is one of the most frequently asked Java interview questions.

- `==` compares object references when used with objects.
- `equals()` compares string contents.

```java
String a = new String("Java");
String b = new String("Java");

System.out.println(a == b);
System.out.println(a.equals(b));
```

Output:

```text
false
true
```

**Best practice:** Use `equals()` when comparing string values.

For a potentially null variable, this is also safe:

```java
String input = null;

System.out.println("Java".equals(input)); // false
```

Calling `input.equals("Java")` would throw a `NullPointerException`.

## 7. String Concatenation

Strings can be concatenated using the `+` operator or the `concat()` method.

```java
String firstName = "A R";
String lastName = "Danish";

String fullName = firstName + " " + lastName;

System.out.println(fullName);
```

Output:

```text
A R Danish
```

For repeated concatenation inside a loop, prefer `StringBuilder` in typical single-threaded code to avoid creating many intermediate strings.

```java
StringBuilder builder = new StringBuilder();

for (int i = 1; i <= 3; i++) {
    builder.append(i).append(" ");
}

System.out.println(builder.toString().trim());
```

Output:

```text
1 2 3
```

## 8. String vs StringBuilder vs StringBuffer

| Feature | `String` | `StringBuilder` | `StringBuffer` |
|---|---|---|---|
| Mutable | No | Yes | Yes |
| Thread-safe for shared mutation | Safe to share immutable value | No | Its methods are synchronized |
| Typical use | Fixed text | Repeated modifications in one thread | Shared mutable text requiring synchronized methods |
| Performance for repeated appends | Often inefficient | Usually preferred | Synchronization adds overhead |

Example:

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");

System.out.println(sb);
```

Output:

```text
Hello World
```

## 9. Common Interview Questions

### Q1. Is String a primitive data type?

No. `String` is a reference type because it is a class. Types such as `int`, `char`, and `boolean` are primitive types.

### Q2. Is String thread-safe?

A `String` object is immutable, so its contents can safely be shared between threads. This does not automatically make a variable that references a string thread-safe when that reference is reassigned concurrently.

### Q3. Can we inherit the String class?

No. `String` is declared `final`, so it cannot be subclassed.

### Q4. Can a String contain `null`?

A `String` variable can reference `null`, but `null` is not a string value.

```java
String a = null;
String b = "null";

System.out.println(a == b);
System.out.println(b.length());
```

Output:

```text
false
4
```

Calling a method on `a` would throw a `NullPointerException`.

### Q5. What does `String.intern()` do?

It returns the canonical pooled string reference for the string value. If an appropriate pooled entry is already available, that reference is returned; otherwise, the value is added to the pool.

### Q6. Why should we avoid `==` for comparing strings?

Because it checks reference identity, not whether the two strings contain the same text. Use `equals()` for content comparison.

### Q7. Is String immutable because its reference is final?

No. A reference being `final` means that the variable cannot be reassigned. String immutability comes from the class's implementation and design, not from every String variable being declared `final`.

```java
final String name = "Java";
// name = "Spring"; // Compilation error
```

Without `final`, a variable can reference another string, even though neither string object can be modified.

## 10. Quick Revision

- `String` is a `final` class in `java.lang`.
- Strings represent sequences of characters and are immutable.
- Identical string literals can share objects through the String Constant Pool.
- `new String("Java")` creates a distinct `String` object.
- `==` compares references; `equals()` compares contents.
- Methods such as `concat()`, `replace()`, and `toUpperCase()` return strings rather than modifying the original.
- Use `StringBuilder` for repeated string modifications in typical single-threaded code.
- Use `StringBuffer` when synchronized mutable string operations are needed.
- `strip()` and `isBlank()` require Java 11 or later.

**Remember for interviews:** String immutability, the String Constant Pool, `==` vs `equals()`, and `String` vs `StringBuilder` are essential topics to understand thoroughly.


---


# Why Is String Immutable in Java?

## 1. What Does String Immutability Mean?

In Java, `String` is an immutable class. Once a `String` object is created, its character sequence cannot be changed.

```java
String name = "Java";

name.concat(" Programming");

System.out.println(name);
```

Output:

```text
Java
```

The `concat()` method returns a new string; it does not modify the original object.

To retain the result, assign it to a variable:

```java
String name = "Java";

name = name.concat(" Programming");

System.out.println(name);
```

Output:

```text
Java Programming
```

The variable `name` now references the returned string. The original `"Java"` object remains unchanged.

## 2. Why Is String Immutable?

Java String immutability provides several important benefits.

### 1. Security

Strings are commonly used to represent file paths, URLs, class names, database connection details, and configuration values.

Suppose an application validates a file path before opening a file. If the same string object could be modified unexpectedly after validation, the application might use a different path from the one it checked.

Immutability ensures that the character sequence of a validated `String` cannot change.

**Important:** Immutability does not make sensitive strings, such as passwords, completely secure. A string's contents cannot be overwritten through its public API, and sensitive values may remain in memory until garbage collection. For certain secrets, mutable character arrays can be preferable when explicit clearing is needed.

### 2. String Constant Pool and Memory Optimization

Java maintains a String Constant Pool so identical string literals can share the same object.

```java
String s1 = "Java";
String s2 = "Java";

System.out.println(s1 == s2);
```

Output:

```text
true
```

Both references point to the same pooled object.

If strings were mutable, changing the contents through `s1` could unexpectedly affect `s2`. Immutability allows Java to share pooled strings safely.

It also reduces unnecessary object creation when the same literals are reused.

### 3. Thread Safety

Immutable objects can be shared between threads without synchronization to protect their internal state.

```java
String message = "Hello";

Runnable task = () -> System.out.println(message);

Thread t1 = new Thread(task);
Thread t2 = new Thread(task);

t1.start();
t2.start();
```

Possible output:

```text
Hello
Hello
```

Both threads can read the same string without competing to modify its contents.

**Interview point:** String immutability makes the object safe to share, but it does not make every operation involving a string reference thread-safe. Concurrent reassignment of a shared variable still requires appropriate coordination.

### 4. Reliability in Class Loading and Resource Access

Java uses strings in many APIs that identify classes, resources, and configuration entries.

```java
Class<?> clazz = Class.forName("java.lang.String");
System.out.println(clazz.getName());
```

Output:

```text
java.lang.String
```

Because the string's contents cannot change, APIs can rely on the value passed to them remaining stable.

This helps prevent unexpected changes to names and paths caused by modifying a shared string object.

### 5. Reliable Hashing in HashMap and HashSet

Strings are frequently used as keys in hash-based collections.

```java
import java.util.HashMap;
import java.util.Map;

public class StringKeyExample {
    public static void main(String[] args) {
        Map<String, String> users = new HashMap<>();

        String key = "user101";
        users.put(key, "Danish");

        key = key.concat("_updated");

        System.out.println(users.get("user101"));
        System.out.println(users.get(key));
    }
}
```

Output:

```text
Danish
null
```

The reassignment changes which string the variable `key` references. It does not modify the original key stored in the map.

Because a String's contents cannot change, its `hashCode()` remains consistent throughout its lifetime. This is important for correctly locating entries in `HashMap` and `HashSet`.

If mutable objects are used as map keys, changing fields involved in `equals()` or `hashCode()` after insertion can make entries difficult to retrieve.

### 6. Predictable Behavior and Safe Sharing

Strings are passed between methods, stored in objects, and returned from APIs.

```java
public static String getGreeting() {
    return "Hello, Java";
}
```

A caller can safely retain the returned string without worrying that another method will modify that object's contents.

This makes program behavior easier to understand and reduces accidental side effects.

### 7. Efficient Hash Code Caching

A string's hash code depends on its contents. Since the contents never change, Java can cache the computed hash code in its implementation.

Repeated calls to `hashCode()` can then reuse the cached result after it has been computed, which is useful when strings are repeatedly used in hash-based collections.

This is an implementation optimization enabled by immutability, rather than a guarantee about the exact internal implementation across all Java versions.

## 3. What Happens When We Modify a String?

String methods that appear to modify text actually return a string containing the result.

```java
String original = "Hello";

String upper = original.toUpperCase();
String replaced = original.replace('H', 'Y');

System.out.println(original);
System.out.println(upper);
System.out.println(replaced);
```

Output:

```text
Hello
HELLO
Yello
```

The original string remains unchanged.

## 4. Why Does Java Provide StringBuilder and StringBuffer?

Immutability is useful for stable text, but repeatedly creating modified strings can be inefficient.

Java provides mutable alternatives for these situations.

| Feature | String | StringBuilder | StringBuffer |
|---|---|---|---|
| Mutable | No | Yes | Yes |
| Synchronized methods | Not applicable to immutable state | No | Yes |
| Typical use | Fixed text | Frequent modifications in one thread | Mutable text requiring synchronized method operations |
| Repeated appends | Can create intermediate strings | Usually efficient | Synchronization can add overhead |

Example:

```java
StringBuilder builder = new StringBuilder("Java");

builder.append(" ");
builder.append("Programming");

System.out.println(builder);
```

Output:

```text
Java Programming
```

`StringBuilder` modifies its internal buffer instead of creating a new immutable string for every append operation.

`StringBuffer` synchronizes its methods, but this does not automatically make a sequence of multiple calls atomic or make all surrounding application logic thread-safe.

## 5. Interview Questions and Answers

### Q1. What are the main reasons String is immutable?

The main reasons are:

- Security and reliable handling of names and paths.
- Safe sharing of strings through the String Constant Pool.
- Thread-safe sharing of immutable objects.
- Stable behavior when strings are used as hash-based collection keys.
- Predictable behavior when passing strings between methods.
- Opportunities for optimizations such as hash-code caching.

### Q2. Is String immutable because it is declared `final`?

No. `final` prevents a class from being subclassed; it does not, by itself, make an object's state immutable.

The `String` class is `final`, and its implementation is designed to prevent callers from changing the character sequence of an existing instance.

### Q3. Does `final String name = "Java"` make String immutable?

No. `final` on the variable prevents reassignment of that variable. String immutability is a property of the class itself.

```java
final String name = "Java";

// name = "Spring"; // Compilation error
```

Without `final`, the reference variable could be reassigned, but the original String object would still be immutable.

### Q4. Does String immutability mean Java never creates new strings?

No. Methods such as `concat()`, `replace()`, and `toUpperCase()` can return new string objects. Some operations may reuse an existing string when appropriate, but they never change the original string's contents.

### Q5. Why is String a good HashMap key?

Its contents remain stable, so its equality behavior and hash code remain consistent. This allows a map to locate the key reliably after insertion.

### Q6. Are all immutable objects automatically thread-safe?

Immutable objects can generally be shared safely when their state is properly constructed and safely published. However, immutability of one object does not make mutable objects it references, or shared variables holding its reference, automatically thread-safe.

## 6. Quick Revision

Remember these five key points for interviews:

1. **Security:** A String's contents cannot be changed unexpectedly after validation.
2. **String Pool:** Identical literals can share objects safely.
3. **Thread safety:** Multiple threads can read the same immutable string without synchronizing its internal state.
4. **Hash-based collections:** Stable contents mean a stable hash code and reliable key lookup.
5. **Performance:** String immutability enables safe sharing and implementation optimizations.

**One-line interview answer:** Java String is immutable to provide safe sharing, predictable behavior, security benefits, reliable hash-based collection keys, and efficient reuse through the String Constant Pool.

---

# Where Is a String Stored in Java?

## 1. Introduction

In Java, `String` objects are stored in **heap memory**, while string literals can be reused through the **String Constant Pool**.

The exact behavior depends on how the string is created:

- **String literal:** Java reuses an existing pooled string if an equal literal is already available; otherwise, it creates a pooled string object.
- **`new String()`:** Creates a distinct `String` object on the heap.
- **String manipulation:** Methods such as `concat()` may return a new string object when a different result is required.
- **`intern()`:** Returns the canonical pooled string for the same content.

**Important:** The String Constant Pool is not a separate memory area outside the heap. In modern Java implementations, pooled strings are stored in heap memory.

## 2. String Literal Pool

When you create a string using a literal, Java uses the String Constant Pool.

```java
String s1 = "Hello";
String s2 = "Hello";

System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
```

Output:

```text
true
true
```

Explanation:

1. Java encounters the literal `"Hello"`.
2. The JVM uses the pooled string associated with that literal.
3. When it encounters the same literal again, it reuses the pooled object.
4. Both variables reference the same object.

## 3. String Created Using `new`

When you use the `new` keyword, Java creates a distinct `String` object on the heap.

```java
String s1 = "Hello";
String s2 = new String("Hello");

System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
```

Output:

```text
false
true
```

Explanation:

- `s1` refers to the pooled string `"Hello"`.
- `s2` refers to a distinct object created by `new String("Hello")`.
- Both objects contain the same text, so `equals()` returns `true`.
- Their references differ, so `==` returns `false`.

Conceptual memory diagram:

<box border radius="lg" padding={3} gap={3}>
  <box background="surface-secondary" radius="md" padding={3} align="center" gap={2}>
    **Stack — reference variables**
    <row gap={5} justify="center">
      <box border radius="md" padding={2} align="center">
        `s1`
        <icon name="arrow-down" />
      </box>
      <box border radius="md" padding={2} align="center">
        `s2`
        <icon name="arrow-down" />
      </box>
    </row>
  </box>
  <grid columns={2} gap={2}>
    <grid-item>
      <box border radius="md" padding={3} align="center" gap={1}>
        **Pooled string**
        `"Hello"`
        <text color="secondary" size="xs">Referenced by s1</text>
      </box>
    </grid-item>
    <grid-item>
      <box border radius="md" padding={3} align="center" gap={1}>
        **Heap object**
        `"Hello"`
        <text color="secondary" size="xs">Referenced by s2</text>
      </box>
    </grid-item>
  </grid>
</box>

The literal `"Hello"` inside the constructor argument can itself refer to the pooled string. Thus, `new String("Hello")` may involve the pooled object and an additional distinct object.

## 4. What Happens When a String Is Modified?

Strings are immutable. Methods such as `concat()` return the resulting string rather than changing the original object.

```java
String s1 = "Hello";
String s2 = s1.concat(" World");

System.out.println(s1);
System.out.println(s2);
System.out.println(s1 == s2);
```

Output:

```text
Hello
Hello World
false
```

Here:

- `s1` continues to reference `"Hello"`.
- `s2` references the result `"Hello World"`.
- The result is a distinct string because its contents differ from the original.

A new object is not guaranteed for every string operation. Java implementations can reuse strings when appropriate, such as when a transformation produces an unchanged result.

## 5. What Does `intern()` Do?

The `intern()` method returns the canonical pooled string for a given sequence of characters.

```java
String s1 = new String("Java");
String s2 = s1.intern();
String s3 = "Java";

System.out.println(s1 == s2);
System.out.println(s2 == s3);
System.out.println(s1.equals(s3));
```

Output:

```text
false
true
true
```

Explanation:

- `s1` references the distinct object created with `new`.
- `s2` references the canonical pooled string.
- `s3` references the same pooled string as `s2`.
- `s1` and `s3` have equal contents but are different objects.

**Interview point:** `intern()` does not move the original object into the pool. It returns the pooled canonical reference for the same content.

## 6. Stack vs Heap vs String Constant Pool

| Concept | Purpose |
|---|---|
| Stack | Holds method-call frames and local variables or references, subject to JVM implementation and optimization |
| Heap | Stores objects, including String objects |
| String Constant Pool | Enables reuse of canonical strings; it is associated with heap storage in modern Java |

Do not say that string literals are stored in the stack. A local variable such as `String name` holds a reference, while the `String` object itself resides on the heap.

## 7. Interview Questions

### Q1. Are String literals stored in the stack or heap?

The string objects are stored in heap memory. The String Constant Pool enables literals to reuse canonical string objects.

### Q2. Does `new String("Hello")` create one object or two?

It guarantees a distinct `String` object for the `new` expression. The literal `"Hello"` may also refer to a pooled object, so two objects can be involved if the pooled string already exists or must be established. The exact number of allocations depends on whether the pooled string already exists and on JVM implementation details.

### Q3. Does `concat()` always create a new object?

No. It returns a string containing the result. If the result differs from the original, it needs a distinct string value; if no change is required, an implementation may reuse an existing string.

### Q4. Does `intern()` move a string object into the pool?

No. It returns the canonical pooled reference for that content. The original object remains a distinct object if it was created separately.

### Q5. Is the String Constant Pool part of heap memory?

In modern Java, yes. It is not a separate memory area outside the heap. Older explanations sometimes describe the pool differently because its implementation location changed between Java versions.

## 8. Quick Revision

- String objects reside in heap memory.
- String literals can share objects through the String Constant Pool.
- `new String("Hello")` creates a distinct String object.
- `==` compares references; `equals()` compares content.
- String operations do not mutate the original object.
- `intern()` returns the canonical pooled reference.
- The String Constant Pool is associated with heap memory in modern Java.

**One-line interview answer:** Java String objects are stored on the heap; string literals can reuse canonical objects from the String Constant Pool, while `new String()` creates a distinct object.


---

# String Memory Storage and Alternatives in Java

## 1. Where Is a String Stored in Java?

In Java, a `String` object is stored in **heap memory**, while a reference identifies that object. The reference's storage location depends on where the variable is declared and how the JVM implements it.

Consider this example:

```java
public class StringMemoryExample {
    public static void main(String[] args) {
        String myString = "Hello, world!";
    }
}
```

There are two important concepts:

1. **String object:** The object containing the text `"Hello, world!"` resides in heap memory. Because the text is a literal, Java can reuse its pooled representation.
2. **Reference variable:** `myString` holds a reference to that object. As a local variable, it is conceptually associated with the current method's stack frame, although JVM optimizations can change the physical representation.

### Conceptual memory diagram

<box border radius="lg" padding={3} gap={3}>
  <box background="surface-secondary" radius="md" padding={3} gap={2}>
    **Stack — `main()` method frame**
    <box border radius="md" padding={2} align="center">
      `myString` — reference
      <icon name="arrow-down" />
    </box>
  </box>
  <box align="center">
    <icon name="arrow-down" size="lg" />
  </box>
  <box border={{size:1,color:"#0D9488"}} radius="md" padding={3} align="center" gap={1}>
    **Heap — String Constant Pool**
    <title size="lg">"Hello, world!"</title>
    <text color="secondary" size="sm">String object</text>
  </box>
</box>

This diagram is conceptual. The String Constant Pool is associated with heap memory in modern Java, and the JVM does not guarantee that every local reference physically resides on the stack.

## 2. Where Is the Reference Stored?

The reference's location depends on how the variable is declared.

| Variable type | Where the reference is conceptually stored |
|---|---|
| Local variable | In the method's stack frame |
| Instance field | As part of the containing object on the heap |
| Static field | As part of class-level runtime state; its exact physical storage is JVM-dependent |
| Method parameter | In the method's frame, conceptually |

### Example A: Local variable

```java
void printName() {
    String name = "Danish";
    System.out.println(name);
}
```

`name` is a local reference variable. Conceptually, it belongs to the method's stack frame, while the string object is on the heap.

### Example B: Instance variable

```java
class Employee {
    String name = "Danish";
}

public class Main {
    public static void main(String[] args) {
        Employee employee = new Employee();
    }
}
```

Conceptually:

- The `Employee` object is on the heap.
- Its `name` field holds a reference to the string object.
- The local variable `employee` holds a reference to the `Employee` object.

```text
Stack
  employee ───────────────┐
                          ↓
Heap                  Employee object
                      └── name ──────────┐
                                         ↓
Heap / String Pool                  "Danish"
```

**Interview tip:** Do not say that all references are stored on the stack. Local variables are commonly explained this way, but object fields reside within their containing objects, and static-field storage is JVM-dependent.

## 3. What Happens When We Use `new String()`?

```java
String s1 = "Java";
String s2 = new String("Java");
```

Conceptually, there are two distinct string objects:

- A pooled `"Java"` string referenced by `s1`.
- A separate heap object created by `new String("Java")`, referenced by `s2`.

```java
System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
```

Output:

```text
false
true
```

The `==` operator compares references, while `equals()` compares string contents.

## 4. What Are the Alternatives to String?

Java provides several alternatives, but they serve different purposes. `StringBuilder`, `StringBuffer`, and `char[]` are not interchangeable in every situation.

### A. StringBuilder

`StringBuilder` is a **mutable character sequence**. It is useful when text must be modified repeatedly.

```java
public class StringBuilderExample {
    public static void main(String[] args) {
        StringBuilder builder = new StringBuilder("Hello");

        builder.append(" ");
        builder.append("Java");

        System.out.println(builder);
    }
}
```

Output:

```text
Hello Java
```

Advantages:

- Mutable; append and insert operations can modify its buffer.
- Usually efficient for repeated string concatenation.
- Does not synchronize its methods, making it suitable for typical single-threaded operations.

**Correction:** `StringBuilder` is not thread-safe for concurrent mutation.

### B. StringBuffer

`StringBuffer` is also mutable, but its methods are synchronized.

```java
public class StringBufferExample {
    public static void main(String[] args) {
        StringBuffer buffer = new StringBuffer("Hello");

        buffer.append(" Java");

        System.out.println(buffer);
    }
}
```

Output:

```text
Hello Java
```

Advantages:

- Synchronizes its methods.
- Can be useful when mutable character data is shared across threads and synchronized method operations are appropriate.

Its synchronized methods can add overhead. Synchronization of individual methods does not automatically make a multi-call workflow atomic.

### C. Character array (`char[]`)

A character array stores individual UTF-16 code units. It is useful when you need direct control over character data, including clearing it after use.

```java
import java.util.Arrays;

public class CharArrayExample {
    public static void main(String[] args) {
        char[] characters = {'J', 'a', 'v', 'a'};

        System.out.println(characters);
        System.out.println(characters.length);

        Arrays.fill(characters, '\0');
    }
}
```

Output:

```text
Java
4
```

Advantages:

- Mutable individual elements.
- Can be overwritten when the data is no longer needed.
- May be useful for handling sensitive character data.

Limitations:

- Does not provide all the convenient text-processing methods of `String`.
- Clearing an array helps remove its contents from that array, but cannot guarantee that no copies of the data exist elsewhere.

For sensitive values such as passwords, APIs that accept `char[]` may be preferable when they allow you to clear the array after use.

## 5. String vs StringBuilder vs StringBuffer vs char[]

| Feature | String | StringBuilder | StringBuffer | `char[]` |
|---|---|---|---|---|
| Mutable | No | Yes | Yes | Yes |
| Built-in text methods | Extensive | Extensive | Extensive | Limited |
| Synchronizes its operations | Not needed for immutable state | No | Yes, for synchronized methods | No |
| Best suited for | Fixed text | Repeated text modification in one thread | Mutable text with synchronized method operations | Direct character access or overwriting data |
| Convenient conversion to text | Already a string | `toString()` | `toString()` | `new String(chars)` |

## 6. Which One Should You Use?

- Use **`String`** for ordinary text that does not need in-place modification.
- Use **`StringBuilder`** for repeated concatenation, especially inside loops.
- Use **`StringBuffer`** when its synchronized method behavior is specifically useful.
- Use **`char[]`** when direct character manipulation or overwriting data is important.

Example of efficient repeated concatenation:

```java
StringBuilder result = new StringBuilder();

for (int i = 1; i <= 5; i++) {
    result.append(i).append(" ");
}

System.out.println(result.toString().trim());
```

Output:

```text
1 2 3 4 5
```

## 7. Interview Questions

### Q1. Are all Java objects stored in heap memory?

Ordinarily, Java objects are conceptually allocated on the heap. However, a JIT compiler may optimize an allocation away or represent an object differently when its identity and behavior permit it. Avoid treating a simplified memory diagram as a guarantee of physical layout.

### Q2. Is a reference itself an object?

No. A reference is a value used to identify or access an object. It is not the object itself.

### Q3. Are references always stored on the stack?

No. Local references are conceptually associated with stack frames, while instance references are fields within objects and static references belong to class-level state. The JVM may optimize how these values are represented.

### Q4. Is StringBuilder thread-safe?

No. `StringBuilder` does not synchronize its operations. Concurrent modifications to the same instance require external coordination.

### Q5. Is StringBuffer always the best option for multithreading?

No. Its methods are synchronized, but the correct choice depends on the use case. Immutable `String` objects can be shared safely, and a separate `StringBuilder` per thread may be more appropriate for building text independently.

## 8. Quick Revision

- String objects are stored in heap memory.
- String literals can reuse objects through the String Constant Pool.
- Local references are conceptually associated with stack frames.
- Instance references are fields within their containing objects.
- Static-field storage details depend on the JVM implementation.
- `StringBuilder` is mutable and not thread-safe.
- `StringBuffer` is mutable and synchronizes its methods.
- `char[]` allows direct character access and overwriting.

**One-line interview answer:** A Java String object resides in heap memory, while the reference's location depends on the variable's scope and representation. Use `StringBuilder` for efficient repeated modifications, `StringBuffer` when synchronized method operations are needed, and `char[]` when direct control over character data is important.


---

# How to Create a Custom Immutable Class in Java

## 1. What Is an Immutable Class?

An immutable class is a class whose object's state cannot be changed after the object has been constructed.

Java's `String` class is a well-known example of an immutable class.

```java
String name = "Java";
name.concat(" Programming");

System.out.println(name);
```

Output:

```text
Java
```

The original String object remains unchanged.

We can create our own immutable class that follows similar principles.

## 2. Rules for Creating an Immutable Class

Follow these guidelines when designing an immutable class:

1. Declare the class `final` to prevent subclassing that could introduce mutability.
2. Make all instance fields `private` and `final`.
3. Initialize fields through the constructor.
4. Do not provide setter methods.
5. Do not expose mutable internal objects directly.
6. Make defensive copies of mutable inputs and outputs when necessary.
7. Ensure that methods do not modify the object's state.
8. Override `equals()` and `hashCode()` when value-based equality is required.

**Important:** Declaring fields `final` alone does not guarantee immutability. A final reference can still point to a mutable object.

## 3. Example: Creating a Custom Immutable String Class

Let's create a class named `ImmutableString` that wraps a Java `String`.

```java
import java.util.Objects;

public final class ImmutableString {

    private final String value;

    public ImmutableString(String value) {
        this.value = Objects.requireNonNull(value, "value cannot be null");
    }

    public String getValue() {
        return value;
    }

    @Override
    public String toString() {
        return value;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof ImmutableString)) {
            return false;
        }

        ImmutableString other = (ImmutableString) obj;
        return value.equals(other.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}
```

### Explanation

- `final class`: Prevents subclasses from introducing mutable state or changing behavior.
- `private final String value`: Stores the text and prevents reassignment of the field after construction.
- Constructor: Initializes the field once and rejects `null`.
- `getValue()`: Returns the String, which is itself immutable.
- `toString()`: Returns the stored text.
- `equals()`: Compares the text values.
- `hashCode()`: Produces a hash code consistent with `equals()`.

There is no setter, and no method changes the stored value.

## 4. Test the Immutable Class

```java
public class Main {
    public static void main(String[] args) {

        ImmutableString s1 = new ImmutableString("Java");
        ImmutableString s2 = new ImmutableString("Java");
        ImmutableString s3 = new ImmutableString("Python");

        System.out.println(s1);
        System.out.println(s1.equals(s2));
        System.out.println(s1.equals(s3));
        System.out.println(s1.hashCode() == s2.hashCode());
    }
}
```

Output:

```text
Java
true
false
true
```

Both `s1` and `s2` contain the same text, so they are equal by value even though they are separate objects.

## 5. What If the Class Contains Mutable Fields?

This is a common interview trap.

Consider a class that stores a `Date` object:

```java
import java.util.Date;

public final class EmployeeRecord {

    private final Date joiningDate;

    public EmployeeRecord(Date joiningDate) {
        this.joiningDate = joiningDate;
    }

    public Date getJoiningDate() {
        return joiningDate;
    }
}
```

Although the field is `final`, this class is **not truly immutable**. The caller can modify the original `Date` after construction or mutate the object returned by the getter.

### Correct implementation using defensive copies

```java
import java.util.Date;
import java.util.Objects;

public final class EmployeeRecord {

    private final Date joiningDate;

    public EmployeeRecord(Date joiningDate) {
        this.joiningDate =
                new Date(Objects.requireNonNull(joiningDate).getTime());
    }

    public Date getJoiningDate() {
        return new Date(joiningDate.getTime());
    }
}
```

Now:

- The constructor copies the supplied `Date`.
- The getter returns a new copy rather than exposing the internal object.
- External code cannot change the internal date through either reference.

For new Java applications, immutable types such as `LocalDate` from `java.time` are often preferable to mutable `Date`.

## 6. Why Make the Class `final`?

If a class is not final, a subclass may introduce mutable fields or override methods in ways that undermine the intended behavior.

```java
public class Parent {
    private final String value;

    public Parent(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}
```

A subclass could add a mutable field or override a non-final method. Declaring the class `final` is a straightforward way to prevent such extensions.

An immutable design can also be achieved without `final` under stricter inheritance rules, but making the class final is usually the simplest approach.

## 7. Is `final` Enough to Make a Class Immutable?

No.

```java
final class Example {

    private final StringBuilder builder;

    Example(StringBuilder builder) {
        this.builder = builder;
    }

    public StringBuilder getBuilder() {
        return builder;
    }
}
```

This class is final, and its field is final, but the state can still change:

```java
StringBuilder builder = new StringBuilder("Java");
Example example = new Example(builder);

builder.append(" Programming");

System.out.println(example.getBuilder());
```

Output:

```text
Java Programming
```

The constructor retained the caller's mutable object, and the getter exposes it.

To make this class immutable, copy the input and avoid exposing the mutable internal object:

```java
final class ImmutableText {

    private final StringBuilder builder;

    ImmutableText(StringBuilder builder) {
        this.builder = new StringBuilder(builder);
    }

    public String getText() {
        return builder.toString();
    }
}
```

Because the mutable field is private and never modified after construction, and the getter returns an immutable String, the internal state cannot be changed through the public API. In practice, storing a `String` directly would be simpler if mutable behavior is unnecessary.

## 8. Common Interview Questions

### Q1. Can we create our own immutable String class?

Yes. Create a final class with private final fields, initialize them in the constructor, avoid setters, and prevent mutable state from escaping.

### Q2. Why should fields be private?

Private fields prevent callers from accessing and modifying the internal state directly.

### Q3. Why should fields be final?

A final field can be assigned only once after initialization. This helps preserve the object's state, although mutable objects referenced by final fields still need protection.

### Q4. Why override `equals()` and `hashCode()`?

They enable value-based comparison and correct behavior in hash-based collections such as `HashMap` and `HashSet`. They are not mandatory for immutability itself.

### Q5. Is a class immutable if it has no setters?

Not necessarily. Its methods might modify fields, mutable inputs might be retained, or getters might expose mutable internal objects.

### Q6. Is a record immutable in Java?

A record has final component fields, but it is only shallowly immutable. If a component refers to a mutable object, defensive copying may still be necessary.

## 9. Quick Revision

To create an immutable class:

- Declare the class `final`.
- Make instance fields `private final`.
- Initialize state through the constructor.
- Avoid setters and state-changing methods.
- Defensively copy mutable inputs and outputs.
- Do not expose mutable internal state.
- Implement `equals()` and `hashCode()` when value equality is needed.

**One-line interview answer:** An immutable class is created by preventing changes to an object's state after construction through final class design, private final fields, constructor initialization, and defensive copying of mutable objects.


---

# String vs StringBuffer vs StringBuilder in Java

## 1. Introduction

Java provides three commonly used classes for working with text:

- `String` — an immutable sequence of characters.
- `StringBuffer` — a mutable character sequence with synchronized methods.
- `StringBuilder` — a mutable character sequence without synchronized methods.

The main differences are **mutability, thread safety, performance, and use cases**.

## 2. Key Differences

| Feature | String | StringBuffer | StringBuilder |
|---|---|---|---|
| Mutability | Immutable | Mutable | Mutable |
| Thread safety | Safe to share its immutable contents | Synchronizes its methods | Not thread-safe for concurrent mutation |
| Synchronization | Not required to protect its internal state | Yes, at method level | No |
| Performance for repeated modifications | Often less efficient | Synchronization can add overhead | Usually fastest for single-threaded modifications |
| Introduced | Java 1.0 | Java 1.0 | Java 1.5 |
| Common use case | Fixed or infrequently modified text | Shared mutable text requiring synchronized method operations | Repeated text modifications in one thread |

**Interview tip:** `StringBuffer` is not automatically the best choice whenever an application is multithreaded. If each thread builds its own text, `StringBuilder` is usually appropriate.

## 3. String — Immutable

Once a String object is created, its character sequence cannot be changed. Operations that produce different text return a result rather than modifying the original object.

```java
public class StringExample {
    public static void main(String[] args) {
        String text = "Hello";

        text.concat(" Java");

        System.out.println(text);

        text = text.concat(" Java");

        System.out.println(text);
    }
}
```

Output:

```text
Hello
Hello Java
```

Explanation:

1. `concat()` returns `"Hello Java"`.
2. The first result is discarded because it is not assigned.
3. The second result is assigned to `text`.
4. The original `"Hello"` object remains unchanged.

### When should you use String?

- Storing names, addresses, messages, and configuration values.
- Comparing text or passing it between methods.
- Representing fixed text.
- Using text as keys in `HashMap` or elements in `HashSet`.

## 4. StringBuffer — Mutable and Synchronized

`StringBuffer` allows its character sequence to be modified. Its methods are synchronized, which helps protect individual operations when multiple threads access the same instance.

```java
public class StringBufferExample {
    public static void main(String[] args) {
        StringBuffer buffer = new StringBuffer("Hello");

        buffer.append(" ");
        buffer.append("Java");
        buffer.insert(5, "World ");

        System.out.println(buffer);
    }
}
```

Output:

```text
HelloWorld Java
```

Explanation:

- `append()` adds text to the end.
- `insert()` inserts text at a specified position.
- The same mutable object is modified.

### When should you use StringBuffer?

Use it when multiple threads genuinely share and modify the same character buffer and synchronized method operations are useful.

**Important:** Synchronization of individual methods does not make a sequence of operations automatically atomic.

For example, if a logical operation requires several calls, another thread may run between those calls unless the entire sequence is protected by an appropriate lock.

## 5. StringBuilder — Mutable Without Synchronization

`StringBuilder` works similarly to `StringBuffer`, but its methods do not synchronize access to the internal buffer.

```java
public class StringBuilderExample {
    public static void main(String[] args) {
        StringBuilder builder = new StringBuilder("Hello");

        builder.append(" ");
        builder.append("Java");
        builder.insert(5, "World ");

        System.out.println(builder);
    }
}
```

Output:

```text
HelloWorld Java
```

The result is the same as the `StringBuffer` example.

### When should you use StringBuilder?

- Building strings inside loops.
- Constructing SQL statements or messages.
- Formatting text through multiple append operations.
- Building text locally within a method.
- Creating output independently in each thread.

For most repeated string modifications in ordinary single-threaded code, `StringBuilder` is the preferred choice.

## 6. Performance Comparison

Consider building a string containing numbers from 1 to 1,000.

### Using String

```java
String result = "";

for (int i = 1; i <= 1000; i++) {
    result = result + i;
}
```

Repeated concatenation can create many intermediate strings, making this approach inefficient.

### Using StringBuilder

```java
StringBuilder result = new StringBuilder();

for (int i = 1; i <= 1000; i++) {
    result.append(i);
}

String output = result.toString();
```

`StringBuilder` reuses its mutable buffer, growing its capacity when needed. This is generally more efficient for repeated appends.

### Using StringBuffer

```java
StringBuffer result = new StringBuffer();

for (int i = 1; i <= 1000; i++) {
    result.append(i);
}

String output = result.toString();
```

This also avoids repeatedly creating intermediate immutable strings, but synchronized methods can add overhead.

The exact performance difference depends on the workload, Java version, and compiler optimizations. Modern Java compilers can optimize some concatenation expressions, especially fixed concatenations.

## 7. Thread-Safety Example

Suppose two threads append text to the same shared object.

```java
StringBuilder builder = new StringBuilder();

Thread t1 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        builder.append("A");
    }
});

Thread t2 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        builder.append("B");
    }
});

t1.start();
t2.start();
```

This is unsafe because `StringBuilder` does not support concurrent mutation without synchronization. The resulting contents are not guaranteed to be correct.

Changing the declaration to:

```java
StringBuffer buffer = new StringBuffer();
```

and having both threads append to `buffer` uses synchronized append operations. Each individual append is protected, but any larger multi-operation workflow still needs appropriate coordination.

### A better alternative for independent work

If each thread builds its own string, use a separate `StringBuilder` per thread. This avoids unnecessary synchronization and shared mutable state.

## 8. Common Methods

All three classes support many similar operations, although their APIs are not identical.

| Operation | Example |
|---|---|
| Append | `builder.append("Java")` |
| Insert | `builder.insert(0, "Hello ")` |
| Delete | `builder.delete(0, 5)` |
| Replace | `builder.replace(0, 4, "Java")` |
| Reverse | `builder.reverse()` |
| Length | `builder.length()` |
| Capacity | `builder.capacity()` |
| Convert to String | `builder.toString()` |

`append()`, `insert()`, `delete()`, and `replace()` generally mutate a `StringBuilder` or `StringBuffer` and return the same builder object for method chaining.

## 9. Frequently Asked Interview Questions

### Q1. What is the main difference between String, StringBuffer, and StringBuilder?

`String` is immutable, while `StringBuffer` and `StringBuilder` are mutable. `StringBuffer` synchronizes its methods; `StringBuilder` does not.

### Q2. Which is faster: StringBuffer or StringBuilder?

`StringBuilder` is generally faster for single-threaded modifications because it avoids synchronization overhead.

### Q3. Is String thread-safe?

A String object's contents cannot be changed after construction, so they can be shared safely. However, a shared variable referencing a String may still be reassigned concurrently, requiring synchronization or another suitable concurrency mechanism.

### Q4. Why was StringBuilder introduced?

It was introduced in Java 1.5 as a mutable alternative to `StringBuffer` without synchronization overhead, improving performance for single-threaded text construction.

### Q5. Does StringBuffer guarantee thread safety for every sequence of operations?

No. Its individual methods are synchronized, but multiple calls that together form one logical operation may require additional synchronization.

### Q6. Can we convert StringBuilder to String?

Yes, using `toString()`:

```java
StringBuilder builder = new StringBuilder("Java");
String text = builder.toString();
```

The resulting String is immutable.

### Q7. Does the `+` operator always create a new String object?

Not necessarily. Java may optimize concatenation, particularly when operands are known at compile time. However, repeated concatenation in loops can still create unnecessary intermediate objects, so `StringBuilder` is usually preferred.

## 10. Quick Revision

- **String:** Immutable; best for ordinary text.
- **StringBuffer:** Mutable; synchronizes its methods.
- **StringBuilder:** Mutable; does not synchronize its methods.
- Prefer `StringBuilder` for repeated modifications in single-threaded code.
- Use `StringBuffer` only when its synchronized operations are useful for shared mutable text.
- Use `String` for values that do not need in-place modification.

**One-line interview answer:** String is immutable, StringBuffer is mutable with synchronized methods, and StringBuilder is mutable without synchronization and is generally preferred for single-threaded string manipulation.

---

# Is StringBuffer Synchronized? Where Is Synchronization Used?

## 1. Is StringBuffer Synchronized?

Yes. `StringBuffer` is designed for thread-safe operations on a mutable character sequence. Its methods that access or modify the buffer are synchronized.

Synchronization allows a thread to acquire the object's monitor before executing a synchronized instance method. Another thread attempting to execute a synchronized method on the same object must wait until the monitor is released.

**Important:** This protects individual method calls. It does not automatically make a sequence of multiple method calls atomic.

## 2. Where Is `synchronized` Used in StringBuffer?

The JDK's `StringBuffer` implementation declares many instance methods with the `synchronized` keyword.

For example, simplified versions of its methods look like this:

```java
public synchronized StringBuffer append(String str) {
    // Append characters to the internal buffer.
    return this;
}

public synchronized StringBuffer insert(int index, String str) {
    // Insert characters into the internal buffer.
    return this;
}

public synchronized StringBuffer delete(int start, int end) {
    // Delete characters from the internal buffer.
    return this;
}

public synchronized char charAt(int index) {
    // Return the character at the specified index.
    return value[index];
}
```

These are illustrative examples, not the complete JDK source implementation.

When a thread calls a synchronized instance method, it acquires the monitor associated with that particular `StringBuffer` object.

For example:

```java
StringBuffer buffer = new StringBuffer("Java");

buffer.append(" Programming");
```

The thread acquires `buffer`'s monitor before executing the synchronized `append()` method and releases it when the method exits, including when it exits by throwing an exception.

### Common synchronized methods

Examples include:

- `append()`
- `insert()`
- `delete()`
- `deleteCharAt()`
- `replace()`
- `setCharAt()`
- `setLength()`
- `reverse()`
- `charAt()`
- `length()`
- `capacity()`
- `ensureCapacity()`
- `trimToSize()`
- `toString()`

The exact declarations and implementation details should be checked against the JDK version being discussed. Do not assume that every inherited method or every method returning a value is synchronized.

## 3. How Does Synchronization Work Internally?

Consider two threads appending text to the same `StringBuffer`.

```java
StringBuffer buffer = new StringBuffer();

Thread t1 = new Thread(() -> buffer.append("Hello"));
Thread t2 = new Thread(() -> buffer.append("World"));

t1.start();
t2.start();
```

Conceptually, execution could look like this:

<box border radius="lg" padding={3} gap={3}>
  <box background="surface-secondary" radius="md" padding={3} gap={1}>
    **Thread 1**
    <text>Requests the buffer's monitor</text>
    <text>Acquires the monitor</text>
    <text>Executes `append("Hello")`</text>
    <text>Releases the monitor</text>
  </box>
  <box align="center">
    <icon name="arrow-down" size="xl" />
    <text color="secondary" size="sm">Monitor becomes available</text>
  </box>
  <box background="surface-secondary" radius="md" padding={3} gap={1}>
    **Thread 2**
    <text>Acquires the monitor when available</text>
    <text>Executes `append("World")`</text>
    <text>Releases the monitor</text>
  </box>
</box>

The final buffer contains both appended strings, but their order is not guaranteed. The output could be `HelloWorld` or `WorldHello`.

The key point is that two threads cannot simultaneously execute synchronized instance methods on the same object while both hold that object's monitor.

However, threads can operate concurrently on different `StringBuffer` instances because each object has its own monitor.

## 4. Does StringBuffer Make Multiple Operations Atomic?

No. Synchronization protects individual method calls, not necessarily a whole sequence.

For example:

```java
StringBuffer buffer = new StringBuffer("A");

if (buffer.length() == 1) {
    buffer.append("B");
}
```

Each call to `length()` and `append()` is synchronized in the usual JDK implementation. But another thread can modify `buffer` between the two calls.

If the condition and append must be treated as one indivisible operation, synchronize the entire sequence:

```java
synchronized (buffer) {
    if (buffer.length() == 1) {
        buffer.append("B");
    }
}
```

This works because the synchronized block and the synchronized instance methods use the same object's monitor.

**Interview point:** Thread-safe individual methods do not automatically guarantee thread safety for compound operations.

## 5. StringBuffer vs StringBuilder

| Feature | StringBuffer | StringBuilder |
|---|---|---|
| Mutable | Yes | Yes |
| Instance methods synchronized | Generally, for buffer operations | No |
| Concurrent modification of one shared instance | Individual synchronized operations are protected | Requires external coordination |
| Performance | Synchronization may add overhead | Usually preferred for local, single-threaded building |
| Introduced | Java 1.0 | Java 1.5 |

### StringBuffer example

```java
StringBuffer buffer = new StringBuffer("Java");
buffer.append(" 21");

System.out.println(buffer);
```

### StringBuilder example

```java
StringBuilder builder = new StringBuilder("Java");
builder.append(" 21");

System.out.println(builder);
```

Both print:

```text
Java 21
```

Use `StringBuilder` when one thread owns the buffer. Use `StringBuffer` when its synchronized method operations are useful for shared mutable text. For more complex shared operations, additional synchronization may still be necessary.

## 6. Frequently Asked Interview Questions

### Q1. Where is the synchronized keyword used in StringBuffer?

Many of its instance methods, including `append()`, `insert()`, `delete()`, and `charAt()`, are synchronized in standard JDK implementations.

### Q2. Does StringBuffer use a class-level lock?

No. Synchronized instance methods use the monitor of the particular object on which they are called, not the `StringBuffer.class` monitor.

### Q3. Can two threads call methods on different StringBuffer objects simultaneously?

Yes. Each instance has its own monitor, so the synchronized methods on separate instances do not block one another merely because they belong to the same class.

### Q4. Is StringBuffer completely thread-safe?

Its synchronized methods protect individual operations on a shared instance. Compound operations and interactions with external state may require additional synchronization.

### Q5. Why is StringBuilder generally faster?

Its methods do not acquire a monitor for synchronization. This makes it a suitable choice for local string construction where concurrent mutation is not required.

## 7. Quick Revision

- `StringBuffer` is mutable and provides synchronized buffer operations.
- Synchronized instance methods use the monitor of the current object.
- Only one thread at a time can hold that object's monitor.
- Different instances can be used concurrently.
- Individual synchronized methods do not make multi-step operations atomic.
- `StringBuilder` avoids synchronization overhead and is usually preferred for local string construction.

**One-line interview answer:** StringBuffer uses synchronized instance methods to protect individual operations on its internal character buffer; its methods use the object's monitor, but compound operations may still need explicit synchronization.


---

# Why Can Java Substrings Be Problematic?

## 1. What Is `substring()` in Java?

The `substring()` method extracts part of a string and returns it as another `String`.

```java
public class SubstringExample {
    public static void main(String[] args) {
        String text = "Hello Java";

        System.out.println(text.substring(6));
        System.out.println(text.substring(0, 5));
    }
}
```

Output:

```text
Java
Hello
```

Java provides two commonly used overloads:

- `substring(beginIndex)` — extracts from `beginIndex` to the end.
- `substring(beginIndex, endIndex)` — extracts from `beginIndex` up to, but excluding, `endIndex`.

## 2. Why Are Java Substrings Sometimes Considered Bad?

The `substring()` method is not inherently bad. Problems can arise from how it is used, particularly in performance-sensitive code or when processing untrusted input.

### 1. Repeated substring operations can create many objects

A substring whose contents differ from the original string generally requires a separate `String` object and its own representation of the resulting text in modern Java implementations.

Consider this example:

```java
String text = "Hello Java Programming";

for (int i = 0; i < text.length(); i++) {
    System.out.println(text.substring(i));
}
```

This creates many substring results. If this pattern is repeated over very large strings, it can cause unnecessary allocations and garbage-collection pressure.

**Better approach:** If the goal is to inspect characters, use `charAt()` rather than creating a substring each time.

```java
String text = "Hello Java Programming";

for (int i = 0; i < text.length(); i++) {
    System.out.println(text.charAt(i));
}
```

Use `StringBuilder` when constructing a new string incrementally. It is not a universal replacement for `substring()` when extraction is actually required.

### 2. Invalid indices cause exceptions

The `substring()` method throws `IndexOutOfBoundsException` if the indices are invalid.

```java
String text = "Java";

System.out.println(text.substring(1, 5));
```

This throws an exception because the string length is `4`, so index `5` is out of range.

Valid indices for `substring(beginIndex, endIndex)` must satisfy:

\[
0 \leq \text{beginIndex} \leq \text{endIndex} \leq \text{length}
\]

For example:

```java
String text = "Java";

if (text.length() >= 3) {
    System.out.println(text.substring(1, 3));
}
```

Output:

```text
av
```

Check indices carefully, especially when they come from external input or calculations.

### 3. Large numbers of substrings can consume memory

Suppose an application processes a large document and retains thousands of extracted substrings. Each retained substring may hold its own string representation, increasing memory usage.

For example:

```java
List<String> parts = new ArrayList<>();
String text = "A very large text ...";

for (int i = 0; i < text.length(); i++) {
    parts.add(text.substring(i));
}
```

This example can use substantial memory because many overlapping suffixes are retained.

If a task only requires examining a portion of text, consider processing it directly, retaining only the required results, or using a suitable parsing strategy.

## 3. Does a Substring Share the Original String's Character Array?

This is an important Java interview question.

**For modern Java versions, generally no.** Since Java 7 Update 6, `substring()` has created a separate string representation for the extracted content rather than retaining the original string's backing array.

Older Java implementations, particularly before Java 7 Update 6, could share the backing character array between the original string and a substring. That could cause a small substring to keep a large original array reachable in memory.

For example, in older implementations:

```java
String largeText = loadLargeDocument();
String smallPart = largeText.substring(0, 10);
```

If the substring shared the backing array, keeping `smallPart` alive could also keep the large array alive.

Modern implementations avoid that specific retention issue by copying the substring's contents into its own representation.

**Important:** Java `String` objects are immutable. Modifying a substring does not modify the original string.

## 4. Does Modifying a Substring Change the Original String?

No. `String` is immutable.

```java
String original = "Hello Java";
String part = original.substring(6);

part.concat(" Programming");

System.out.println(original);
System.out.println(part);
```

Output:

```text
Hello Java
Java
```

The result of `concat()` was not assigned, and neither string was modified.

To retain the concatenated result:

```java
part = part.concat(" Programming");
```

This reassigns the variable `part`; it still does not change `original`.

## 5. Performance: `substring()` vs `charAt()` vs `StringBuilder`

| Requirement | Suitable approach |
|---|---|
| Extract a portion of text | `substring()` |
| Read individual characters | `charAt()` |
| Build text through repeated appends | `StringBuilder` |
| Parse large text without creating many intermediate strings | Consider index-based parsing or a suitable parser |
| Retain extracted pieces | Keep only the pieces actually needed |

Do not avoid `substring()` without reason. For normal extraction operations, it is readable and appropriate. Optimize only when allocation, runtime, or memory measurements show a real problem.

## 6. Common Interview Questions

### Q1. Is `substring()` an expensive operation?

It depends on the string length, how often it is called, and whether the resulting objects are retained. Modern Java generally copies the selected content into a new string representation when a distinct substring is needed.

### Q2. Does `substring()` modify the original string?

No. `String` is immutable, so the original string remains unchanged.

### Q3. What exception does `substring()` throw for invalid indices?

It throws `IndexOutOfBoundsException`.

### Q4. Why could substring cause memory problems in older Java versions?

Older implementations could share the backing character array with the original string. A small substring could therefore keep a much larger array reachable even when most of the original text was no longer needed.

### Q5. Should we always replace `substring()` with `StringBuilder`?

No. `substring()` is appropriate when extraction is required. `StringBuilder` is useful for repeated text construction, while `charAt()` is useful when individual characters need to be inspected.

## 7. Quick Revision

- `substring()` extracts text without modifying the original string.
- Invalid indices cause `IndexOutOfBoundsException`.
- Repeated extraction can create many objects and increase memory pressure.
- Modern Java does not generally share the original string's backing array with a substring.
- Older Java versions could retain large arrays through small substrings.
- Use `charAt()` for character inspection and `StringBuilder` for repeated construction.
- Do not assume `substring()` is bad; use it when extraction is the clearest solution.

**One-line interview answer:** Java substrings are not inherently bad, but repeated extraction can cause allocation overhead; older Java versions could also retain large backing arrays through small substrings, an issue avoided by modern implementations.

---
