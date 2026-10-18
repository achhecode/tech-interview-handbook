---
title: Collections Framework
parent: Core Java
nav_order: 5
---

# Collections Framework
{: .no_toc }

List, Set, Map internals and choosing the right collection.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

# Java Collections Framework Hierarchy

## 1. What Is the Java Collections Framework?

The **Java Collections Framework (JCF)** is a set of interfaces, implementation classes, and utility algorithms used to store, retrieve, manipulate, and process groups of objects.

It provides reusable data structures such as lists, sets, queues, and maps, so developers do not need to implement these structures from scratch.

The main components are:

- **Interfaces:** Define operations and contracts, such as `List`, `Set`, `Queue`, and `Map`.
- **Implementation classes:** Provide concrete data structures, such as `ArrayList`, `HashSet`, and `HashMap`.
- **Utility algorithms:** The `Collections` class provides operations such as sorting, reversing, searching, and creating unmodifiable views.

## 2. Java Collections Framework Hierarchy

The most important distinction is that **`Map` does not extend `Collection`**. It belongs to the Collections Framework but follows a separate interface hierarchy.

<CodeBlock language="mermaid" editable>
flowchart TD
    Iterable --> Collection
    Collection --> List
    Collection --> Set
    Collection --> Queue
    Queue --> Deque

    List --> ArrayList
    List --> LinkedList
    List --> Vector
    Vector --> Stack

    Set --> HashSet
    Set --> SortedSet
    SortedSet --> NavigableSet
    NavigableSet --> TreeSet
    HashSet --> LinkedHashSet

    Queue --> PriorityQueue
    Deque --> ArrayDeque
    Deque --> LinkedList

    Map["Map (separate hierarchy)"] --> HashMap
    Map --> LinkedHashMap
    Map --> SortedMap
    SortedMap --> NavigableMap
    NavigableMap --> TreeMap
    Map --> Hashtable
    Map --> ConcurrentMap
    ConcurrentMap --> ConcurrentHashMap
</CodeBlock>

This diagram highlights the commonly used interfaces and classes. It is a simplified view rather than the complete Java API hierarchy.

## 3. The `Iterable` and `Collection` Interfaces

### `Iterable`

`Iterable` is the parent interface of `Collection`. It allows an object to be traversed using the enhanced `for` loop.

```java
List<String> names = List.of("Ali", "Sara", "John");

for (String name : names) {
    System.out.println(name);
}
```

### `Collection`

`Collection` extends `Iterable` and provides common operations for groups of elements.

Important methods include:

| Method | Purpose |
|---|---|
| `add(e)` | Adds an element when supported |
| `remove(o)` | Removes a matching element |
| `contains(o)` | Checks whether an element exists |
| `size()` | Returns the number of elements |
| `isEmpty()` | Checks whether the collection is empty |
| `clear()` | Removes all elements when supported |
| `iterator()` | Returns an iterator |

`Collection` has three principal subinterfaces: `List`, `Set`, and `Queue`. `Deque` extends `Queue`.

## 4. List Interface

A `List` represents an ordered sequence of elements.

Characteristics:

- Maintains element order.
- Allows duplicates.
- Supports index-based access.
- Usually allows `null`, depending on the implementation.

Common implementations:

| Class | Characteristics |
|---|---|
| `ArrayList` | Resizable array; efficient indexed access |
| `LinkedList` | Doubly linked list; implements both `List` and `Deque` |
| `Vector` | Legacy synchronized resizable array |
| `Stack` | Legacy stack class extending `Vector` |

Example:

```java
import java.util.ArrayList;
import java.util.List;

public class ListExample {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();

        names.add("Ali");
        names.add("Sara");
        names.add("Ali");

        System.out.println(names);
        System.out.println(names.get(1));
    }
}
```

Output:

```text
[Ali, Sara, Ali]
Sara
```

**Interview tip:** Prefer `ArrayList` for general-purpose list usage. `LinkedList` is not automatically faster for insertions; finding the insertion position can require traversal.

## 5. Set Interface

A `Set` represents a collection that does not contain duplicate elements according to its equality or ordering rules.

Common implementations:

| Class | Characteristics |
|---|---|
| `HashSet` | Hash-based; no guaranteed iteration order |
| `LinkedHashSet` | Maintains insertion order |
| `TreeSet` | Maintains sorted order |

Example:

```java
import java.util.HashSet;
import java.util.Set;

public class SetExample {
    public static void main(String[] args) {
        Set<Integer> numbers = new HashSet<>();

        numbers.add(10);
        numbers.add(20);
        numbers.add(10);

        System.out.println(numbers.size());
        System.out.println(numbers.contains(20));
    }
}
```

Output:

```text
2
true
```

The duplicate `10` is not added as a second distinct element. The printed iteration order of a `HashSet` is not guaranteed.

### Sorted sets

`SortedSet` extends `Set` and defines sorted-order behavior. `NavigableSet` extends `SortedSet` and adds navigation methods such as:

- `lower(e)`
- `floor(e)`
- `ceiling(e)`
- `higher(e)`

`TreeSet` implements `NavigableSet`.

## 6. Queue and Deque Interfaces

### Queue

A `Queue` holds elements for processing according to a queue discipline. Many queues follow FIFO (first in, first out), but not all do. For example, `PriorityQueue` orders elements by priority rather than insertion order.

Common methods:

| Method | Behavior |
|---|---|
| `offer(e)` | Adds an element; returns `false` if it cannot be added |
| `poll()` | Retrieves and removes the head, or returns `null` if empty |
| `peek()` | Retrieves the head without removing it, or returns `null` if empty |
| `remove()` | Retrieves and removes the head; throws if empty |
| `element()` | Retrieves the head without removing it; throws if empty |

Common implementations include `PriorityQueue` and `LinkedList`.

### Deque

`Deque` means **double-ended queue**. It supports insertion and removal at both ends.

Common implementations:

- `ArrayDeque`
- `LinkedList`

Example:

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequeExample {
    public static void main(String[] args) {
        Deque<String> deque = new ArrayDeque<>();

        deque.addFirst("A");
        deque.addLast("B");
        deque.addFirst("C");

        System.out.println(deque);
        System.out.println(deque.removeFirst());
        System.out.println(deque.removeLast());
    }
}
```

Output:

```text
[C, A, B]
C
B
```

`ArrayDeque` is often a good choice for stack or queue operations. Unlike `Stack`, it does not require the legacy `Vector` inheritance hierarchy. It does not permit `null` elements.

## 7. Map Interface — A Separate Hierarchy

A `Map` stores key-value pairs. Each key is unique according to the map's key-equality rules, while multiple keys can map to the same value.

**`Map` does not extend `Collection`.**

Common implementations:

| Class | Characteristics |
|---|---|
| `HashMap` | Hash-based; no guaranteed iteration order |
| `LinkedHashMap` | Predictable encounter order, usually insertion order by default |
| `TreeMap` | Sorts keys according to natural ordering or a comparator |
| `Hashtable` | Legacy synchronized map; rejects null keys and values |
| `ConcurrentHashMap` | Designed for concurrent access; rejects null keys and values |

Example:

```java
import java.util.HashMap;
import java.util.Map;

public class MapExample {
    public static void main(String[] args) {
        Map<Integer, String> students = new HashMap<>();

        students.put(101, "Ali");
        students.put(102, "Sara");
        students.put(101, "Ahmed");

        System.out.println(students);
        System.out.println(students.get(101));
    }
}
```

Output:

```text
{101=Ahmed, 102=Sara}
Ahmed
```

Adding another value for key `101` replaces the existing value associated with that key.

### Important Map subinterfaces

- `SortedMap` defines sorted key ordering.
- `NavigableMap` extends `SortedMap` with navigation operations.
- `ConcurrentMap` defines additional operations useful for concurrent map access.

`TreeMap` implements `NavigableMap`, and `ConcurrentHashMap` implements `ConcurrentMap`.

## 8. How to Choose the Right Collection

| Requirement | Common choice |
|---|---|
| General-purpose resizable list | `ArrayList` |
| Unique elements, order unimportant | `HashSet` |
| Unique elements in insertion order | `LinkedHashSet` |
| Unique elements in sorted order | `TreeSet` |
| FIFO queue | `ArrayDeque` |
| Priority-based processing | `PriorityQueue` |
| Stack operations | `ArrayDeque` |
| Key-value lookup | `HashMap` |
| Key-value lookup preserving encounter order | `LinkedHashMap` |
| Key-value lookup sorted by key | `TreeMap` |
| Concurrent key-value operations | `ConcurrentHashMap` |

The best implementation depends on access patterns, ordering requirements, concurrency, null handling, and performance needs.

## 9. `Collection` vs `Collections` vs `Map`

| Term | Meaning |
|---|---|
| `Collection` | Root interface for collection types such as lists and sets |
| `Collections` | Utility class containing static algorithms and wrappers |
| `Map` | Separate interface for key-value associations |
| `Arrays` | Utility class for working with arrays |

Example:

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class CollectionsUtilityExample {
    public static void main(String[] args) {
        List<Integer> numbers =
                new ArrayList<>(List.of(30, 10, 20));

        Collections.sort(numbers);

        System.out.println(numbers);
    }
}
```

Output:

```text
[10, 20, 30]
```

## 10. Common Interview Questions

**Q1. Is `Map` a child of `Collection`?**

No. `Map` belongs to the Java Collections Framework but does not extend `Collection`.

**Q2. What is the difference between `List` and `Set`?**

A `List` preserves sequence order and permits duplicates. A `Set` enforces uniqueness, with iteration order depending on the implementation.

**Q3. What is the difference between `Queue` and `Deque`?**

A `Queue` provides queue-oriented operations. A `Deque` supports operations at both the front and back and can be used as either a queue or a stack.

**Q4. Which interface does `HashMap` implement?**

`HashMap` implements `Map`, not `Collection`.

**Q5. Is `LinkedList` a `Queue`?**

Yes. `LinkedList` implements `List` and `Deque`; because `Deque` extends `Queue`, it is also a `Queue`.

**Q6. What is the difference between `HashSet` and `TreeSet`?**

`HashSet` does not guarantee iteration order. `TreeSet` maintains sorted order according to natural ordering or a supplied comparator.

**Q7. Why is the Collections Framework useful?**

It provides standard interfaces, reusable implementations, common algorithms, and interoperability between different data structures.

## 11. Quick Revision

- `Iterable` is the parent of `Collection`.
- The principal `Collection` subinterfaces are `List`, `Set`, and `Queue`.
- `Deque` extends `Queue`.
- `Map` is part of the framework but is separate from the `Collection` hierarchy.
- `ArrayList`, `HashSet`, and `ArrayDeque` are common collection implementations.
- `HashMap`, `LinkedHashMap`, and `TreeMap` are common map implementations.
- `Collections` is a utility class, not an interface.

