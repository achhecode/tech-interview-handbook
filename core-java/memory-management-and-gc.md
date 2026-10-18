---
title: Memory Management and Garbage Collection
parent: Core Java
nav_order: 9
---

# Memory Management and Garbage Collection
{: .no_toc }

JVM memory areas, garbage collectors and memory leaks.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

# Garbage Collection in Java: Algorithms, JVM, and System.gc()

## 1. What Is Garbage Collection in Java?

**Garbage Collection (GC)** is the JVM's automatic memory-management process that reclaims heap memory occupied by objects that are no longer reachable by the application.

In Java, developers generally do not manually free an object's memory. Instead, the JVM identifies objects that are no longer reachable and reclaims their memory when appropriate.

### Example

```java
public class GarbageCollectionDemo {
    public static void main(String[] args) {
        Person person = new Person();

        person = null;

        // The Person object may now be eligible for GC.
    }
}

class Person {
}
```

Initially, `person` references a `Person` object. After assigning `null`, that reference no longer points to the object.

If no other reachable references to the object exist, it becomes eligible for garbage collection.

**Important:** Eligible for garbage collection does not mean the object will be collected immediately.

## 2. Why Is Garbage Collection Needed?

Garbage collection helps:

- Reclaim memory occupied by unreachable objects.
- Reduce the need for manual memory deallocation.
- Prevent many common memory-management errors, such as dangling pointers.
- Manage heap memory automatically.
- Allow developers to focus on application logic rather than explicitly freeing object memory.

However, garbage collection does not prevent every memory leak. An application can retain references to objects it no longer needs, preventing the garbage collector from reclaiming them.

## 3. How Does Garbage Collection Work Internally?

Modern Java garbage collectors generally use **tracing algorithms** to determine which objects remain reachable.

The process can be simplified into three stages:

<box gap={2}>
  <row align="start" gap={3}>
    <box background="surface-secondary" radius="lg" size="44px" align="center" justify="center">
      <icon name="search" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **1. Identify GC roots**

      The JVM starts from references such as relevant thread stacks, active execution state, and certain static or JVM-managed references.
    </box>
  </row>
  <row align="start" gap={3}>
    <box background="surface-secondary" radius="lg" size="44px" align="center" justify="center">
      <icon name="git-branch" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **2. Trace reachable objects**

      It follows references from GC roots to determine which objects remain reachable.
    </box>
  </row>
  <row align="start" gap={3}>
    <box background="surface-secondary" radius="lg" size="44px" align="center" justify="center">
      <icon name="recycle" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **3. Reclaim memory**

      The collector reclaims memory occupied by unreachable objects. Depending on the collector, it may also copy or compact surviving objects.
    </box>
  </row>
</box>

### Reachable vs Unreachable Objects

```java
Person p1 = new Person();
Person p2 = new Person();

p1 = null;
```

- The object originally referenced by `p1` may become eligible for GC if no other reachable reference points to it.
- The object referenced by `p2` remains reachable through `p2`.

The garbage collector determines reachability, not simply whether a variable is currently being used in source code.

## 4. What Is a GC Root?

A **GC root** is a reference from which the garbage collector begins tracing object reachability.

Common examples include:

- References in active thread stacks and execution state.
- References held by certain static fields.
- References maintained by the JVM, such as those associated with active class loading or runtime infrastructure.
- JNI references, where applicable.

If an object can be reached by following references from a GC root, it is generally considered reachable and cannot be reclaimed as ordinary unreachable garbage.

## 5. What Are the Main Garbage Collection Algorithms?

### 5.1 Mark-and-Sweep

This tracing algorithm has two main phases:

1. **Mark:** Trace from GC roots and mark reachable objects.
2. **Sweep:** Reclaim memory occupied by unmarked objects.

```text
Before GC:
[A] [B] [C] [D] [E]

Reachable: A, C, E

After marking:
[A✓] [B✗] [C✓] [D✗] [E✓]

After sweeping:
[A] [free] [C] [free] [E]
```

**Advantages**
- Relatively straightforward.
- Reclaims unreachable objects without copying every live object.

**Disadvantages**
- Can leave fragmented free memory.
- May require additional work to allocate large contiguous objects.

### 5.2 Mark-and-Compact

This algorithm marks reachable objects and moves them together to reduce fragmentation.

**Advantages**
- Reduces fragmentation.
- Can create larger contiguous free regions.

**Disadvantages**
- Moving objects can be costly.
- References to moved objects must be handled correctly.

### 5.3 Copying Collection

This algorithm copies surviving objects from one memory region to another and then reclaims the source region.

**Advantages**
- Efficient when relatively few objects survive.
- Reclaims the source region in bulk.
- Helps avoid fragmentation in the destination region.

**Disadvantages**
- Requires space for the destination region.
- Copying live objects consumes processing time.

### 5.4 Generational Collection

Many Java collectors use generational techniques based on the observation that many objects become unreachable shortly after creation.

The traditional generational model separates heap memory into:

- **Young Generation:** Where many newly allocated objects begin.
- **Old Generation:** Where objects that survive long enough may be placed.

