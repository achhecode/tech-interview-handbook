---
title: Multithreading Programs
parent: Coding Problems
nav_order: 4
---

# Multithreading Programs
{: .no_toc }

Concurrency coding questions.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Implement a producer-consumer using BlockingQueue.

Medium
{: .label .label-yellow }

**Answer**

```java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);

Thread producer = new Thread(() -> {
    try { for (int i = 1; i <= 5; i++) queue.put(i); queue.put(-1); }   // -1 = stop signal
    catch (InterruptedException e) { Thread.currentThread().interrupt(); }
});
Thread consumer = new Thread(() -> {
    try { int n; while ((n = queue.take()) != -1) System.out.println("Got " + n); }
    catch (InterruptedException e) { Thread.currentThread().interrupt(); }
});
producer.start(); consumer.start();
```
`put` blocks when full and `take` blocks when empty, so no manual wait/notify is needed.

---

## Write a deadlock example and explain how to avoid it.

Hard
{: .label .label-red }

**Answer**

```java
Object a = new Object(), b = new Object();
new Thread(() -> { synchronized (a) { sleep(); synchronized (b) { } } }).start();
new Thread(() -> { synchronized (b) { sleep(); synchronized (a) { } } }).start();
```
Deadlock needs mutual exclusion, hold-and-wait, no preemption and circular wait. Avoid it with a **consistent lock ordering**, `tryLock` with a timeout, or by reducing lock scope.

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