**One-line interview answer:** The Java Collections Framework organizes data structures through interfaces such as `List`, `Set`, and `Queue`, with `Deque` extending `Queue`; `Map` is a separate hierarchy for key-value pairs, and classes such as `ArrayList`, `HashSet`, and `HashMap` provide concrete implementations.

---


# How Does `HashMap` Work Internally in Java?

`HashMap` is a class in the Java Collections Framework that stores data as **key-value pairs**. It uses a hash table to provide efficient insertion, retrieval, and deletion of entries.

Internally, Java's `HashMap` uses an array of buckets. Each bucket can contain a linked list of nodes or, under certain conditions, a balanced red-black tree to handle collisions.

## 1. Internal Structure of HashMap

A `HashMap` primarily uses these components:

- **Bucket array:** An array that stores references to the first node in each bucket.
- **Node:** Stores the key, value, hash, and a reference to the next node when entries share a bucket.
- **Hash function:** Transforms the key's hash code to help determine its bucket index.
- **Collision handling:** Uses linked lists and can convert a sufficiently large collision chain into a red-black tree.
- **Load factor and threshold:** Determine when the backing array should grow.

A simplified representation of a node in modern Java is:

```java
static class Node<K, V> {
    final int hash;
    final K key;
    V value;
    Node<K, V> next;
}
```

This is a conceptual representation of the structure used by OpenJDK's `HashMap`.

## 2. How Does HashMap Calculate the Bucket Index?

When you insert or retrieve an entry, `HashMap` calculates a hash from the key's `hashCode()` value and uses it to determine the bucket.

In modern OpenJDK implementations, the hash is spread using a transformation equivalent to:

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

The bucket index is calculated using:

```java
int index = (capacity - 1) & hash;
```

Here, `capacity` is the current length of the bucket array, normally a power of two.

**Why spread the hash?** The transformation mixes higher-order bits into lower-order bits, helping the bucket calculation use more of the hash code.

The exact implementation details can vary by Java version, but this describes the approach used by modern OpenJDK implementations.

## 3. How Does `put(key, value)` Work?

When you execute:

```java
map.put("Apple", 100);
```

Java generally follows these steps:

<box border radius="lg" padding={3} gap={2}>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **1. Calculate the key's hash**

    <text size="sm" textAlign="center">Call `hashCode()` and spread the hash bits.</text>
  </box>
  <box align="center"><icon name="arrow-down" size="lg" /></box>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **2. Calculate the bucket index**

    <text size="sm" textAlign="center">Use the hash and array capacity to find the bucket.</text>
  </box>
  <box align="center"><icon name="arrow-down" size="lg" /></box>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **3. Check the bucket**

    <text size="sm" textAlign="center">If empty, insert a new node. Otherwise, compare existing entries.</text>
  </box>
  <box align="center"><icon name="arrow-down" size="lg" /></box>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **4. Resolve the entry**

    <text size="sm" textAlign="center">Update an equal key's value or add a new node. A long collision chain may become a tree.</text>
  </box>
  <box align="center"><icon name="arrow-down" size="lg" /></box>
  <box background="surface" border radius="md" padding={3} align="center">
    **5. Check the resize threshold**

    <text size="sm" textAlign="center">Resize the table when required.</text>
  </box>
</box>

### Example

```java
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();

        map.put("Apple", 100);
        map.put("Banana", 200);
        map.put("Orange", 300);

        System.out.println(map);
    }
}
```

The map stores three key-value pairs. The printed order is not guaranteed.

### What Happens When the Same Key Is Inserted Again?

```java
Map<String, Integer> map = new HashMap<>();

map.put("Apple", 100);
map.put("Apple", 500);

System.out.println(map.get("Apple"));
System.out.println(map.size());
```

**Output:**

```text
500
1
```

When an equal key already exists, `HashMap` replaces its value rather than adding another entry.

## 4. How Does `get(key)` Work?

Suppose we execute:

```java
Integer price = map.get("Apple");
```

The process is:

1. Calculate the hash for `"Apple"`.
2. Determine the corresponding bucket index.
3. Examine the first node in that bucket.
4. Compare the stored hash and key, using reference equality or `equals()` as appropriate.
5. If necessary, traverse the linked list or search the tree.
6. Return the matching value, or `null` if no matching key exists.

### Example

```java
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        Map<Integer, String> employees = new HashMap<>();

        employees.put(101, "Danish");
        employees.put(102, "Rahul");

        System.out.println(employees.get(101));
        System.out.println(employees.get(999));
    }
}
```

**Output:**

```text
Danish
null
```

The key `101` exists, so its value is returned. The key `999` does not exist, so `get()` returns `null`.

## 5. What Is a Hash Collision?

A collision occurs when two different keys map to the same bucket index. The keys may have different hash codes but still map to the same bucket; they can also have identical hash codes.

For example, consider a simplified bucket array:

<box border radius="lg" padding={3} gap={2}>
  <row align="center" gap={3}>
    <box background="surface-secondary" radius="md" padding={3} width="68px" align="center">
      **0**
    </box>
    <box flex="1" background="surface" border radius="md" padding={3}>
      <text color="secondary" size="sm">Empty bucket</text>
    </box>
  </row>
  <row align="center" gap={3}>
    <box background="surface-secondary" radius="md" padding={3} width="68px" align="center">
      **1**
    </box>
    <box flex="1" gap={1}>
      <box border radius="md" padding={2} align="center">`("Apple", 100)`</box>
      <box align="center"><icon name="arrow-down" /></box>
      <box border radius="md" padding={2} align="center">`("Banana", 200)`</box>
      <text color="secondary" size="xs" textAlign="center">Two nodes in the same bucket, for illustration</text>
    </box>
  </row>
  <row align="center" gap={3}>
    <box background="surface-secondary" radius="md" padding={3} width="68px" align="center">
      **2**
    </box>
    <box flex="1" background="surface" border radius="md" padding={3}>
      <text color="secondary" size="sm">Another bucket</text>
    </box>
  </row>
</box>

In modern Java, collisions are handled primarily through:

- **Linked lists:** Multiple nodes are linked together in one bucket.
- **Red-black trees:** A sufficiently large collision bin can be converted into a tree when capacity and other implementation conditions permit.

`HashMap` does not normally use open addressing; it uses separate chaining with linked lists and tree bins.

## 6. What Is Treeification in HashMap?

When too many entries accumulate in a single bucket, Java can convert its linked list into a red-black tree to improve search performance.

In modern OpenJDK implementations, the important constants include:

| Constant | Typical value | Meaning |
|---|---:|---|
| `TREEIFY_THRESHOLD` | 8 | Node-count threshold considered for treeification |
| `UNTREEIFY_THRESHOLD` | 6 | Threshold used in certain tree-splitting operations when resizing |
| `MIN_TREEIFY_CAPACITY` | 64 | Minimum table capacity for treeification instead of resizing |

These are implementation details, not guarantees for every Java implementation. The precise conversion behavior also depends on the insertion or resizing operation.

A linked-list bucket can require \(O(n)\) comparisons in the worst case. A tree bin can generally search in \(O(\log n)\) time, assuming suitable hash and comparison behavior.

## 7. What Is the Load Factor in HashMap?

The load factor controls how full the table can become before a resize is triggered.

The default load factor is:

```java
0.75f
```

The threshold is generally calculated as:

\[
\text{threshold} = \text{capacity} \times \text{load factor}
\]

For example, if the table capacity is 16:

\[
16 \times 0.75 = 12
\]

The resize threshold is 12 entries. When the number of stored entries exceeds the threshold, the table generally grows.

### Why Is Resizing Necessary?

As a hash table becomes more crowded, collisions can increase. Resizing increases the bucket array capacity and redistributes entries to help maintain efficient operations.

A resize is relatively expensive because entries must be redistributed, but it does not happen on every insertion.

### Example

```java
Map<Integer, String> map = new HashMap<>(16, 0.75f);
```

This creates a map with an initial capacity request of 16 and a load factor of 0.75. The internal table is allocated lazily in standard OpenJDK implementations, so construction does not necessarily allocate the bucket array immediately.

## 8. What Happens When the Key Is `null`?

`HashMap` allows one `null` key and multiple `null` values.

```java
Map<String, Integer> map = new HashMap<>();

map.put(null, 10);
map.put("Apple", null);

System.out.println(map.get(null));
System.out.println(map.containsKey("Apple"));
System.out.println(map.get("Apple"));
```

**Output:**

```text
10
true
null
```

Notice that `get("Apple")` returns `null`, but this does not mean the key is absent. Use `containsKey()` to distinguish an absent key from a key mapped to `null`.

## 9. Why Are `equals()` and `hashCode()` Important for HashMap?

`HashMap` depends on both methods to locate keys correctly.

The contract is:

- If two keys are equal according to `equals()`, they must have the same hash code.
- If two keys have the same hash code, they are not necessarily equal.
- When keys share a bucket, `equals()` helps identify the correct key.

### Example with a Custom Key

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

class Employee {
    private final int id;

