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

# Java Memory Management, Metaspace, Memory Leaks, and OutOfMemoryError

Java memory management is primarily handled by the Java Virtual Machine (JVM). The JVM allocates memory for objects, manages runtime memory areas, and uses garbage collection to reclaim memory occupied by objects that are no longer reachable.

Understanding heap memory, stack memory, Metaspace, garbage collectors, memory leaks, and `OutOfMemoryError` is important for Java interviews and production troubleshooting.

## 1. What Is Memory Management in Java?

Memory management is the process of allocating, using, and reclaiming memory while a Java application executes.

The JVM manages most application memory automatically. Developers still need to manage object references, resource lifecycles, and memory-intensive operations carefully.

### Major JVM memory areas

| Memory area | Purpose | Typical contents |
|---|---|---|
| Heap | Stores dynamically allocated objects and arrays | Objects, arrays |
| Java stack | Maintains per-thread method execution frames | Local variables, intermediate computations, references |
| Metaspace | Stores class metadata in HotSpot JVMs | Class metadata, method metadata |
| PC register | Tracks the current JVM instruction for each thread | Current instruction address |
| Native method stack | Supports native method execution | Native call state |
| Code cache | Stores JIT-compiled machine code in HotSpot | Compiled methods |

**Important:** A local reference may be stored in a stack frame, but the object it refers to is generally allocated on the heap. JVM optimizations can change the physical allocation details.

### Heap memory and garbage collection

The heap is the primary memory area managed by garbage collection.

```java
public class MemoryExample {

    public static void main(String[] args) {
        Employee employee = new Employee("Danish");

        System.out.println(employee.getName());

        employee = null;
    }
}

class Employee {

    private final String name;

    Employee(String name) {
        this.name = name;
    }

    String getName() {
        return name;
    }
}
```

When `employee` is assigned `null`, the previously referenced `Employee` object may become eligible for garbage collection if no other reachable reference points to it.

Eligibility does not mean the object is reclaimed immediately.

### Generational garbage collection

Many Java garbage collectors organize heap management around the observation that many objects become unreachable quickly.

Conceptually, the traditional generational model contains:

- **Young generation:** Newly allocated objects, often collected frequently.
- **Old generation:** Objects that survive long enough to be promoted or otherwise managed as long-lived objects.

The exact heap organization and collection algorithms depend on the selected garbage collector. Do not assume every collector uses identical young and old generation structures.

## 2. What Is Garbage Collection in Java?

Garbage collection (GC) automatically reclaims memory associated with objects that are no longer reachable from the JVM's live object graph.

Developers generally do not manually free Java objects.

Common garbage collectors include:

| Collector | General characteristics |
|---|---|
| Serial GC | Uses a single GC thread; useful for some small or resource-constrained applications |
| Parallel GC | Uses multiple threads for collection and prioritizes throughput |
| G1 GC | Divides the heap into regions and aims to balance throughput and pause times |
| ZGC | Designed for very low pause times, including with large heaps |
| Shenandoah | A low-pause collector available in supported OpenJDK distributions |

The default garbage collector depends on the JDK distribution, version, platform, and runtime configuration. **Serial GC is not the universal default.** G1 is commonly the default on modern server-class HotSpot JVMs.

### Configure the heap size

```bash
java -Xms512m -Xmx2g -jar application.jar
```

- `-Xms512m`: Sets the initial or minimum heap sizing target, depending on the JVM.
- `-Xmx2g`: Sets the maximum Java heap size.

These options do not set the total memory limit for the entire JVM process. Native memory, thread stacks, Metaspace, code cache, and other allocations also consume memory.

### Select a garbage collector

```bash
java -XX:+UseG1GC -jar application.jar
```

This explicitly requests G1 GC on JVMs that support the option.

Other useful diagnostic commands include:

```bash
java -Xlog:gc* -jar application.jar
jcmd <pid> GC.heap_info
jcmd <pid> GC.class_histogram
```

The `jcmd` commands require a suitable running JVM process and appropriate permissions. Exact diagnostic options vary by JDK version.

### Is `System.gc()` guaranteed to run garbage collection?

No.

```java
System.gc();
```

This requests that the JVM perform garbage collection, but the JVM may ignore the request. It does not guarantee that a particular object will be collected.

### Is `finalize()` used for manual memory management?

No. Java does not provide ordinary manual deallocation of heap objects through `finalize()`.

Finalization is deprecated for removal and is unreliable for resource cleanup. Prefer `try-with-resources`, `AutoCloseable`, and explicit resource management.

## 3. What Is Metaspace in Java?

**Metaspace** is native memory used by HotSpot JVMs to store class metadata. It was introduced in Java 8, replacing the former PermGen area.

Class metadata includes information needed to represent loaded classes, such as class structure and method-related metadata.

Metaspace is distinct from the Java heap, although class metadata and class loading are related to heap objects such as `java.lang.Class` instances.

### PermGen vs Metaspace

