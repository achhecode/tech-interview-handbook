---
title: Java 12 to 17
parent: Modern Java (8 to 25)
nav_order: 3
---

# Java 12 to 17
{: .no_toc }

Switch expressions, text blocks, records and sealed classes.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Explain records, sealed classes and switch expressions.

Medium
{: .label .label-yellow }

**Answer**

- **Records** (16): concise immutable data carriers.
- **Sealed classes** (17): restrict which classes can extend or implement a type.
- **Switch expressions** (14): switch returns a value, no fall-through.
- **Text blocks** (15) and **pattern matching for instanceof** (16).

```java
record Point(int x, int y) {}

sealed interface Shape permits Circle, Square {}
record Circle(double r) implements Shape {}
record Square(double side) implements Shape {}

double area(Shape s) {
    return switch (s) {
        case Circle c -> Math.PI * c.r() * c.r();
        case Square q -> q.side() * q.side();
    };
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
