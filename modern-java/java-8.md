---
title: Java 8
parent: Modern Java (8 to 25)
nav_order: 1
---

# Java 8
{: .no_toc }

Lambdas, streams and the start of modern Java.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What are the major features introduced in Java 8?

Easy
{: .label .label-green }

**Answer**

- Lambda expressions and method references
- Functional interfaces (`java.util.function`)
- Stream API
- `Optional`
- Default and static methods in interfaces
- New Date and Time API (`java.time`)
- `CompletableFuture`

```java
List<String> names = employees.stream()
        .filter(e -> e.salary() > 50_000)
        .map(Employee::name)
        .sorted()
        .toList();   // toList() arrived in Java 16
```

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
