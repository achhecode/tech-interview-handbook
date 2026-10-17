---
title: Exception Handling
parent: Core Java
nav_order: 4
---

# Exception Handling
{: .no_toc }

Checked vs unchecked exceptions and resource handling.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the difference between checked and unchecked exceptions?

Easy
{: .label .label-green }

**Answer**

- **Checked** (extend `Exception`, not `RuntimeException`): the compiler forces you to catch or declare them, e.g. `IOException`, `SQLException`.
- **Unchecked** (extend `RuntimeException`) and `Error`: not enforced, e.g. `NullPointerException`, `IllegalArgumentException`.

Use checked exceptions for recoverable conditions the caller must handle, and unchecked for programming errors.

---

## How does try-with-resources work?

Medium
{: .label .label-yellow }

**Answer**

Any resource implementing `AutoCloseable` declared in the `try (...)` header is closed automatically in reverse order, even on exceptions. If both the body and `close()` throw, the `close()` exception is attached as a **suppressed** exception.

```java
try (var in = Files.newBufferedReader(path)) {
    return in.readLine();
}
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
