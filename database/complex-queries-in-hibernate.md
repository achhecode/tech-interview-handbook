---
title: Complex Queries in Hibernate
parent: SQL, JPA and Hibernate
nav_order: 5
---

# Complex Queries in Hibernate
{: .no_toc }

Joins, projections and dynamic queries in JPA.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How do you write complex joins and aggregations in JPA/Hibernate?

Hard
{: .label .label-red }

**Answer**

| Option | Use when |
|:--|:--|
| JPQL / HQL with `join` | Most queries over entities |
| DTO projection (constructor expression) | Read-only reports |
| Criteria API / Specifications | Dynamic filters |
| Native query | DB-specific features, window functions |
| Querydsl or jOOQ | Type-safe complex queries |

```java
@Query("""
    select new com.example.dto.OrderSummary(c.name, count(o), sum(o.total))
    from Customer c join c.orders o
    where o.status = :status
    group by c.name
    having sum(o.total) > :min
""")
List<OrderSummary> summarize(@Param("status") String status, @Param("min") BigDecimal min);
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
