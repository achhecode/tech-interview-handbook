---
title: JPA and Hibernate
parent: SQL, JPA and Hibernate
nav_order: 4
---

# JPA and Hibernate
{: .no_toc }

How JPA, Hibernate and Spring Data fit together, plus common problems.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How do JPA, Hibernate and Spring Data JPA relate?

Medium
{: .label .label-yellow }

**Answer**

```text
Your code -> Spring Data JPA (repositories)
          -> JPA API (EntityManager, annotations)  <- specification
          -> Hibernate (JPA provider, implementation)
          -> JDBC -> Database
```

JPA is a specification, Hibernate is the most common implementation, and Spring Data JPA removes boilerplate by generating repository implementations on top of JPA.

---

## What is the N+1 problem and how do you fix it?

Hard
{: .label .label-red }

**Answer**

Loading N parent entities triggers one extra query per parent to load a lazy association, giving 1 + N queries.

**Fixes**
- `JOIN FETCH` in JPQL
- `@EntityGraph` on the repository method
- `@BatchSize` or `hibernate.default_batch_fetch_size`
- DTO projections for read-only views

```java
@Query("select o from Order o join fetch o.items where o.status = :status")
List<Order> findWithItems(@Param("status") String status);
```

---

## Lazy vs eager loading, and what is LazyInitializationException?

Medium
{: .label .label-yellow }

**Answer**

Lazy loads an association on first access; eager loads it immediately. Accessing a lazy association after the session or transaction is closed throws `LazyInitializationException`. Fetch what you need inside the transaction (join fetch, entity graph) or return DTOs. Do not "fix" it by making everything eager.

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
