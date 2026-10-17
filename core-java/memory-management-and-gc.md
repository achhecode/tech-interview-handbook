---
title: Memory Management and Garbage Collection
parent: Core Java
nav_order: 8
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

## Explain JVM memory areas and how garbage collection works.

Medium
{: .label .label-yellow }

**Answer**

- **Heap:** objects. Split into Young (Eden, Survivor S0/S1) and Old generation.
- **Metaspace:** class metadata (replaced PermGen in Java 8), uses native memory.
- **Thread stacks:** frames and local variables per thread.
- **PC register and native stacks:** per-thread execution state.

GC is based on the generational hypothesis: most objects die young. New objects go to Eden, survivors are copied between survivor spaces, and long-lived ones are promoted to Old. G1 is the default collector since Java 9; ZGC and Shenandoah target very low pause times.

---

## How do you diagnose a memory leak in production?

Hard
{: .label .label-red }

**Answer**

1. Watch heap trend and GC logs (heap keeps growing after full GCs).
2. Enable `-XX:+HeapDumpOnOutOfMemoryError` or take a dump with `jcmd` / `jmap`.
3. Analyse the dump in Eclipse MAT or VisualVM and find the dominator objects.

Common causes: static collections that only grow, unclosed resources, `ThreadLocal` not removed, and unremoved listeners.

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