    Employee(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        Employee employee = (Employee) obj;
        return id == employee.id;
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}

public class Main {
    public static void main(String[] args) {
        Map<Employee, String> employees = new HashMap<>();

        employees.put(new Employee(101), "Danish");

        System.out.println(employees.get(new Employee(101)));
    }
}
```

**Output:**

```text
Danish
```

The two `Employee` instances are distinct objects, but they are logically equal because they have the same ID. Implementing both methods consistently allows the second instance to retrieve the value stored using the first.

**Best practice:** Avoid changing fields that participate in `equals()` or `hashCode()` while an object is being used as a map key.

## 10. What Is the Time Complexity of HashMap?

| Operation | Typical/Average Case | Worst Case |
|---|---|---|
| `put()` | \(O(1)\) amortized | Can be \(O(n)\) in certain cases, such as resizing or pathological collisions |
| `get()` | \(O(1)\) | \(O(n)\) for a heavily collided linked-list bin |
| `remove()` | \(O(1)\) | Can be \(O(n)\) for a heavily collided linked-list bin |
| Search in a tree bin | — | Generally \(O(\log n)\) under suitable conditions |

These are expected complexity descriptions, not strict guarantees for every key type or workload. Hash distribution, equality checks, resizing, and tree-bin behavior all affect actual performance.

## 11. Common Interview Questions

**Q1. Is HashMap thread-safe?**

No. `HashMap` is not designed for concurrent modification by multiple threads without appropriate synchronization. For concurrent access, consider `ConcurrentHashMap` when its behavior meets your requirements.

**Q2. Does HashMap maintain insertion order?**

No. `HashMap` does not guarantee insertion order. Use `LinkedHashMap` when predictable encounter order is needed.

**Q3. Can HashMap contain duplicate keys?**

It cannot contain multiple distinct entries for keys that are equal according to its key-equality rules. Inserting an equal key replaces the existing value.

**Q4. What happens when two keys have the same hash code?**

They may land in the same bucket. `HashMap` uses equality checks to distinguish unequal keys and can store both.

**Q5. Why is the default load factor 0.75?**

It is a practical balance between memory usage and collision frequency. A lower load factor generally uses more memory, while a higher one can increase collisions.

**Q6. Why does HashMap capacity typically use powers of two?**

Modern OpenJDK implementations use power-of-two table capacities so bucket indices can be calculated efficiently using a bitwise operation.

## Quick Revision

| Concept | Key Point |
|---|---|
| Internal structure | Bucket array with linked-list or tree bins |
| Hash calculation | Uses the key's `hashCode()` and spreads bits |
| Bucket index | `(capacity - 1) & hash` in modern OpenJDK |
| Collision resolution | Linked lists and red-black trees |
| Default load factor | `0.75` |
| Resizing | Occurs when the threshold is exceeded |
| Duplicate key | Replaces the existing value |
| `null` key | One allowed |
| Average lookup | Typically \(O(1)\) |
| Correct custom keys | Override `equals()` and `hashCode()` consistently |

**Interview takeaway:** `HashMap` combines hashing, bucket indexing, equality checks, collision handling, and resizing to provide efficient key-value operations. A solid understanding of `put()`, `get()`, collisions, treeification, and the `equals()`/`hashCode()` contract is essential for Java interviews.

---


# How HashSet Works Internally in Java

## 1. What is HashSet?

`HashSet` is a class in the Java Collections Framework that implements the `Set` interface.

It stores **unique elements** and does not allow duplicates according to the `equals()` method.

Key characteristics:

- Stores unique elements.
- Allows one `null` element.
- Does not guarantee insertion order.
- Uses a `HashMap` internally.
- Is not thread-safe.
- Provides average \(O(1)\) time complexity for adding, removing, and checking elements, assuming a good hash distribution.

## 2. How does HashSet work internally?

Internally, `HashSet` uses a `HashMap` to store its elements.

- **HashSet element:** Stored as a key in the underlying `HashMap`.
- **Dummy object:** Stored as the value for every key.
- **hashCode():** Helps determine the bucket where an element should be stored.
- **equals():** Determines whether two elements are equal.

In OpenJDK, the dummy value is a shared constant named `PRESENT`.

Conceptually, the implementation looks like this:

```java
public class HashSet<E> {
    private transient HashMap<E, Object> map;

    private static final Object PRESENT = new Object();

    public boolean add(E element) {
        return map.put(element, PRESENT) == null;
    }
}
```

This is a simplified illustration of the implementation, not the complete source code.

### Internal structure

<pre>
HashSet
   |
   v
HashMap
   |
   +-- Key: "Java"     Value: PRESENT
   +-- Key: "Python"   Value: PRESENT
   +-- Key: "Spring"   Value: PRESENT
</pre>

The elements are stored as keys, so the `HashMap` key-uniqueness rules ensure that duplicate elements are not stored.

## 3. What happens when we add an element?

Consider this code:

```java
import java.util.HashSet;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        Set<String> languages = new HashSet<>();

        System.out.println(languages.add("Java"));
        System.out.println(languages.add("Python"));
        System.out.println(languages.add("Java"));

        System.out.println(languages);
    }
}
```

Output:

```text
true
true
false
[Java, Python]
```

The output order of the elements is not guaranteed.

### Step-by-step execution

**Step 1: Add `"Java"`**

```java
languages.add("Java");
```

- `HashSet` delegates the operation to `HashMap.put("Java", PRESENT)`.
- The `HashMap` calculates the hash and determines the bucket.
- Since the bucket has no equal key, the entry is inserted.
- `add()` returns `true`.

**Step 2: Add `"Python"`**

```java
languages.add("Python");
```

- The hash and bucket are calculated for `"Python"`.
- If no equal key exists, it is inserted.
- `add()` returns `true`.

**Step 3: Add `"Java"` again**

```java
languages.add("Java");
```

- The hash is calculated again.
- The map checks the appropriate bucket.
- The existing key is identified using hash comparison and `equals()`.
- The existing mapping is retained rather than creating a duplicate key.
- `add()` returns `false`.

## 4. How do hashCode() and equals() work?

Both methods are important for maintaining uniqueness in a `HashSet`.

### hashCode()

The `hashCode()` method returns an integer hash value for an object.

The hash helps the underlying `HashMap` identify the bucket to search.

```java
String language = "Java";

System.out.println(language.hashCode());
```

The exact hash value depends on the object's implementation. For `String`, the hash is calculated from its character sequence.

**Important:** A hash code is not necessarily unique. Different objects can have the same hash code; this is called a *hash collision*.

### equals()

The `equals()` method determines whether two objects should be considered equal.

When a matching hash leads the `HashMap` to an existing key, it uses equality checks to determine whether the keys represent the same element.

For custom objects, override both `equals()` and `hashCode()` consistently.

### Example: HashSet with custom objects

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

class Employee {
    private final int id;
    private final String name;

    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Employee)) {
            return false;
        }

        Employee other = (Employee) obj;

        return id == other.id
                && Objects.equals(name, other.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}

public class Main {
    public static void main(String[] args) {
        Set<Employee> employees = new HashSet<>();

        employees.add(new Employee(101, "Danish"));
        employees.add(new Employee(101, "Danish"));
        employees.add(new Employee(102, "Rahul"));

        System.out.println(employees.size());
    }
}
```

Output:

```text
2
```

Although two different `Employee` objects are created for Danish, they are considered equal because their `id` and `name` fields match.

Therefore, only one of them is stored.

**Interview point:** If you override `equals()`, you should also override `hashCode()` so that equal objects always have equal hash codes.

## 5. What happens during a hash collision?

A collision occurs when different elements map to the same bucket.

For example, consider a custom class that always returns the same hash code:

```java
import java.util.HashSet;
import java.util.Set;

class Item {
    private final int id;

    Item(int id) {
        this.id = id;
    }

    @Override
    public int hashCode() {
        return 1;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Item)) {
            return false;
        }

        return id == ((Item) obj).id;
    }
}

public class Main {
    public static void main(String[] args) {
        Set<Item> items = new HashSet<>();

        items.add(new Item(1));
        items.add(new Item(2));
        items.add(new Item(1));

        System.out.println(items.size());
    }
}
```

Output:

```text
2
```

All three insertions have the same hash code, but the two objects with `id = 1` are equal. Therefore, only two unique elements remain.

In modern OpenJDK implementations, the underlying `HashMap` can store colliding entries in a linked structure and, under certain conditions, convert a heavily populated bucket into a red-black tree.

A poor `hashCode()` implementation can degrade performance even though `equals()` still preserves uniqueness.

## 6. How does HashSet check whether an element exists?

The `contains()` method delegates to the underlying map's key lookup.

```java
import java.util.HashSet;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        Set<String> languages = new HashSet<>();

        languages.add("Java");
        languages.add("Python");

        System.out.println(languages.contains("Java"));
        System.out.println(languages.contains("C++"));
    }
}
```

Output:

```text
true
false
```

The map uses the hash to find the relevant bucket and equality checks to determine whether the key exists.

## 7. Time complexity of HashSet

| Operation | Average case | Worst case |
|---|---:|---:|
| `add()` | \(O(1)\) | \(O(n)\) in general |
| `contains()` | \(O(1)\) | \(O(n)\) in general |
| `remove()` | \(O(1)\) | \(O(n)\) in general |
| Iteration | \(O(n + c)\) | \(O(n + c)\) |

Here, \(n\) is the number of elements and \(c\) is the backing table capacity.

Modern `HashMap` implementations can improve some heavily collided buckets to tree-based lookups, but worst-case performance depends on the implementation and the behavior of key comparison.

## 8. HashSet vs HashMap

| Feature | HashSet | HashMap |
|---|---|---|
| Interface | `Set` | `Map` |
| Stores | Unique elements | Key-value pairs |
| Internal implementation | Uses `HashMap` | Uses a hash table |
| Duplicate handling | Rejects equal elements | Replaces the value for an existing equal key |
| Null support | One `null` element | One `null` key; multiple null values |
| Typical use | Uniqueness and membership checks | Key-based lookup |

Example:

```java
Set<String> set = new HashSet<>();
set.add("Java");

Map<String, Integer> map = new HashMap<>();
map.put("Java", 1);
```

## 9. Is HashSet thread-safe?

No. `HashSet` is not thread-safe for concurrent modification.

For concurrent access, choose an appropriate solution based on your requirements.

### Option 1: Synchronized set

```java
Set<String> set = Collections.synchronizedSet(
    new HashSet<>()
);
```

When iterating, synchronize on the returned set:

```java
synchronized (set) {
    for (String value : set) {
        System.out.println(value);
    }
}
```

### Option 2: ConcurrentHashMap-backed set

```java
Set<String> set = ConcurrentHashMap.newKeySet();

set.add("Java");
set.add("Spring Boot");
```

`ConcurrentHashMap.newKeySet()` provides a concurrent set implementation suitable for many multi-threaded use cases. Unlike `HashSet`, it does not allow `null` elements.

There is no standard JDK class named `ConcurrentHashSet`; a concurrent set can be created using `ConcurrentHashMap.newKeySet()`.

## 10. Common interview questions

### Q1. Why does HashSet not allow duplicates?

Because it stores elements as keys in an underlying `HashMap`. A map cannot hold multiple distinct mappings for keys that are equal according to its key-equality rules.

### Q2. Can HashSet store null?

Yes. A `HashSet` can store one `null` element.

### Q3. What happens if two objects have the same hash code?

They may land in the same bucket. The `HashMap` uses equality checks to distinguish equal keys from different keys.

### Q4. What happens if we override equals() but not hashCode()?

Equal objects may produce different hash codes, causing the hash-based collection to search different buckets and potentially store what should be duplicate elements.

### Q5. Does HashSet maintain insertion order?

No. Use `LinkedHashSet` if you need insertion-order iteration.

### Q6. What happens if an object's fields change after insertion?

If fields used by `equals()` and `hashCode()` change, lookups and removal can fail because the object may now hash to a different bucket. Avoid mutating equality-defining fields while an object is stored in a hash-based collection.

### Q7. What is the difference between HashSet and TreeSet?

- `HashSet` generally provides average \(O(1)\) basic operations and does not guarantee iteration order.
- `TreeSet` maintains elements in sorted order and typically provides \(O(\log n)\) basic operations.

## 11. Quick revision

- `HashSet` implements the `Set` interface.
- It uses a `HashMap` internally.
- Elements are keys; a shared dummy object is used as the value.
- `hashCode()` helps locate the bucket.
- `equals()` determines whether keys are equal.
- Hash collisions do not automatically mean elements are duplicates.
- One `null` element is allowed.
- Iteration order is not guaranteed.
- `HashSet` is not thread-safe.
- Override `equals()` and `hashCode()` together for custom element classes.


---


# HashMap vs Hashtable in Java

## 1. Introduction

Both `HashMap` and `Hashtable` implement the `Map` interface and store data as key-value pairs.

The main differences are synchronization, null handling, performance, and legacy status.

