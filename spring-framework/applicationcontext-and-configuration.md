---
title: ApplicationContext and Configuration
parent: Spring Framework
nav_order: 3
---

# ApplicationContext and Configuration
{: .no_toc }

Containers and configuration styles.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the difference between BeanFactory and ApplicationContext?

Easy
{: .label .label-green }

**Answer**

`BeanFactory` is the basic container with lazy bean creation. `ApplicationContext` extends it and adds eager singleton initialization, event publishing, internationalization (`MessageSource`), resource loading and automatic `BeanPostProcessor` registration. Use `ApplicationContext` in practice.

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
