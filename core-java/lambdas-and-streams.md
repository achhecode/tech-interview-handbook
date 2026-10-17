---
title: Lambdas and Streams
parent: Core Java
nav_order: 7
---

# Lambdas and Streams
{: .no_toc }

Functional interfaces, Stream API and Optional.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the difference between map() and flatMap()?

Medium
{: .label .label-yellow }

**Answer**

`map` transforms each element one-to-one. `flatMap` transforms each element into a stream and flattens the results.

```java
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3, 4));
List<Integer> flat = nested.stream()
        .flatMap(List::stream)
        .toList();                 // [1, 2, 3, 4]
```

---

## Name the core functional interfaces.

Easy
{: .label .label-green }

**Answer**

| Interface | Method | Use |
|:--|:--|:--|
| `Predicate<T>` | `boolean test(T)` | filter conditions |
| `Function<T,R>` | `R apply(T)` | transform |
| `Consumer<T>` | `void accept(T)` | side effects |
| `Supplier<T>` | `T get()` | lazy creation |

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