```java
import java.util.HashMap;
import java.util.Hashtable;
import java.util.Map;

Map<String, String> map = new HashMap<>();

Hashtable<String, String> table = new Hashtable<>();
```

## 2. Key differences

| Feature | HashMap | Hashtable |
|---|---|---|
| Thread safety | Not synchronized | Synchronized methods |
| Null key | Allows one | Not allowed |
| Null values | Allowed | Not allowed |
| Performance | Generally faster in single-threaded use | Often slower due to synchronization overhead |
| Iteration | Iterators are generally fail-fast | Legacy `Enumeration` is not fail-fast |
| Introduced | Java 1.2 | Java 1.0 |
| Legacy status | Modern Collections Framework class | Legacy class |
| Recommended for concurrent use | No, without external coordination | Usually prefer `ConcurrentHashMap` |

## 3. Synchronization

### HashMap

`HashMap` is not synchronized. Concurrent modifications by multiple threads require suitable coordination.

```java
Map<String, Integer> map = new HashMap<>();

map.put("Java", 10);
map.put("Spring", 20);
```

For single-threaded use, or when access is properly coordinated, `HashMap` is a common choice.

### Hashtable

`Hashtable` synchronizes its individual methods.

```java
Hashtable<String, Integer> table = new Hashtable<>();

table.put("Java", 10);
table.put("Spring", 20);
```

However, synchronized methods do not make every multi-step operation or iteration automatically atomic. Additional synchronization may be required for compound operations.

## 4. Null keys and values

`HashMap` allows one null key and multiple null values.

```java
Map<String, String> map = new HashMap<>();

map.put(null, "Unknown");
map.put("Java", null);
map.put("Spring", null);

System.out.println(map.size()); // 3
```

`Hashtable` does not permit null keys or null values.

```java
Hashtable<String, String> table = new Hashtable<>();

// Throws NullPointerException
table.put(null, "Unknown");

// Also throws NullPointerException
table.put("Java", null);
```

## 5. Iteration

A `HashMap` iterator is generally fail-fast on a best-effort basis. Structural modification outside the iterator's permitted operations can cause `ConcurrentModificationException`.

```java
Map<String, Integer> map = new HashMap<>();

map.put("Java", 1);
map.put("Spring", 2);

for (String key : map.keySet()) {
    System.out.println(key);
}
```

`Hashtable` supports legacy `Enumeration` methods such as `keys()` and `elements()`. These enumerations are not fail-fast.

```java
Hashtable<String, Integer> table = new Hashtable<>();

table.put("Java", 1);
table.put("Spring", 2);

var keys = table.keys();

while (keys.hasMoreElements()) {
    System.out.println(keys.nextElement());
}
```

Note that `Hashtable` also provides collection-view iterators; these should not be confused with its legacy enumerations.

## 6. Performance

`HashMap` often performs better when synchronization is unnecessary because it avoids the synchronization overhead associated with `Hashtable`.

Actual performance depends on workload, contention, and implementation details.

For concurrent applications, `ConcurrentHashMap` is generally a better choice than `Hashtable`.

## 7. Interview questions

**Q1. Is Hashtable thread-safe?**

Its individual methods are synchronized, but compound operations and iteration may require additional coordination.

**Q2. Why does Hashtable not allow null?**

Its API explicitly rejects null keys and values. Do not assume that thread safety inherently requires rejecting null.

**Q3. What should we use instead of Hashtable?**

Use `HashMap` when external synchronization is unnecessary. For concurrent access, consider `ConcurrentHashMap`.

## 8. Quick revision

- `HashMap`: unsynchronized, permits null keys and values.
- `Hashtable`: legacy synchronized map, rejects null keys and values.
- `HashMap` iterators are generally fail-fast.
- Legacy `Hashtable` enumerations are not fail-fast.
- `ConcurrentHashMap` is usually preferred for concurrent key-value storage.


---


# HashMap vs LinkedHashMap in Java

## 1. Introduction

Both `HashMap` and `LinkedHashMap` implement the `Map` interface and store key-value pairs.

The primary difference is **iteration order**.

- `HashMap` does not guarantee iteration order.
- `LinkedHashMap` maintains a predictable iteration order, usually insertion order by default.

```java
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

Map<Integer, String> map = new HashMap<>();

Map<Integer, String> linkedMap = new LinkedHashMap<>();
```

## 2. Key differences

| Feature | HashMap | LinkedHashMap |
|---|---|---|
| Iteration order | Not guaranteed | Insertion order by default |
| Internal structure | Hash table | Hash table plus doubly linked entry chain |
| Average `put()` / `get()` | \(O(1)\) | \(O(1)\) |
| Memory usage | Generally lower | Generally higher |
| Null keys | One | One |
| Null values | Allowed | Allowed |
| Thread safety | Not synchronized | Not synchronized |
| Access-order mode | No | Yes |

## 3. Iteration order example

```java
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        Map<Integer, String> hashMap = new HashMap<>();
        hashMap.put(3, "Three");
        hashMap.put(1, "One");
        hashMap.put(2, "Two");

        Map<Integer, String> linkedMap = new LinkedHashMap<>();
        linkedMap.put(3, "Three");
        linkedMap.put(1, "One");
        linkedMap.put(2, "Two");

        System.out.println("HashMap: " + hashMap);
        System.out.println("LinkedHashMap: " + linkedMap);
    }
}
```

The `LinkedHashMap` output follows insertion order:

```text
LinkedHashMap: {3=Three, 1=One, 2=Two}
```

The `HashMap` output may appear in a particular order, but that order is not guaranteed by its API.

## 4. How LinkedHashMap works internally

`LinkedHashMap` extends `HashMap` and maintains links between entries in a doubly linked chain.

Conceptually:

```text
Hash table
    |
    +---- Entry: 3 = Three
    +---- Entry: 1 = One
    +---- Entry: 2 = Two

Linked entry chain:

3 = Three <-> 1 = One <-> 2 = Two
```

The hash table enables efficient key lookup, while the linked chain preserves the iteration order.

The extra links require additional memory compared with a typical `HashMap` entry.

## 5. Insertion order vs access order

By default, `LinkedHashMap` uses insertion order. It can also be configured to use access order, which is useful for implementing an LRU (Least Recently Used) cache.

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        Map<Integer, String> map = new LinkedHashMap<>(
            16, 0.75f, true
        );

        map.put(1, "One");
        map.put(2, "Two");
        map.put(3, "Three");

        map.get(1);

        System.out.println(map);
    }
}
```

Output:

```text
{2=Two, 3=Three, 1=One}
```

Because access-order mode is enabled, accessing key `1` moves its entry to the end of the iteration order.

## 6. Performance

Both collections provide average constant-time basic operations when hash distribution is good.

`LinkedHashMap` generally has extra memory overhead because each entry maintains order-related links. Iteration through its entries is efficient and follows the linked chain.

## 7. When should you use each?

| Requirement | Recommended collection |
|---|---|
| Store key-value pairs without needing a specific iteration order | `HashMap` |
| Preserve insertion order | `LinkedHashMap` |
| Build an access-order map or LRU cache | `LinkedHashMap` |
| Need sorted keys | `TreeMap` |

## 8. Interview questions

**Q1. Is LinkedHashMap a subclass of HashMap?**

Yes. `LinkedHashMap` extends `HashMap`.

**Q2. Does LinkedHashMap allow null keys and values?**

Yes, it permits one null key and multiple null values.

**Q3. Is LinkedHashMap thread-safe?**

No. It requires external coordination or an appropriate alternative when shared mutable access is concurrent.

**Q4. Does LinkedHashMap always maintain insertion order?**

No. Its default mode is insertion order, but it can be configured for access order.

## 9. Quick revision

- `HashMap` does not guarantee iteration order.
- `LinkedHashMap` maintains insertion order by default.
- `LinkedHashMap` extends `HashMap`.
- It uses additional links to maintain entry order.
- Access-order mode can support LRU cache implementations.
- Both allow null keys and values and are not inherently thread-safe.


---

# HashMap vs ConcurrentHashMap in Java

## 1. Introduction

Both `HashMap` and `ConcurrentHashMap` store key-value pairs through the `Map` interface.

The key difference is that `ConcurrentHashMap` is designed for concurrent access by multiple threads, whereas `HashMap` does not provide built-in thread safety.

```java
import java.util.HashMap;
import java.util.concurrent.ConcurrentHashMap;

HashMap<String, Integer> map = new HashMap<>();

ConcurrentHashMap<String, Integer> concurrentMap =
    new ConcurrentHashMap<>();
```

## 2. Key differences

| Feature | HashMap | ConcurrentHashMap |
|---|---|---|
| Thread safety | Not synchronized | Designed for concurrent access |
| Concurrent reads and writes | Requires coordination | Supports concurrent operations |
| Null keys | One allowed | Not allowed |
| Null values | Allowed | Not allowed |
| Iteration | Generally fail-fast | Weakly consistent |
| Compound atomic operations | Require coordination | Provides atomic methods such as `putIfAbsent()` and `compute()` |
| Performance | Often simpler when concurrency is unnecessary | Designed for concurrent workloads |
| Introduced | Java 1.2 | Java 1.5 |

## 3. Thread safety

### HashMap

`HashMap` is suitable when a map is used by one thread or when shared access is properly coordinated.

```java
HashMap<String, Integer> map = new HashMap<>();

map.put("Java", 10);
map.put("Spring", 20);
```

Concurrent access involving modifications without proper coordination can lead to race conditions and unpredictable results.

### ConcurrentHashMap

`ConcurrentHashMap` is designed to allow multiple threads to access and update the map safely.

```java
ConcurrentHashMap<String, Integer> map =
    new ConcurrentHashMap<>();

map.put("Java", 10);
map.put("Spring", 20);
```

It does not lock the entire map for every operation. Modern implementations use a combination of atomic operations, synchronization for particular updates, and other concurrency mechanisms.

This design enables better concurrency than a single globally synchronized map in many workloads.

## 4. Null keys and values

`HashMap` allows one null key and multiple null values.

```java
HashMap<String, String> map = new HashMap<>();

map.put(null, "Unknown");
map.put("Java", null);
```

`ConcurrentHashMap` does not allow null keys or null values.

```java
ConcurrentHashMap<String, String> map =
    new ConcurrentHashMap<>();

// Throws NullPointerException
map.put(null, "Unknown");

// Also throws NullPointerException
map.put("Java", null);
```

One practical benefit is that a null result from `ConcurrentHashMap.get(key)` unambiguously means that no mapping is present.

## 5. Iteration: fail-fast vs weakly consistent

A `HashMap` iterator is generally fail-fast on a best-effort basis. Structural modification outside the iterator's permitted operations can result in `ConcurrentModificationException`.

`ConcurrentHashMap` iterators are weakly consistent. They:

- Do not throw `ConcurrentModificationException` merely because another thread modifies the map during iteration.
- May reflect some updates made after iteration begins.
- Do not represent a guaranteed snapshot of the map at one instant.

Example:

```java
ConcurrentHashMap<Integer, String> map =
    new ConcurrentHashMap<>();