| Feature | PermGen | Metaspace |
|---|---|---|
| Used in | HotSpot through Java 7 | HotSpot Java 8 and later |
| Memory location | Part of the former JVM-managed permanent generation | Native memory |
| Sizing | Configured with options such as `-XX:MaxPermSize` | Configured with options such as `-XX:MaxMetaspaceSize` |
| Limitation | Fixed/configured maximum could be reached | Can grow within available native memory and configured limits |
| Error | `OutOfMemoryError: PermGen space` | `OutOfMemoryError: Metaspace` |

Metaspace was **not replaced by Class Data Sharing (CDS)** in Java 11. CDS is a separate feature that can share certain class metadata and related data to reduce startup time and memory usage across JVM processes.

### Benefits of Metaspace

- Uses native memory rather than the former PermGen heap region.
- Can grow dynamically as classes are loaded, subject to memory availability and configured limits.
- Can reclaim metadata when eligible classes and their class loaders are unloaded.
- Removes the need for the old PermGen sizing configuration.

### Configure Metaspace

```bash
java -XX:MaxMetaspaceSize=256m -jar application.jar
```

This limits Metaspace growth to the configured maximum on supported JVMs. An overly restrictive limit can cause class-loading failures.

### What causes `OutOfMemoryError: Metaspace`?

Possible causes include:

- Too many classes being loaded.
- Class loaders being retained and preventing class unloading.
- Runtime code generation or proxy creation producing many classes.
- A configured Metaspace limit that is too small for the application.

Increasing the limit may help when the application genuinely needs more class metadata, but it does not fix a class-loader leak.

## 4. What Is a Memory Leak in Java?

A **memory leak** occurs when an application retains references to objects it no longer needs, preventing the garbage collector from reclaiming them.

Java has automatic memory management, but it can still experience memory leaks.

### Example of a memory leak

```java
import java.util.ArrayList;
import java.util.List;

public class MemoryLeakExample {

    private static final List<byte[]> CACHE = new ArrayList<>();

    public static void main(String[] args) {
        while (true) {
            CACHE.add(new byte[1024 * 1024]);
        }
    }
}
```

The static list continually retains references to allocated byte arrays. Because those arrays remain reachable through `CACHE`, garbage collection cannot reclaim them while the references remain.

This example can exhaust available heap memory; do not run it in a production environment.

### Common causes of memory leaks

1. **Unbounded collections:** Lists, maps, and caches grow without limits or eviction.
2. **Static references:** Static collections retain objects for the lifetime of their class loader.
3. **Listeners and callbacks:** Registrations remain after the associated component should have been released.
4. **Incorrect `ThreadLocal` usage:** Values remain associated with long-lived threads after the logical task finishes.
5. **Class-loader leaks:** Application classes or class loaders remain reachable after redeployment.
6. **Resource leaks:** Files, sockets, and database connections are not closed. These are resource leaks and can cause failures even when Java heap memory is available.

### How to prevent memory leaks

- Remove references when objects are no longer required.
- Use bounded caches and appropriate eviction policies.
- Unregister listeners and callbacks when their lifecycle ends.
- Call `ThreadLocal.remove()` when appropriate, especially when using pooled threads.
- Close resources with `try-with-resources`.
- Review static collections and long-lived object graphs.
- Analyze heap dumps when memory usage grows unexpectedly.

## 5. How to Find a Memory Leak Using a Profiler

A memory profiler helps identify which objects consume memory and why they remain reachable.

### Recommended investigation workflow

1. **Observe memory usage:** Check heap utilization, garbage collection frequency, and post-GC memory trends.
2. **Reproduce the workload:** Exercise the application using a repeatable request pattern.
3. **Capture heap dumps:** Capture dumps when memory usage is high and, when useful, compare them with earlier dumps.
4. **Inspect retained objects:** Identify large collections, unexpectedly numerous objects, and objects retaining large object graphs.
5. **Analyze reference paths:** Inspect paths from GC roots to determine why objects remain reachable.
6. **Fix and verify:** Correct the retaining references, rerun the workload, and compare memory trends.

### Common profiling tools

| Tool | Purpose |
|---|---|
| Java VisualVM | Monitoring, profiling, and heap-dump analysis, depending on available plugins and configuration |
| IntelliJ IDEA profiler | CPU and memory profiling in supported editions and configurations |
| Eclipse Memory Analyzer (MAT) | Heap-dump analysis, dominator trees, and retained-heap investigation |
| JProfiler | Commercial profiling, allocation analysis, and leak investigation |
| `jcmd` | JVM diagnostics, including heap information and heap-dump operations |
| `jstat` | Monitoring selected JVM statistics, including garbage collection |

### Capture a heap dump

```bash
jcmd <pid> GC.heap_dump /tmp/heap.hprof
```

Replace `<pid>` with the Java process ID.

A heap dump can be large and may contain sensitive application data. Store it securely and avoid capturing it unnecessarily in production.

### What should you look for in a heap dump?

- Classes with unexpectedly high instance counts.
- Objects consuming a large amount of retained heap.
- Static collections retaining application objects.
- Repeatedly accumulating request, session, or cache objects.
- Unexpected reference chains from GC roots.

A large object is not automatically a leak. The key question is whether it should still be reachable and whether its retained memory grows unexpectedly over time.

## 6. What Is `OutOfMemoryError` in Java?

