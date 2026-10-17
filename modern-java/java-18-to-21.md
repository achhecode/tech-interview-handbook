---
title: Java 18 to 21
parent: Modern Java (8 to 25)
nav_order: 4
---

# Java 18 to 21
{: .no_toc }

Virtual threads and pattern matching, with the Java 21 LTS.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What are virtual threads and when should you use them?

Hard
{: .label .label-red }

**Answer**

Virtual threads (final in Java 21) are lightweight threads managed by the JVM and scheduled onto a small pool of carrier (platform) threads. Blocking I/O parks the virtual thread instead of blocking an OS thread.

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> callRemoteService());
}
```

Use them for high-concurrency, I/O-bound workloads. They do not speed up CPU-bound work. Avoid pooling them, and be careful with long `synchronized` blocks that can pin the carrier thread.

---

## What else is new in Java 21?

Medium
{: .label .label-yellow }

**Answer**

Record patterns and pattern matching for `switch` (both final), sequenced collections (`getFirst()`, `reversed()`), and generational ZGC.

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
