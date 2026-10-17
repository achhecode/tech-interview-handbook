---
title: Testing
parent: Spring Boot
nav_order: 8
---

# Testing
{: .no_toc }

Slice tests, mocking and integration tests.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## @SpringBootTest vs @WebMvcTest vs @DataJpaTest?

Medium
{: .label .label-yellow }

**Answer**

- `@SpringBootTest`: loads the full application context (integration test, slowest).
- `@WebMvcTest`: only the web layer; mock services with `@MockitoBean` (`@MockBean` before Boot 3.4).
- `@DataJpaTest`: only JPA components with an embedded database, rolled back after each test.

Use plain unit tests with Mockito (`@Mock`, `@InjectMocks`) for business logic that does not need Spring.

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
