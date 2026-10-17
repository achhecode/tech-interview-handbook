---
title: Language Fundamentals
parent: Core Java
nav_order: 2
---

# Language Fundamentals
{: .no_toc }

Strings, equals/hashCode, immutability and other basics.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the equals() and hashCode() contract?

Medium
{: .label .label-yellow }

**Answer**

- If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must be true.
- Equal hash codes do **not** imply equal objects (collisions).
- Both must be consistent across calls while fields used in them do not change.

If you override `equals` but not `hashCode`, hash-based collections (`HashMap`, `HashSet`) can fail to find the object.

```java
@Override public boolean equals(Object o) {
    return o instanceof Employee e && id == e.id;
}
@Override public int hashCode() { return Integer.hashCode(id); }
```

---

## Why is String immutable in Java?

Medium
{: .label .label-yellow }

**Answer**

- **String pool:** literals can be shared safely.
- **Thread safety:** immutable objects need no synchronization.
- **Cached hash code:** makes `String` a fast `HashMap` key.
- **Security:** values such as class names and file paths cannot be changed after validation.

Use `StringBuilder` for single-threaded concatenation in loops, and `StringBuffer` only when you need synchronized methods.

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