map.put(1, "Java");
map.put(2, "Spring");

for (var entry : map.entrySet()) {
    System.out.println(entry);

    map.put(3, "Docker");
}
```

This iteration does not fail simply because the map is concurrently modified. Whether a particular new entry is observed during the iteration is not guaranteed.

## 6. Atomic operations

One major advantage of `ConcurrentHashMap` is its atomic compound-operation APIs.

For example, avoid the following pattern when multiple threads may update the same key:

```java
// Not an atomic compound operation
if (!map.containsKey("Java")) {
    map.put("Java", 1);
}
```

Use `putIfAbsent()` instead:

```java
ConcurrentHashMap<String, Integer> map =
    new ConcurrentHashMap<>();

Integer previous = map.putIfAbsent("Java", 1);

System.out.println(previous);
System.out.println(map.get("Java"));
```

Output:

```text
null
1
```

`putIfAbsent()` atomically inserts the value only when the key does not already have a mapping.

For more complex updates, methods such as `compute()`, `computeIfAbsent()`, and `merge()` can be useful.

## 7. Performance

`HashMap` may be a good choice when concurrency is unnecessary because it has less concurrency-related overhead.

`ConcurrentHashMap` is designed to support concurrent operations efficiently. It is not necessarily slower in every scenario: performance depends on the number of threads, contention, and workload.

If a map is shared across threads and requires concurrent updates, use a suitable concurrent collection rather than relying on an ordinary `HashMap`.

## 8. When should you use each?

| Scenario | Recommended choice |
|---|---|
| Local map used by one thread | `HashMap` |
| Shared map with coordinated access | `HashMap` with suitable synchronization, if appropriate |
| Shared map with concurrent reads and writes | `ConcurrentHashMap` |
| Need atomic insertion or update operations | `ConcurrentHashMap` |
| Need null keys or values | `HashMap`, with appropriate coordination if shared |

## 9. Interview questions

**Q1. Is ConcurrentHashMap completely lock-free?**

No. It uses atomic operations and, when needed, synchronization for updates. It should not be described as entirely lock-free.

**Q2. Does ConcurrentHashMap throw ConcurrentModificationException during iteration?**

Its iterators are weakly consistent and do not throw that exception merely because concurrent modifications occur.

**Q3. Why does ConcurrentHashMap reject null?**

Rejecting null keys and values avoids ambiguity between an absent mapping and a mapping whose value is null, particularly for concurrent lookups.

**Q4. Is ConcurrentHashMap always faster than HashMap?**

No. It is designed for concurrent access. For single-threaded workloads, `HashMap` may have less overhead.

**Q5. Is `putIfAbsent()` thread-safe?**

Yes. `ConcurrentHashMap.putIfAbsent()` performs the conditional insertion atomically.

## 10. Quick revision

- `HashMap` is not thread-safe.
- `ConcurrentHashMap` supports concurrent access.
- Neither `put()` nor `get()` on a plain `HashMap` makes concurrent modification safe.
- `ConcurrentHashMap` rejects null keys and values.
- Its iterators are weakly consistent.
- Use `putIfAbsent()`, `computeIfAbsent()`, and related methods for atomic updates.
- Prefer `ConcurrentHashMap` for many shared-map concurrency use cases.


---

# Can We Insert a Null Key in HashMap and Hashtable?

## 1. Short answer

**`HashMap` allows one null key, whereas `Hashtable` does not allow null keys.**

| Feature | HashMap | Hashtable |
|---|---|---|
| Null key | Allowed (one) | Not allowed |
| Null value | Allowed | Not allowed |
| Behavior on inserting null | Accepts it | Throws `NullPointerException` |

## 2. Example using HashMap

```java
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();

        map.put(null, "Danish");
        map.put("Java", "Programming");
        map.put(null, "Updated");

        System.out.println(map);
        System.out.println(map.get(null));
        System.out.println(map.size());
    }
}
```

Possible output:

```text
{null=Updated, Java=Programming}
Updated
2
```

**Explanation:**

- `HashMap` accepts a null key.
- A map cannot have multiple separate entries for the same key, so the second null-key insertion replaces the first value.
- Therefore, only one null-key entry exists.

## 3. Example using Hashtable

```java
import java.util.Hashtable;

public class Main {
    public static void main(String[] args) {
        Hashtable<String, String> table = new Hashtable<>();

        table.put(null, "Danish");
    }
}
```

Output: `NullPointerException`

`Hashtable` explicitly rejects null keys and null values.

## 4. How does HashMap handle a null key internally?

In OpenJDK's `HashMap` implementation, a null key is assigned a hash value of `0`.

Conceptually:

```java
int hash = (key == null) ? 0 : hash(key);
```

The hash value helps determine the bucket in which the entry is stored.

**Important:** A null key does not mean that every key with hash code `0` is the same key. Hash collisions are resolved by checking key equality.

Also, do not assume that `Hashtable` handles null keys by assigning them hash code `0`: it rejects them before normal key hashing.

## 5. Interview takeaway

- `HashMap`: allows one null key and multiple null values.
- `Hashtable`: allows neither null keys nor null values.
- Inserting a null key twice into a `HashMap` replaces its existing value.
- `ConcurrentHashMap` also rejects null keys and null values.


---

# How to Create an Immutable Map in Java

## 1. What is an immutable map?

An immutable map cannot be modified after creation. Depending on how it is constructed, its entries cannot be added, removed, or replaced.

Java provides several ways to create maps that cannot be modified through the returned reference.

## 2. Using Map.of() — Java 9+

This is the simplest option for a small map.

```java
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        Map<String, String> map = Map.of(
            "key1", "value1",
            "key2", "value2"
        );

        System.out.println(map);

        // Throws UnsupportedOperationException
        // map.put("key3", "value3");
    }
}
```

Key characteristics:

- The returned map is unmodifiable.
- Null keys and null values are not allowed.
- Duplicate keys are rejected.
- Suitable for a fixed set of entries.

## 3. Using Map.ofEntries() — Java 9+

Use this when you want to create a map from entry objects.

```java
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        Map<String, String> map = Map.ofEntries(
            Map.entry("key1", "value1"),
            Map.entry("key2", "value2"),
            Map.entry("key3", "value3")
        );

        System.out.println(map);
    }
}
```

`Map.entry()` also rejects null keys and values.

## 4. Using Collections.unmodifiableMap()

This method returns an unmodifiable **view** of an existing map.

```java
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        Map<String, String> original = new HashMap<>();

        original.put("key1", "value1");
        original.put("key2", "value2");

        Map<String, String> readOnly =
            Collections.unmodifiableMap(original);

        // Throws UnsupportedOperationException
        // readOnly.put("key3", "value3");

        original.put("key3", "value3");

        System.out.println(readOnly);
    }
}
```

Output:

```text
{key1=value1, key2=value2, key3=value3}
```

**Important:** The original map remains modifiable. Changes made through the original reference are visible in the unmodifiable view.

To create a defensive, unmodifiable copy:

```java
Map<String, String> immutable =
    Collections.unmodifiableMap(new HashMap<>(original));
```

This prevents changes to the map structure from being reflected through another reference to the original map. It is still a shallow copy: mutable key or value objects are not automatically made immutable.

## 5. Using Google's Guava ImmutableMap

If the Guava library is already part of your project, you can use `ImmutableMap`.

```java
import com.google.common.collect.ImmutableMap;

public class Main {
    public static void main(String[] args) {
        ImmutableMap<String, String> map = ImmutableMap.of(
            "key1", "value1",
            "key2", "value2"
        );

        System.out.println(map);
    }
}
```

For more entries:

```java
ImmutableMap<String, String> map =
    ImmutableMap.<String, String>builder()
        .put("key1", "value1")
        .put("key2", "value2")
        .put("key3", "value3")
        .build();
```

Guava is an external dependency, unlike Java's built-in `Map.of()` methods.

## 6. Comparison

| Method | Available since | Allows null keys/values? | Behavior |
|---|---|---|---|
| `Map.of()` | Java 9 | No | Unmodifiable map |
| `Map.ofEntries()` | Java 9 | No | Unmodifiable map |
| `Collections.unmodifiableMap()` | Java 1.2 | Depends on backing map | Unmodifiable view |
| Guava `ImmutableMap` | External library | No | Immutable map implementation |

## 7. Interview questions

**Q1. Is `Collections.unmodifiableMap()` truly immutable?**

Not necessarily. It is an unmodifiable view, and changes to the backing map can still appear in the view.

**Q2. Can `Map.of()` contain null values?**

No. It throws `NullPointerException` for null keys or values.

**Q3. What is the difference between unmodifiable and immutable?**

Unmodifiable means callers cannot mutate the map through that reference. Immutable means its observable state cannot change after construction. An unmodifiable view of a mutable map is not necessarily immutable.

**Q4. Which approach should I use?**

Use `Map.of()` for small, fixed maps on Java 9 or later. Use a defensive copy plus an unmodifiable wrapper when you need to expose a copy of an existing map. Consider Guava `ImmutableMap` if the dependency is appropriate.


---

# Immutable Classes in Java

## 1. What is an immutable class?

An immutable class is a class whose object state cannot change after the object has been created.

For example:

```java
String name = "Java";

name.concat(" Programming");

System.out.println(name);
```

Output:

```text
Java
```

The `concat()` method returns a new string rather than modifying the original string.

To retain the result:

```java
name = name.concat(" Programming");

System.out.println(name);
```

Output:

```text
Java Programming
```

The variable can reference a different object, but the original `String` object remains unchanged.

## 2. Common built-in immutable classes

| Class | Package | Purpose |
|---|---|---|
| `String` | `java.lang` | Text |
| `Boolean` | `java.lang` | Boolean values |
| `Byte` | `java.lang` | Byte values |
| `Short` | `java.lang` | Short integer values |
| `Integer` | `java.lang` | Integer values |
| `Long` | `java.lang` | Long integer values |
| `Float` | `java.lang` | Floating-point values |
| `Double` | `java.lang` | Double-precision values |
| `Character` | `java.lang` | Character values |
| `BigInteger` | `java.math` | Arbitrary-precision integers |
| `BigDecimal` | `java.math` | Arbitrary-precision decimal arithmetic |
| `LocalDate` | `java.time` | Date without a time zone |
| `LocalTime` | `java.time` | Time without a date |
| `LocalDateTime` | `java.time` | Date and time without a time zone |
| `Instant` | `java.time` | Point on the UTC timeline |
| `Duration` | `java.time` | Time-based amount |
| `Period` | `java.time` | Date-based amount |
| `Optional` | `java.util` | Container for a possibly absent value |

These classes are designed so that their own state cannot be modified through their public APIs. Some classes, such as `Optional`, may hold a reference to another object, and that referenced object can itself be mutable.

## 3. Why are wrapper classes immutable?

Java wrapper classes such as `Integer`, `Long`, and `Boolean` represent primitive values as objects.

```java
Integer number = 10;

