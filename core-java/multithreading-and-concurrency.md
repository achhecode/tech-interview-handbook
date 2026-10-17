---
title: Multithreading and Concurrency
parent: Core Java
nav_order: 5
---

# Multithreading and Concurrency
{: .no_toc }

Threads, locks, executors and the Java Memory Model.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What are the ways to create a thread, and why prefer ExecutorService?

Medium
{: .label .label-yellow }

**Answer**

Options: extend `Thread`, implement `Runnable`, implement `Callable` (returns a result), or submit tasks to an `ExecutorService`.

Prefer executors because they reuse threads (pooling), bound resource usage, return `Future`s, and give lifecycle control.

```java
ExecutorService pool = Executors.newFixedThreadPool(4);
Future<Integer> f = pool.submit(() -> 21 * 2);
System.out.println(f.get());
pool.shutdown();
```

---

## What is the difference between volatile, synchronized and AtomicInteger?

Hard
{: .label .label-red }

**Answer**

| | Guarantees |
|:--|:--|
| `volatile` | Visibility and ordering of a variable; **no atomicity** for `count++` |
| `synchronized` | Mutual exclusion plus visibility |
| `AtomicInteger` | Lock-free atomic updates using CAS |

**Follow-up questions**
- What are the conditions for a deadlock and how do you prevent it?
- Virtual threads vs platform threads?

---

<!--
ADD NEW QUESTIONS ABOVE THIS COMMENT. Copy the block below, then replace the text.
Levels: Easy (label-green), Medium (label-yellow), Hard (label-red).

## Your question here?

Medium
{: .label .label-yellow }

**Answer**

Write the answer here. Use tables, lists and ```java code blocks.

**Key points**
- Point one

**Follow-up questions**
- Follow-up one

Separate questions with a horizontal rule (three dashes on their own line).
-->
