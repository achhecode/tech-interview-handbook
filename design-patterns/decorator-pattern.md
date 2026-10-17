---
title: Decorator
parent: Design Patterns and LLD
nav_order: 7
---

# Decorator
{: .no_toc }

Adding behaviour dynamically.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the Decorator pattern and how does it differ from inheritance?

Medium
{: .label .label-yellow }

**Answer**

A decorator wraps an object that implements the same interface and adds behaviour at runtime, without subclass explosion. Example: `new BufferedReader(new InputStreamReader(System.in))`.

```java
interface Coffee { double cost(); }
class Basic implements Coffee { public double cost() { return 50; } }
class Milk implements Coffee {
    private final Coffee base;
    Milk(Coffee base) { this.base = base; }
    public double cost() { return base.cost() + 10; }
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