number = number + 5;

System.out.println(number);
```

Output:

```text
15
```

The original `Integer` object is not modified. The expression produces a value that is boxed into an `Integer`, and the variable is assigned the resulting reference.

## 4. Why is BigDecimal immutable?

`BigDecimal` is commonly used when decimal precision matters, such as financial calculations.

```java
import java.math.BigDecimal;

public class Main {
    public static void main(String[] args) {
        BigDecimal amount = new BigDecimal("100.50");

        BigDecimal updated = amount.add(
            new BigDecimal("20.25")
        );

        System.out.println(amount);
        System.out.println(updated);
    }
}
```

Output:

```text
100.50
120.75
```

The `add()` method returns a new `BigDecimal`; it does not modify the original object.

Using a string constructor avoids the initial binary floating-point approximation associated with a `double` literal.

## 5. How to create your own immutable class

Common practices include:

1. Declare the class `final`, unless you have another robust design that prevents mutable subclasses.
2. Make fields `private final`.
3. Initialize all fields through the constructor.
4. Do not provide setters.
5. Make defensive copies of mutable inputs.
6. Do not expose mutable internal objects directly.

Example:

```java
import java.util.ArrayList;
import java.util.List;

public final class Student {
    private final String name;
    private final List<String> subjects;

    public Student(String name, List<String> subjects) {
        this.name = name;
        this.subjects = new ArrayList<>(subjects);
    }

    public String getName() {
        return name;
    }

    public List<String> getSubjects() {
        return List.copyOf(subjects);
    }
}
```

The constructor copies the supplied list, and the getter returns an unmodifiable copy. This prevents callers from changing the student's stored subject list through an external reference.

The example assumes the list elements are themselves immutable or that shallow copying is sufficient.

## 6. Advantages of immutability

- Easier reasoning and debugging.
- Safer sharing between threads when the whole reachable state is immutable.
- Stable values for hash-based collections.
- Reduced risk of accidental modification.
- Useful for value objects and configuration data.

## 7. Interview questions

**Q1. Is String immutable in Java?**

Yes. String objects cannot be modified after creation.

**Q2. Is every final class immutable?**

No. `final` prevents subclassing; it does not prevent the class from changing its own fields.

**Q3. Is every object referenced by an immutable object also immutable?**

No. Immutability is not automatically transitive. Defensive copying or immutable members may be required.

**Q4. Is Optional immutable?**

Yes, the `Optional` container is immutable, but an object referenced by it can still be mutable.

**Q5. Why are immutable objects useful as HashMap keys?**

Their equality and hash code remain stable when their state cannot change, reducing the risk of failed lookups after insertion.



---



# Database Indexes: Advantages and Disadvantages

## 1. What is an index?

A database index is a data structure that helps the database find rows without scanning every row of a table.

An index is similar to the index at the back of a book: instead of reading every page, you can locate the relevant topic and jump to its location.

Consider an `Employee` table:

| employee_id | name | department |
|---:|---|---|
| 101 | Danish | IT |
| 102 | Rahul | HR |
| 103 | Ayesha | IT |
| 104 | Imran | Finance |

Without a suitable index, a query searching for an employee by name may require a table scan.

```sql
SELECT *
FROM Employee
WHERE name = 'Ayesha';
```

With an index on `name`, the database may be able to find the matching row more efficiently.

```sql
CREATE INDEX idx_employee_name
ON Employee(name);
```

The database optimizer decides whether using the index is more efficient than scanning the table.

## 2. How does an index work?

Many relational databases use B-tree or B+ tree structures for common indexes. Some databases and workloads also support hash indexes, bitmap indexes, full-text indexes, and other index types.

A simplified B-tree-style index looks like this:

```text
             [Rahul]
             /     \
       [Danish]   [Ayesha, Imran]
```

This diagram is illustrative rather than a precise database implementation.

The index stores searchable key information and references that help the database locate the associated rows. The exact structure depends on the database engine and index type.

## 3. Types of indexes

### Primary key index

A primary key uniquely identifies each row. Most relational database engines create an index to enforce the primary key constraint, although the physical implementation differs.

```sql
CREATE TABLE Employee (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100)
);
```

### Unique index

A unique index prevents duplicate indexed key values according to the database's rules.

```sql
CREATE UNIQUE INDEX idx_employee_email
ON Employee(email);
```

The exact treatment of nulls in unique indexes varies by database engine.

### Clustered index

A clustered index determines how table rows are organized around the indexed key in database engines that support this concept.

Some engines allow only one clustered index per table because a table can have only one primary physical organization of this kind.

### Non-clustered index

A non-clustered index stores indexed key values and references to the corresponding rows. A table can generally have multiple non-clustered indexes.

### Composite index

A composite index covers multiple columns.

```sql
CREATE INDEX idx_employee_department_name
ON Employee(department, name);
```

This can help queries that filter by `department` or by `department` and `name`, depending on the database engine and query.

Column order matters: an index on `(department, name)` generally does not provide the same benefits for a query filtering only by `name`.

### Full-text index

A full-text index supports text-search operations over large text fields. Its behavior and syntax depend on the database engine.

## 4. Advantages of indexes

### 1. Faster searches

Indexes can reduce the number of rows or data pages that need to be examined.

### 2. Improved joins

Indexes on join columns can help the optimizer find matching rows efficiently.

### 3. Efficient sorting and grouping

Suitable indexes may help satisfy `ORDER BY`, grouping, or range-search requirements without additional expensive operations.

### 4. Constraint enforcement

Unique indexes can enforce uniqueness, while indexes support primary key constraints in common relational database implementations.

## 5. Disadvantages of indexes

### 1. Additional storage

Indexes consume disk space and may also require memory for frequently accessed index pages.

### 2. Slower writes

`INSERT`, `UPDATE`, and `DELETE` operations may need to update affected indexes. Tables with many indexes can therefore incur additional write overhead.

### 3. Maintenance overhead

Indexes may need maintenance, statistics updates, or rebuilding depending on the database engine and workload.

### 4. Not every query benefits

An index may be ineffective when a query returns a large percentage of the table, when the indexed columns are unsuitable, or when the optimizer estimates that a table scan is cheaper.

## 6. Index vs table scan

| Feature | Index access | Table scan |
|---|---|---|
| Data access | Uses an index to locate qualifying rows | Examines table rows or pages |
| Best suited for | Selective lookups and suitable ranges | Small tables or queries reading many rows |
| Storage overhead | Requires index storage | No additional index required for the scan itself |
| Write overhead | Indexes must be maintained | No index-maintenance cost if no relevant index exists |

A table scan is not inherently bad. It can be the fastest option when a query needs most of the rows.

## 7. How to check whether an index is being used

The exact syntax depends on the database.

For PostgreSQL:

```sql
EXPLAIN
SELECT *
FROM Employee
WHERE name = 'Ayesha';
```

For MySQL:

```sql
EXPLAIN
SELECT *
FROM Employee
WHERE name = 'Ayesha';
```

`EXPLAIN` shows the optimizer's planned access strategy. PostgreSQL also supports `EXPLAIN ANALYZE` to execute the query and report actual execution details.

Do not assume that creating an index guarantees its use. Check the execution plan and measure performance against representative data.

## 8. When should you create an index?

Indexes are particularly worth considering for:

- Frequently filtered columns.
- Columns used in joins.
- Frequently sorted or ranged columns.
- Columns that need uniqueness enforced.
- Composite filters used by important queries.

Avoid creating indexes indiscriminately. Consider data distribution, query patterns, write frequency, index size, and execution plans.

## 9. Interview questions

**Q1. Does an index always improve query performance?**

No. It can increase storage and write costs, and a table scan may be more efficient for some queries.

**Q2. Can a table have multiple indexes?**

Yes. A table can have multiple indexes, subject to database-specific constraints.

**Q3. What is a composite index?**

An index built on two or more columns. Its column order affects which query predicates can benefit.

**Q4. What is the difference between a clustered and non-clustered index?**

A clustered index organizes rows according to an indexed key in engines supporting clustered storage. A non-clustered index is a separate access structure that points to the corresponding rows.

**Q5. Does a primary key always create an index?**

Common relational database systems create an index to support primary key enforcement, but the exact physical implementation depends on the database engine.

## 10. Quick revision

- An index helps locate rows efficiently.
- B-tree-style indexes are common for general-purpose relational queries.
- Indexes can improve filtering, joins, and sorting.
- Indexes require storage and increase write-maintenance costs.
- Composite index column order matters.
- Use execution plans and real measurements to evaluate index effectiveness.


---

# Programming to an Interface vs Implementation in Java

## 1. What Is the Difference Between These Two Syntaxes?

```java
List<String> list = new ArrayList<>();

