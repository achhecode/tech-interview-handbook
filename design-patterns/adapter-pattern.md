---
title: Adapter
parent: Design Patterns and LLD
nav_order: 5
---

# Adapter
{: .no_toc }

Making incompatible interfaces work together.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the Adapter pattern?

Easy
{: .label .label-green }

**Answer**

It converts one interface into another that clients expect, often to integrate legacy or third-party code.

```java
class LegacyPrinter { void printText(String t) { } }
interface Printer { void print(String text); }
class PrinterAdapter implements Printer {
    private final LegacyPrinter legacy = new LegacyPrinter();
    public void print(String text) { legacy.printText(text); }
}
```

Java examples: `Arrays.asList()`, `InputStreamReader`.

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
