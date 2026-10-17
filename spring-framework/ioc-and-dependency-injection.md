---
title: IoC and Dependency Injection
parent: Spring Framework
nav_order: 1
---

# IoC and Dependency Injection
{: .no_toc }

Inversion of Control and the ways Spring injects dependencies.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the difference between IoC and DI?

Easy
{: .label .label-green }

**Answer**

**IoC** is the principle: the framework controls object creation and wiring instead of your code. **DI** is the main technique that implements it: dependencies are provided to a class from outside.

```java
@Service
class OrderService {
    private final PaymentClient client;
    OrderService(PaymentClient client) { this.client = client; }  // constructor injection
}
```

---

## Constructor vs setter vs field injection: which is best?

Medium
{: .label .label-yellow }

**Answer**

**Constructor injection** is preferred: dependencies are mandatory and can be `final`, objects are never half-built, tests need no Spring, and cyclic dependencies fail fast. Use setter injection for optional dependencies. Avoid field injection (`@Autowired` on fields) because it hides dependencies and complicates testing.

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