The exact layout and terminology depend on the collector. For example, G1 divides the heap into regions and does not use the same fixed contiguous young/old layout as older collectors.

## 6. What Is the Difference Between an Algorithm and a Garbage Collector?

An algorithm describes how memory is identified or reclaimed. A garbage collector is an implementation that combines algorithms and runtime techniques.

| Garbage collector | General characteristics |
|---|---|
| Serial GC | Uses a single GC thread for collection work; suitable for some smaller workloads |
| Parallel GC | Uses multiple threads for collection work; aims for throughput |
| G1 GC | Region-based collector designed to balance throughput and pause-time goals |
| ZGC | Designed for very low pause times, including on large heaps |
| Shenandoah GC | Uses concurrent work to reduce pause times |

Availability and defaults depend on the JDK distribution, version, platform, and configuration.

### Parallel GC

Parallel GC uses multiple threads for garbage collection. In commonly used generational configurations, collection work can involve copying surviving objects in the young generation and compacting the old generation.

However, **parallel GC is not itself a single memory-reclamation algorithm**. It describes parallel execution of collection work.

Similarly, mark-and-sweep is an algorithmic approach, while G1, ZGC, and Parallel GC are names of specific garbage collectors with their own implementations.

## 7. What Is System.gc() in Java?

`System.gc()` requests that the JVM make an effort to reclaim memory by running garbage collection.

```java
public class Main {
    public static void main(String[] args) {
        Object obj = new Object();

        obj = null;

        System.gc();

        System.out.println("GC requested");
    }
}
```

**Possible output:**

```text
GC requested
```

The output does not prove that garbage collection ran or that any particular object was collected.

### Important Points

- `System.gc()` is a request, not a guarantee.
- The JVM may ignore the request or handle it according to its configuration.
- It does not guarantee immediate collection of a particular object.
- It does not guarantee that heap usage will immediately decrease.
- Calling it repeatedly can harm performance by causing unnecessary collection work.

In most production applications, let the JVM manage garbage collection automatically. Use explicit GC requests only when there is a well-understood reason, such as a controlled experiment or profiling scenario.

## 8. Can We Force Garbage Collection in Java?

No. Java does not provide a general guarantee that an application can force the JVM to collect a particular object at a specific time.

```java
Person person = new Person();
person = null;

System.gc();
```

After the reference is cleared, the object may be eligible for collection, but the timing is controlled by the JVM.

Also, clearing one reference is insufficient if another reachable object still references the same object.

## 9. What Is finalize() in Java?

Historically, a class could override `finalize()` to run cleanup-related code before an object was reclaimed.

```java
@Override
protected void finalize() throws Throwable {
    // Legacy finalization mechanism
}
```

However, finalization is deprecated for removal and should not be used for resource management. It is unpredictable, can delay reclamation, and may introduce security and reliability problems.

Use `try-with-resources` and `AutoCloseable` for resources such as files and streams.

```java
try (var reader = new java.io.BufferedReader(
        new java.io.FileReader("input.txt"))) {
    System.out.println(reader.readLine());
}
```

The resource is closed when execution leaves the `try` block, without waiting for garbage collection.

## 10. What Is a Memory Leak in Java?

A memory leak occurs when an application retains references to objects that it no longer needs, preventing those objects from becoming unreachable.

Example:

```java
import java.util.ArrayList;
import java.util.List;

public class MemoryLeakDemo {
    private static final List<byte[]> CACHE = new ArrayList<>();

    public static void main(String[] args) {
        while (true) {
            CACHE.add(new byte[1024 * 1024]);
        }
    }
}
```

This example continuously adds arrays to a static list without removing them. The list retains references to those arrays, so the garbage collector cannot reclaim them while those references remain reachable.

### How to Prevent Memory Leaks

- Remove obsolete entries from collections and caches.
- Unregister listeners when they are no longer needed.
- Close resources such as files and database connections.
- Avoid unnecessary long-lived references.
- Use heap dumps and profiling tools to investigate memory growth.

Garbage collection reclaims unreachable objects; it cannot determine that a reachable object is logically unnecessary.

## 11. Does Garbage Collection Always Run on the Heap?

Java garbage collection primarily manages heap objects. Other resources, such as file handles, sockets, and database connections, require appropriate resource-management practices.

The JVM also manages memory outside the ordinary Java heap, including native memory used by some runtime components. Not all such memory is reclaimed by the Java heap garbage collector in the same way.

This is why an application can run out of native memory or file descriptors even when its Java heap appears to have available space.

## 12. Common Interview Questions

### Q1. When does an object become eligible for garbage collection?

When it is no longer reachable through the JVM's reachability model, subject to details such as special references and runtime behavior.

### Q2. Can we guarantee when garbage collection will happen?

No. The JVM controls the timing of garbage collection.

### Q3. Is garbage collection the same as memory-leak prevention?

No. GC reclaims unreachable objects, but it cannot reclaim objects that remain reachable through unnecessary references.

### Q4. Does `System.gc()` guarantee collection?

No. It only requests that the JVM make an effort to reclaim memory.

### Q5. What is the difference between mark-and-sweep and mark-and-compact?

