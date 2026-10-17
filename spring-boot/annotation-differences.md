---
title: Annotation Differences
parent: Spring Boot
nav_order: 3
---

# Annotation Differences
{: .no_toc }

Frequently asked "X vs Y" annotation questions.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## @Component vs @Service vs @Repository vs @Controller?

Easy
{: .label .label-green }

**Answer**

All are stereotypes registered by component scanning. `@Component` is generic. `@Service` marks business logic (no extra behaviour). `@Repository` marks the data layer and enables persistence exception translation. `@Controller` marks a web controller.

---

## @Controller vs @RestController?

Easy
{: .label .label-green }

**Answer**

`@RestController` = `@Controller` + `@ResponseBody`. Return values are written to the response body (JSON) instead of being resolved as view names.

---

## @Bean vs @Component?

Medium
{: .label .label-yellow }

**Answer**

`@Component` is placed on **your own class** and detected by scanning. `@Bean` is placed on a **method** inside a `@Configuration` class and is used when you need to create third-party objects or control the instantiation logic.

---

## @RequestParam vs @PathVariable?

Easy
{: .label .label-green }

**Answer**

`@PathVariable` reads from the URL path (`/users/{id}`). `@RequestParam` reads query parameters (`/users?role=admin`).

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