`OutOfMemoryError` is an unchecked error thrown when the JVM cannot satisfy a memory allocation requirement.

It is not limited to heap exhaustion.

### Common types of memory-related failures

| Error | Possible cause | Investigation direction |
|---|---|---|
| `OutOfMemoryError: Java heap space` | Heap cannot accommodate required objects | Heap dump, retained objects, heap sizing |
| `OutOfMemoryError: GC overhead limit exceeded` | Excessive time spent in GC with little memory reclaimed, when the limit is enabled and triggered | Heap dump, allocation patterns, leak analysis |
| `OutOfMemoryError: Metaspace` | Metaspace allocation failure or configured limit reached | Class loading and class-loader retention |
| `OutOfMemoryError: unable to create native thread` | OS or native-memory constraints prevent new thread creation | Thread count, stack size, OS limits, native memory |
| `OutOfMemoryError: Direct buffer memory` | Direct-buffer allocation limit or native memory pressure | Direct buffers, buffer lifecycle, JVM settings |

### Common causes

- Memory leaks or unbounded object retention.
- Insufficient heap size for the workload.
- Large allocations or unexpectedly high object-creation rates.
- Excessive thread creation or native memory consumption.
- Excessive class loading or class-loader retention.
- Large direct-buffer allocations.
- Memory limits imposed by the operating system or a container.

### How to troubleshoot `OutOfMemoryError`

**Step 1: Identify the exact error.**

Determine whether it is heap exhaustion, Metaspace exhaustion, native-thread exhaustion, or another memory-related failure.

**Step 2: Inspect memory and GC logs.**

Check heap utilization, allocation rates, GC frequency, and the amount of memory reclaimed after collection.

**Step 3: Capture diagnostic evidence.**

Use a heap dump for Java heap issues, class-loading statistics for Metaspace issues, and native-memory or thread diagnostics for native-memory problems.

**Step 4: Identify the root cause.**

Determine whether memory usage is legitimate for the workload or caused by unbounded retention, excessive allocation, or incorrect resource management.

**Step 5: Apply the appropriate fix.**

Possible actions include correcting a leak, bounding a cache, reducing allocation pressure, adjusting heap sizing, or tuning native-memory and thread usage.

### Enable automatic heap dumps on heap exhaustion

```bash
java \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/tmp \
  -jar application.jar
```

Ensure the target directory has sufficient disk space and appropriate access controls.

### Should you increase `-Xmx`?

Only when the application legitimately needs more heap and the machine or container has enough memory available.

Increasing `-Xmx` may delay a failure caused by a memory leak without fixing the underlying problem. Also account for Metaspace, thread stacks, direct buffers, code cache, and other native allocations when setting container memory limits.

## 7. Important JVM Memory Flags

| JVM option | Purpose |
|---|---|
| `-Xms512m` | Sets the initial heap sizing target |
| `-Xmx2g` | Sets the maximum heap size |
| `-XX:MaxMetaspaceSize=256m` | Sets a maximum Metaspace size |
| `-XX:+UseG1GC` | Selects G1 GC on supported JVMs |
| `-XX:+HeapDumpOnOutOfMemoryError` | Requests a heap dump when an `OutOfMemoryError` occurs |
| `-Xlog:gc*` | Enables detailed GC logging on modern HotSpot JVMs |

Older JVMs used options such as `-XX:NewRatio` to influence generational heap sizing. Applicability and behavior depend on the selected collector and JDK version. Avoid applying tuning flags without evidence from measurements.

## 8. Common Interview Questions

**Q1. Does Java have automatic memory management?**

Yes. The JVM allocates memory and garbage collection reclaims memory occupied by unreachable objects. Developers still need to manage references and external resources correctly.

**Q2. Can Java have memory leaks?**

Yes. Objects that are no longer logically needed can remain reachable through static fields, caches, listeners, or other references.

**Q3. What is the difference between heap and Metaspace?**

The heap stores objects and arrays, whereas Metaspace stores class metadata in HotSpot JVMs and uses native memory.

**Q4. What replaced PermGen in Java 8?**

Metaspace replaced PermGen in HotSpot. Class Data Sharing is a separate feature.

**Q5. What is the difference between a memory leak and `OutOfMemoryError`?**

A memory leak is a retention problem. `OutOfMemoryError` is a failure to satisfy a memory allocation request. A leak can cause the error, but legitimate memory demand or native-memory constraints can also cause it.

**Q6. What is the first step when a production application runs out of memory?**

Identify the exact error and collect appropriate diagnostic evidence. Heap dumps, GC logs, thread diagnostics, and native-memory information help distinguish the possible causes.

**Q7. Does garbage collection close files or database connections?**

No. Garbage collection reclaims memory; it does not provide deterministic cleanup for external resources. Use `try-with-resources` and explicit lifecycle management.

## One-Line Interview Answer

Java memory management is primarily handled by the JVM through heap allocation and garbage collection; Metaspace stores class metadata, memory leaks occur when unneeded objects remain reachable, and `OutOfMemoryError` is diagnosed using GC logs, heap dumps, profilers, and JVM memory diagnostics.


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