ArrayList<String> alist = new ArrayList<>();
```

Both statements create an `ArrayList` object. The difference is the **reference type** used to refer to that object.

### Syntax 1: Programming to an interface

```java
List<String> list = new ArrayList<>();
```

- `List` is an interface.
- `ArrayList` is the implementation class.
- The reference exposes methods defined by `List` and inherited from its parent interfaces.
- The implementation can be changed to another `List` implementation with minimal changes.

For example:

```java
List<String> list = new LinkedList<>();
```

This is also valid because `LinkedList` implements `List`.

### Syntax 2: Programming to an implementation

```java
ArrayList<String> alist = new ArrayList<>();
```

- `ArrayList` is the concrete implementation class.
- The reference exposes methods available on `ArrayList`, including its public methods that are not declared by `List`.
- The code is more tightly coupled to `ArrayList`, making it less flexible if a different implementation is needed.

## 2. Example: Why Prefer Programming to an Interface?

### Using the interface

```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();

        names.add("Ali");
        names.add("Sara");

        System.out.println(names);

        names = new LinkedList<>();
        names.add("Ahmed");

        System.out.println(names);
    }
}
```

Output:

```text
[Ali, Sara]
[Ahmed]
```

The reference type remains `List`, while the implementation can change from `ArrayList` to `LinkedList`.

### Using the implementation

```java
ArrayList<String> names = new ArrayList<>();
```

You cannot directly assign a `LinkedList` to this reference:

```java
names = new LinkedList<>(); // Compilation error
```

The object assigned to an `ArrayList` reference must be compatible with `ArrayList`.

## 3. Key Differences

| Feature | `List<String> list` | `ArrayList<String> alist` |
|---|---|---|
| Reference type | Interface | Concrete class |
| Object created | `ArrayList` | `ArrayList` |
| Access to `List` methods | Yes | Yes |
| Access to `ArrayList`-specific methods | Not directly through the `List` reference | Yes |
| Can assign a `LinkedList`? | Yes | No |
| Coupling | Lower | Higher |
| Flexibility | More flexible | More implementation-specific |

Both declarations create the same kind of object in these examples. The reference type determines which methods are available at compile time.

## 4. When Should You Use Each?

**Prefer programming to an interface:**

```java
List<String> names = new ArrayList<>();
Map<Integer, String> students = new HashMap<>();
Set<Integer> numbers = new HashSet<>();
```

This is generally preferred in application code because it reduces coupling and makes implementations easier to replace.

**Use a concrete implementation reference** when you specifically need functionality that is not exposed by the interface, or when working with implementation-specific behavior.

## 5. One-Line Interview Answer

**Programming to an interface means declaring a reference using an interface such as `List` while creating an object using a concrete class such as `ArrayList`, reducing coupling and allowing the implementation to be changed without changing code that depends only on the interface.**


---

# ArrayList vs LinkedList: Which Is Better for Manipulation?

## 1. What Is the Difference?

Both `ArrayList` and `LinkedList` implement the `List` interface, but they use different internal data structures.

- **ArrayList:** Uses a resizable array.
- **LinkedList:** Uses a doubly linked list, where each node stores an element and references to its previous and next nodes.

The right choice depends on what kind of operations your application performs.

## 2. Comparison Table

| Operation or Feature | ArrayList | LinkedList |
|---|---|---|
| Internal structure | Resizable array | Doubly linked list |
| Access by index | O(1) | O(n) |
| Search by value | O(n) | O(n) |
| Add at end | Amortized O(1) | O(1) |
| Insert at beginning | O(n) | O(1) |
| Remove from beginning | O(n) | O(1) |
| Insert or remove at a known position | Usually O(n) due to shifting | O(1) after reaching the node |
| Memory overhead | Generally lower | Generally higher due to node references |
| Cache locality | Usually better | Usually worse |
| Implements `Deque` | No | Yes |

**Important:** Insertion or deletion in a `LinkedList` is O(1) only when the relevant node or position is already available. Finding an arbitrary index or searching for an element still takes O(n).

## 3. Example: ArrayList vs LinkedList

```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class ListComparison {
    public static void main(String[] args) {
        List<Integer> arrayList = new ArrayList<>();
        List<Integer> linkedList = new LinkedList<>();

        arrayList.add(10);
        arrayList.add(20);
        arrayList.add(30);

        linkedList.add(10);
        linkedList.add(20);
        linkedList.add(30);

        // Insert at the beginning
        arrayList.add(0, 5);
        linkedList.add(0, 5);

        // Remove an element
        arrayList.remove(Integer.valueOf(20));
        linkedList.remove(Integer.valueOf(20));

        System.out.println(arrayList);
        System.out.println(linkedList);
    }
}
```

Output:

```text
[5, 10, 30]
[5, 10, 30]
```

Both collections produce the same result. Their internal operations and performance characteristics differ.

## 4. Why Is ArrayList Usually Preferred?

Even though `LinkedList` can insert or remove an element in O(1) time when the node is already known, `ArrayList` is usually the better default for general-purpose list manipulation.

Reasons:

1. **Fast random access:** `get(index)` is O(1).
2. **Better cache locality:** Elements are stored in a backing array, making traversal efficient.
3. **Lower memory overhead:** It does not require a separate linked node with previous and next references for every element.
4. **Efficient iteration:** Sequential traversal is generally fast.
5. **Good append performance:** Adding elements at the end is amortized O(1).

For example, if an application frequently reads and updates records by index, `ArrayList` is usually a better choice.

## 5. When Should You Use LinkedList?

Use `LinkedList` when its deque operations fit the problem, or when you already have a node through an iterator and need to insert or remove near that position.

```java
import java.util.LinkedList;

public class LinkedListExample {
    public static void main(String[] args) {
        LinkedList<String> tasks = new LinkedList<>();

        tasks.addFirst("Urgent task");
        tasks.addLast("Normal task");

        System.out.println(tasks.removeFirst());
        System.out.println(tasks);
    }
}
```

Output:

```text
Urgent task
[Normal task]
```

However, for ordinary queue and stack operations, `ArrayDeque` is often a better starting choice because it avoids the per-node allocation overhead of `LinkedList`.

## 6. Interview Scenario: Which Collection Will You Use for Manipulation?

The answer depends on what “manipulation” means:

- Frequent indexed access or traversal: **ArrayList**
- Frequent appending at the end: **ArrayList** is usually preferred.
- Frequent insertion or deletion at arbitrary indices: Neither is automatically best; consider how positions are found and how much data must shift.
- Insertion and removal at both ends: **ArrayDeque** is often a good choice.
- Insertion or removal around an iterator's current position: `LinkedList` can be useful for specific workloads.

Avoid saying that `LinkedList` is always better for insertion and deletion. Its O(1) advantage applies after reaching the relevant node; locating an arbitrary position can itself take O(n).

## 7. One-Line Interview Answer

**ArrayList is generally preferred for list manipulation because it offers O(1) indexed access, efficient traversal, and lower memory overhead, while LinkedList is useful when operations at the ends or around an already-located node are important.**


---

# Iterator in Java: Uses and Examples

## 1. What Is an Iterator?

An **Iterator** is an interface in `java.util` used to traverse elements in a collection one at a time without exposing the collection's internal implementation.

It is commonly used with collections such as `ArrayList`, `LinkedList`, and `HashSet`.

The `Iterator` interface provides three main methods:

| Method | Purpose |
|---|---|
| `hasNext()` | Returns `true` if another element is available |
| `next()` | Returns the next element; throws `NoSuchElementException` if none remains |
| `remove()` | Removes the last element returned by `next()`, if supported |

## 2. Example: Using an Iterator

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorExample {
    public static void main(String[] args) {
        List<String> fruits = new ArrayList<>();

        fruits.add("Apple");
        fruits.add("Banana");
        fruits.add("Cherry");

        Iterator<String> iterator = fruits.iterator();

        while (iterator.hasNext()) {
            String fruit = iterator.next();
            System.out.println(fruit);
        }
    }
}
```

Output:

```text
Apple
Banana
Cherry
```

### How it works

1. `fruits.iterator()` obtains an iterator for the list.
2. `hasNext()` checks whether another element is available.
3. `next()` retrieves the next element and advances the iterator.
4. The loop ends when no elements remain.

## 3. Why Do We Use an Iterator?

### A. Traverse collections without knowing their implementation

The same traversal pattern works with different collection implementations.

```java
List<String> names = new ArrayList<>();
Iterator<String> iterator = names.iterator();
```

You can change the list implementation without changing the basic iterator loop.

### B. Safely remove elements during iteration

Use `iterator.remove()` when you need to remove elements during traversal.

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class RemoveUsingIterator {
    public static void main(String[] args) {
        List<Integer> numbers =
                new ArrayList<>(List.of(10, 20, 30, 40));

        Iterator<Integer> iterator = numbers.iterator();

        while (iterator.hasNext()) {
            Integer number = iterator.next();

            if (number > 20) {
                iterator.remove();
            }
        }

        System.out.println(numbers);
    }
}
```

Output:

```text
[10, 20]
```

**Important:** Calling `list.remove()` directly during a normal `ArrayList` iterator traversal can cause `ConcurrentModificationException`. Using `iterator.remove()` is the supported approach for removing the current element, provided the iterator supports removal.

### C. Work with collections that do not support index-based access

A `HashSet` does not provide indexed access like `get(index)`. An iterator allows you to traverse its elements without requiring indices.

```java
Set<String> cities = new HashSet<>();
cities.add("Delhi");
cities.add("Patna");

Iterator<String> iterator = cities.iterator();

while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
```

The iteration order of a `HashSet` is not guaranteed.

## 4. Iterator vs Enhanced For Loop

| Feature | Iterator | Enhanced `for` loop |
|---|---|---|
| Syntax | More explicit | Simpler |
| Traversal | Yes | Yes |
| Remove current element during traversal | `iterator.remove()` when supported | No direct removal operation |
| Access to iterator methods | Yes | No |
| Best use | Controlled traversal and removal | Simple traversal |

Example enhanced `for` loop:

```java
for (String fruit : fruits) {
    System.out.println(fruit);
}
```

Use an enhanced `for` loop for straightforward traversal. Use an explicit iterator when you need iterator-specific operations.

## 5. Iterator vs ListIterator

`ListIterator` extends `Iterator` and is available for lists.

| Feature | Iterator | ListIterator |
|---|---|---|
| Forward traversal | Yes | Yes |
| Backward traversal | No | Yes |
| Add elements during traversal | No | Yes, using `add()` |
| Replace the last returned element | No | Yes, using `set()` |
| Applicable to general collections | Yes | No, list-specific |

## 6. Important Interview Points

- `Iterator` belongs to `java.util`.
- `next()` advances the iterator; `hasNext()` does not.
- `remove()` removes the last element returned by `next()` and can be called only once per successful `next()` before another `next()` call.
- Removal is optional and depends on the iterator implementation.
- Many standard collection iterators are fail-fast on structural modifications made outside the iterator, but this is best-effort behavior, not a guarantee for all collections.
- Concurrent collections may provide weakly consistent or snapshot iterators instead.

## 7. One-Line Interview Answer

**An Iterator in Java provides a standard way to traverse a collection element by element and, when supported, remove elements safely during iteration without depending on the collection's internal implementation.**


---

# HashMap Default Capacity, Load Factor, and Resizing

## 1. What Is the Default Capacity of HashMap?

The default initial capacity of a Java `HashMap` is **16**, and its default load factor is **0.75**.

```java
Map<Integer, String> map = new HashMap<>();
```

In modern Java implementations, creating a `HashMap` this way does not necessarily allocate the internal bucket array immediately. The table is generally allocated when the first entry is inserted.

Important terms:

- **Capacity:** The number of buckets in the internal hash table.
- **Load factor:** Determines how full the table can become before resizing is triggered.
- **Threshold:** The approximate number of entries that triggers a resize, calculated as capacity multiplied by load factor.

For the default settings:

\[
\text{Threshold} = 16 \times 0.75 = 12
\]

When the table capacity is 16, inserting the 13th entry normally triggers a resize in standard modern JDK implementations.

## 2. What Happens When HashMap Reaches Its Threshold?

When the number of entries exceeds the threshold, `HashMap` generally increases its table capacity to twice the previous capacity.

For example:

| Capacity | Load factor | Threshold |
|---:|---:|---:|
| 16 | 0.75 | 12 |
| 32 | 0.75 | 24 |
| 64 | 0.75 | 48 |
| 128 | 0.75 | 96 |

The sequence continues as the map grows, subject to implementation limits.

### Example

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapCapacityExample {
    public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();

        for (int i = 1; i <= 13; i++) {
            map.put(i, "Value" + i);
        }

        System.out.println(map.size());
    }
}
```

Output:

```text
13
```

The map contains 13 entries. With the default settings in a modern standard JDK, adding the 13th entry triggers growth from 16 to 32 buckets.

Note that `HashMap` does not provide a public method to retrieve its internal capacity directly; the capacity sequence above describes its typical implementation behavior.

## 3. How Does HashMap Resize Internally?

When resizing is triggered, the `HashMap` typically performs these operations:

