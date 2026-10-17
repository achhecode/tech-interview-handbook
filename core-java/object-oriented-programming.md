---
title: Object-Oriented Programming
parent: Core Java
nav_order: 1
---

# Object-Oriented Programming
{: .no_toc }

Pillars of OOP, abstract classes, interfaces and design basics.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What are the four pillars of OOP? Explain with examples.

Easy
{: .label .label-green }

**Answer**

- **Encapsulation:** hide state behind private fields and expose behaviour through methods.
- **Abstraction:** expose what an object does, hide how (interfaces, abstract classes).
- **Inheritance:** a class reuses behaviour of a parent (`extends`). Prefer composition when the relation is not truly "is-a".
- **Polymorphism:** the same call behaves differently by type. Compile-time = overloading, runtime = overriding.

```java
interface Payment { void pay(double amount); }
class UpiPayment implements Payment {
    public void pay(double amount) { System.out.println("UPI " + amount); }
}
Payment p = new UpiPayment();
p.pay(100);   // runtime polymorphism: resolved by actual object type
```

**Follow-up questions**
- Method overloading vs overriding?
- Can you override a static or private method?

---

## What is the difference between an abstract class and an interface?

Medium
{: .label .label-yellow }

**Answer**

| | Abstract class | Interface |
|:--|:--|:--|
| State | Instance fields allowed | Only constants (`public static final`) |
| Constructors | Yes | No |
| Inheritance | Single | Multiple |
| Methods | Abstract and concrete | Abstract, `default`, `static`, `private` (Java 9+) |

Use an **interface** for a capability contract; use an **abstract class** to share state or partial implementation.

**Follow-up questions**
- How does Java resolve the diamond problem with default methods?

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
