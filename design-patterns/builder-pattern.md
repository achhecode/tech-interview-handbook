---
title: Builder
parent: Design Patterns and LLD
nav_order: 3
---

# Builder
{: .no_toc }

Constructing complex immutable objects.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the Builder pattern and where is it used?

Easy
{: .label .label-green }

**Answer**

It separates construction of an object from its representation, avoiding telescoping constructors. Examples: `StringBuilder`, `Stream.Builder`, Lombok `@Builder`.

```java
public class User {
    private final String name; private final String email;
    private User(Builder b) { this.name = b.name; this.email = b.email; }
    public static Builder builder() { return new Builder(); }
    public static class Builder {
        private String name, email;
        public Builder name(String n) { this.name = n; return this; }
        public Builder email(String e) { this.email = e; return this; }
        public User build() { return new User(this); }
    }
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