1. Allocates a larger bucket array, usually twice the previous capacity.
2. Redistributes existing entries into the new table.
3. Updates the table reference and threshold.

This process is commonly called **resizing** or **rehashing**.

An important implementation detail: modern Java `HashMap` implementations can often redistribute entries using stored hash values rather than calling each key's `hashCode()` method again. Therefore, it is inaccurate to say that every resize necessarily recalculates all key hash codes.

### What about hash collisions?

A collision occurs when multiple keys map to the same bucket. Resizing can redistribute entries across more buckets, potentially reducing collisions, but it does not guarantee that all collisions disappear.

In modern Java, a heavily populated bucket can be converted from a linked structure into a tree under suitable conditions, helping avoid the worst-case behavior of long collision chains.

## 4. What Is the Maximum Capacity of HashMap?

In common OpenJDK implementations, the maximum table capacity is:

\[
2^{30} = 1,073,741,824
\]

This is an implementation limit, not a promise that the JVM can allocate such a large table.

When the table reaches its maximum capacity, it cannot continue doubling. The threshold may be set to the maximum integer value, and further insertions can continue subject to available memory and other runtime constraints. A large map can still encounter performance and memory problems.

## 5. Why Does Load Factor Matter?

A lower load factor tends to cause earlier resizing, potentially reducing collisions but using more memory.

A higher load factor delays resizing, potentially saving memory but increasing collisions and slowing some operations.

The default load factor of `0.75` is a practical compromise for general-purpose use.

You can specify capacity and load factor explicitly:

```java
Map<Integer, String> map = new HashMap<>(100, 0.75f);
```

The requested initial capacity is a sizing hint. The actual internal table capacity may be rounded to a supported power of two, and allocation is typically deferred until the first insertion.

If you know approximately how many entries you need, choose an appropriate initial capacity to reduce repeated resizing.

## 6. One-Line Interview Answer

**Java HashMap has a default initial capacity of 16 and a default load factor of 0.75, giving a threshold of 12 entries; when the threshold is exceeded, it typically doubles its bucket array and redistributes entries to maintain efficient lookup performance.**


---


# Using Custom Objects as HashMap Keys in Java

## 1. How Do You Create a Custom Object as a HashMap Key?

You can use any object as a `HashMap` key, including objects of your own classes. However, to ensure that logically equivalent objects behave as the same key, you should correctly override `equals()` and `hashCode()`.

- **`hashCode()`** helps determine which bucket to search.
- **`equals()`** determines whether two keys are logically equal.

The contract requires that if two objects are equal according to `equals()`, they must return the same hash code.

## 2. Complete Example

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Person)) {
            return false;
        }

        Person other = (Person) obj;

        return age == other.age
                && Objects.equals(name, other.name);
    }
}

public class CustomKeyExample {
    public static void main(String[] args) {
        Map<Person, String> people = new HashMap<>();

        Person john = new Person("John", 30);
        Person sarah = new Person("Sarah", 25);

        people.put(john, "12345678");
        people.put(sarah, "2345678");

        String phone = people.get(new Person("John", 30));

        System.out.println(phone);
    }
}
```

Output:

```text
12345678
```

Although `new Person("John", 30)` creates a different object from the original `john`, both objects are logically equal because their `name` and `age` fields match. Therefore, the lookup succeeds.

## 3. How Does HashMap Use `hashCode()` and `equals()`?

When executing:

```java
people.get(new Person("John", 30));
```

The process is conceptually:

1. **Calculate the hash:** `HashMap` obtains a hash based on the lookup key's `hashCode()`.
2. **Find the bucket:** The hash is used to identify the bucket to search.
3. **Compare keys:** The map checks candidate keys using hash information and equality comparisons.
4. **Return the value:** When an equal key is found, its associated value is returned.

If two unequal objects have the same hash code, that is a **hash collision**. `HashMap` handles collisions by comparing candidate keys for equality.

## 4. What Happens If You Override Only One Method?

| Implementation                                      | Result                                                              |
| --------------------------------------------------- | ------------------------------------------------------------------- |
| Override both `equals()` and `hashCode()` correctly | Logically equal objects work as equivalent keys                     |
| Override only `equals()`                            | Equal objects can have different hash codes, so lookups may fail    |
| Override only `hashCode()`                          | Different objects with the same hash code are not necessarily equal |
| Override neither                                    | Default `Object` behavior generally compares object identity        |

**Interview rule:** Always implement `equals()` and `hashCode()` consistently when custom objects are intended to have value-based key equality.

## 5. Why Should HashMap Keys Be Immutable?

If a key's fields used by `equals()` or `hashCode()` change after insertion, the map may no longer find the entry using that key.

For example, if `Person` had mutable `name` and `age` fields and those fields changed after `put()`, a subsequent lookup could calculate a different hash and search a different bucket.

In the example, `name` and `age` are `final`, and no setters are provided. This prevents those fields from changing after construction.

## 6. One-Line Interview Answer

**To use a custom object reliably as a HashMap key, override `equals()` and `hashCode()` consistently using the fields that define logical equality, and avoid modifying those fields after inserting the object into the map.**


---

# Does HashMap Maintain Insertion Order in Java?

## 1. Does HashMap Store Elements in an Ordered Way?

**No, `HashMap` does not guarantee any specific order** for its key-value pairs.

The order in which entries are stored or returned during iteration is not guaranteed to match their insertion order, key order, or value order.

`HashMap` uses the hash codes of keys to help determine where entries belong in its internal structure. Its iteration order can change when entries are added, removed, or the map is resized.

## 2. Example: HashMap Does Not Guarantee Insertion Order

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapOrderExample {
    public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();

        map.put(3, "Three");
        map.put(1, "One");
        map.put(2, "Two");

        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            System.out.println(
                entry.getKey() + " = " + entry.getValue()
            );
        }
    }
}
```

The output order is **not guaranteed**. It might appear sorted or match insertion order for some keys, but you must not rely on that behavior.

## 3. Which Map Should You Use for Ordering?

| Feature | HashMap | LinkedHashMap | TreeMap |
|---|---|---|---|
| Maintains insertion order | No | Yes, by default | No |
| Sorts entries by key | No | No | Yes, by natural order or a comparator |
| Average/basic lookup performance | O(1) | O(1) | O(log n) |
| Allows one null key | Yes | Yes | Usually no with natural ordering |
| Best use case | General key-value storage | Predictable iteration order | Sorted keys |

## 4. Example: LinkedHashMap Maintains Insertion Order

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LinkedHashMapOrderExample {
    public static void main(String[] args) {
        Map<Integer, String> map = new LinkedHashMap<>();

        map.put(3, "Three");
        map.put(1, "One");
        map.put(2, "Two");

        map.forEach((key, value) ->
            System.out.println(key + " = " + value)
        );
    }
}
```

Output:

```text
3 = Three
1 = One
2 = Two
```

`LinkedHashMap` preserves the order in which distinct keys were first inserted by default. Updating an existing key's value does not move its entry to the end in insertion-order mode.

## 5. Example: TreeMap Sorts Entries by Key

```java
import java.util.Map;
import java.util.TreeMap;

public class TreeMapOrderExample {
    public static void main(String[] args) {
        Map<Integer, String> map = new TreeMap<>();

        map.put(3, "Three");
        map.put(1, "One");
        map.put(2, "Two");

        map.forEach((key, value) ->
            System.out.println(key + " = " + value)
        );
    }
}
```

Output:

```text
1 = One
2 = Two
3 = Three
```

`TreeMap` maintains its entries sorted by key, using natural ordering or a supplied comparator. With natural ordering, a null key is not permitted.

## 6. One-Line Interview Answer

**HashMap does not guarantee iteration order; use LinkedHashMap to preserve insertion order or TreeMap to sort entries by key.**


---

# How Does LinkedHashMap Maintain Insertion Order in Java?

## 1. How Does LinkedHashMap Maintain Insertion Order?

`LinkedHashMap` maintains insertion order by combining two data structures:

1. **Hash table:** Provides efficient key-based lookup, insertion, and removal.
2. **Doubly linked list:** Connects entries in iteration order.

Unlike `HashMap`, which does not guarantee iteration order, `LinkedHashMap` links its entries so that iteration follows their insertion order by default.

When a new key is inserted, its entry is linked at the end of the doubly linked list. Iteration follows these links rather than relying on the hash table's bucket order.

## 2. Internal Structure

Conceptually, a `LinkedHashMap` maintains both a hash table and a linked list:

```text
Hash table:
    Bucket 0 ──> Entry C
    Bucket 1 ──> Entry A
    Bucket 2 ──> Entry B

Doubly linked list (insertion order):

    head
      |
      v
    Entry A <──> Entry B <──> Entry C
                                  |
                                  v
                                 tail
```

The hash table determines where entries can be found efficiently. The linked list determines the order in which entries are visited during iteration.

The bucket arrangement above is illustrative; actual bucket placement depends on hashing.

## 3. Example: LinkedHashMap Maintains Insertion Order

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LinkedHashMapExample {
    public static void main(String[] args) {
        Map<Integer, String> map = new LinkedHashMap<>();

        map.put(3, "Three");
        map.put(1, "One");
        map.put(2, "Two");

        map.forEach((key, value) ->
            System.out.println(key + " = " + value)
        );
    }
}
```

Output:

```text
3 = Three
1 = One
2 = Two
```

Although the keys were inserted in the order `3, 1, 2`, iteration follows that same order rather than sorting the keys.

## 4. What Happens When an Existing Key Is Updated?

Updating the value of an existing key does not move its entry in the default insertion-order mode.

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class UpdateExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new LinkedHashMap<>();

        map.put("A", 1);
        map.put("B", 2);
        map.put("C", 3);

        map.put("B", 20);

        System.out.println(map);
    }
}
```

Output:

```text
{A=1, B=20, C=3}
```

The value associated with `"B"` changes from `2` to `20`, but its position remains unchanged.

## 5. Can LinkedHashMap Maintain Access Order Instead?

Yes. `LinkedHashMap` can also be configured to maintain **access order**, which moves an accessed entry toward the end of the iteration order.

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class AccessOrderExample {
    public static void main(String[] args) {
        Map<String, Integer> map =
            new LinkedHashMap<>(16, 0.75f, true);

        map.put("A", 1);
        map.put("B", 2);
        map.put("C", 3);

        map.get("A");

        System.out.println(map);
    }
}
```

Output:

```text
{B=2, C=3, A=1}
```

The third constructor argument, `true`, enables access-order mode. This behavior is useful when implementing an LRU (Least Recently Used) cache.

## 6. Time Complexity

| Operation | Typical performance |
|---|---|
| `put()` | O(1) average |
| `get()` | O(1) average |
| `remove()` | O(1) average |
| Iteration over entries | O(n) |

These are typical expected complexities, assuming reasonable hash distribution.

## 7. One-Line Interview Answer

**LinkedHashMap maintains insertion order by combining a hash table for efficient key-based operations with a doubly linked list that connects entries in iteration order.**


---










