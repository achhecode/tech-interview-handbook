---
title: Factory and Abstract Factory
parent: Design Patterns and LLD
nav_order: 4
---

# Factory and Abstract Factory
{: .no_toc }

Hiding object creation logic.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the Factory pattern?

Easy
{: .label .label-green }

**Answer**

A factory creates objects without exposing the instantiation logic; callers depend on an interface.

```java
interface Notifier { void send(String msg); }
class NotifierFactory {
    static Notifier of(String type) {
        return switch (type) {
            case "EMAIL" -> new EmailNotifier();
            case "SMS"   -> new SmsNotifier();
            default -> throw new IllegalArgumentException(type);
        };
    }
}
```

**Abstract Factory** creates families of related objects. Spring's `BeanFactory` is a factory itself.

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