Mark-and-sweep reclaims unmarked objects but can leave fragmented memory. Mark-and-compact also moves surviving objects together to reduce fragmentation.

### Q6. Is Java garbage collection based on reference counting?

Modern mainstream Java garbage collectors primarily use tracing and reachability rather than simple reference counting. This allows them to reclaim unreachable cycles of objects.

### Q7. Can an object be collected while a method is still executing?

Potentially, yes, if the object is no longer strongly reachable according to the JVM's liveness analysis, even if a local variable still appears in source code. The JVM is not required to keep every local reference alive until the end of its lexical scope.

## 13. One-Line Interview Answer

Garbage collection in Java is the JVM-managed process of reclaiming memory occupied by unreachable objects, using tracing and reclamation techniques such as marking, sweeping, copying, and compaction to manage memory automatically.

---

# Where Are Static Variables and Methods Stored in Java Memory?

## 1. Where Are Static Variables and Static Methods Stored?

Static variables and static methods belong to a class rather than to individual objects. However, **it is inaccurate to say that both are stored in Metaspace**.

In HotSpot JVM:

- **Metaspace** stores class metadata, such as class structure, method metadata, and runtime constant-pool information.
- **Static variables** are associated with the class's runtime representation. Their exact storage location is a JVM implementation detail; in modern HotSpot JVMs, static field values are associated with the class mirror, a `java.lang.Class` object on the heap.
- **Static method bytecode and metadata** are associated with class metadata, while compiled machine code may be stored in the code cache.
- **Instance variables** are part of individual objects, which are generally allocated on the heap.

The exact physical layout depends on the JVM implementation and version.

## 2. Java Memory Areas

| Memory area | What it stores |
|---|---|
| Heap | Objects, arrays, and class mirror objects |
| Java stack | Per-thread stack frames, local variables, and intermediate execution state |
| Metaspace | Class metadata in HotSpot JVM; uses native memory |
| Code cache | JIT-compiled machine code and related runtime code |
| Program counter (PC) register | Per-thread information about the current JVM instruction being executed |

## 3. Example: Static vs Instance Variables

```java
class Employee {
    static String company = "ABC Ltd";
    String name;

    Employee(String name) {
        this.name = name;
    }

    static void showCompany() {
        System.out.println(company);
    }
}

public class Main {
    public static void main(String[] args) {
        Employee e1 = new Employee("Ali");
        Employee e2 = new Employee("Sara");

        Employee.showCompany();

        System.out.println(e1.name);
        System.out.println(e2.name);
    }
}
```

**Output:**

```text
ABC Ltd
Ali
Sara
```

### How memory is associated with these fields

- `company` is one static field associated with the loaded `Employee` class, not a separate field in every employee object.
- `name` is an instance field, so each `Employee` object has its own value.
- `showCompany()` is a static method that can be invoked using the class name without creating an `Employee` object.

Conceptually:

```text
Class metadata (Metaspace)
    Employee class structure
    Method metadata
           |
           v
Heap
    Employee.class mirror
        |
        +-- static field: company = "ABC Ltd"

    Employee object 1
        +-- name = "Ali"

    Employee object 2
        +-- name = "Sara"
```

This diagram illustrates the association between the class, static state, and instances. It is not an exact physical memory layout.

## 4. What Happened to PermGen in Java 8?

Before Java 8, HotSpot JVM used **PermGen (Permanent Generation)** for class metadata and certain related data.

In Java 8, PermGen was removed and replaced by **Metaspace**, which uses native memory for class metadata.

| Feature | PermGen | Metaspace |
|---|---|---|
| Used in HotSpot | Before Java 8 | Java 8 onward |
| Memory source | JVM-managed heap region | Native memory |
| Class metadata | Stored here | Stored here |
| Capacity | Fixed/configurable limit | Can grow within available native memory and configured limits |

Metaspace can still run out of memory. For example, excessive class loading or class-loader leaks can cause `OutOfMemoryError: Metaspace`.

**Important:** The PermGen-to-Metaspace change does not mean that every static field and static method was simply moved into Metaspace.

## 5. Common Interview Questions

### Q1. Are static variables stored in Metaspace?

Not exactly. In modern HotSpot JVMs, static field values are associated with the class mirror object on the heap. Metaspace stores class metadata. JVM implementations can differ.

### Q2. Are static methods stored in Metaspace?

Method metadata and bytecode are associated with class metadata. JIT-compiled machine code may reside in the code cache. Therefore, saying that the entire static method is stored in Metaspace is an oversimplification.

### Q3. Where are instance variables stored?

Instance fields are part of their containing objects, which are generally allocated on the heap.

### Q4. Are local variables always stored on the stack?

Not necessarily. Local variables and execution state are logically associated with stack frames, but the JIT compiler can optimize storage, including keeping values in registers or eliminating allocations.

### Q5. Does every object get a separate copy of a static variable?

No. A static field is associated with the class, so all instances of that class share the same static field value for that particular loaded class definition.

## 6. One-Line Interview Answer

In the HotSpot JVM, Metaspace stores class metadata, while static field values are associated with the class mirror object on the heap; method metadata and bytecode are associated with class metadata, and JIT-compiled code may reside in the code cache.


---

