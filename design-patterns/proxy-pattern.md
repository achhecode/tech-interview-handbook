---
title: Proxy
parent: Design Patterns and LLD
nav_order: 6
---

# Proxy
{: .no_toc }

Controlling access to an object.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the Proxy pattern and how is it used in Spring?

Medium
{: .label .label-yellow }

**Answer**

A proxy has the same interface as the real object and controls access (lazy loading, security, caching, logging). Spring uses JDK dynamic proxies or CGLIB proxies to implement `@Transactional`, `@Cacheable`, `@Async` and AOP. Hibernate uses proxies for lazy-loaded entities.

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
