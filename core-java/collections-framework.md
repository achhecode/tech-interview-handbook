---
title: Collections Framework
parent: Core Java
nav_order: 3
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

## How does HashMap work internally?

Hard
{: .label .label-red }

**Answer**

1. `hashCode()` of the key is spread: `h ^ (h >>> 16)`.
2. The bucket index is `(n - 1) & hash`, where `n` is the table size (default 16).
3. Collisions form a linked list in the bucket; in Java 8+ a bin becomes a red-black tree when it has 8 or more nodes and the table has at least 64 buckets, otherwise the table is resized.
4. When size exceeds `capacity * 0.75` (load factor), the table doubles and entries are redistributed.

`HashMap` is not thread-safe. Use `ConcurrentHashMap` (CAS plus per-bin locking) for concurrent access.

**Follow-up questions**
- HashMap vs Hashtable vs ConcurrentHashMap?
- What happens if a mutable key is changed after insertion?

---

## ArrayList vs LinkedList: when would you use each?

Easy
{: .label .label-green }

**Answer**

- `ArrayList`: backed by an array, O(1) random access, amortised O(1) append, O(n) insert in the middle.
- `LinkedList`: O(1) insert or remove at a known node, O(n) access by index, higher memory per element.

In practice `ArrayList` wins in most cases because of CPU cache locality.

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
