---
title: Singleton
parent: Design Patterns and LLD
nav_order: 2
---

# Singleton
{: .no_toc }

One instance per JVM.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How do you implement a thread-safe Singleton?

Medium
{: .label .label-yellow }

**Answer**

```java
// Bill Pugh holder: lazy and thread-safe
public class Config {
    private Config() {}
    private static class Holder { static final Config INSTANCE = new Config(); }
    public static Config getInstance() { return Holder.INSTANCE; }
}

// Enum: safest against reflection and serialization
public enum AppState { INSTANCE }
```

Double-checked locking also works but the field must be `volatile`. In Spring, singleton scope is per container, not per JVM.

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
