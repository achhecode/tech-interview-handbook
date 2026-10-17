---
title: Joins, Indexing and Performance
parent: SQL, JPA and Hibernate
nav_order: 2
---

# Joins, Indexing and Performance
{: .no_toc }

Join types, how indexes work and query tuning.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Explain the types of SQL joins.

Easy
{: .label .label-green }

**Answer**

- `INNER JOIN`: rows matching in both tables.
- `LEFT JOIN`: all rows from the left table plus matches (NULL when none).
- `RIGHT JOIN`: the mirror of left.
- `FULL JOIN`: all rows from both.
- `CROSS JOIN`: Cartesian product.

---

## How do indexes work and why might a query not use one?

Hard
{: .label .label-red }

**Answer**

Most indexes are B-trees that keep keys sorted, so lookups, range scans and ordering avoid a full table scan. They speed up reads but slow down writes and use space.

An index may be ignored when:
- a function wraps the column (`WHERE UPPER(name) = 'A'`),
- the condition uses a leading wildcard (`LIKE '%abc'`),
- the column order in a composite index does not match the filter (leftmost prefix rule),
- low selectivity makes a full scan cheaper.

Use `EXPLAIN` to confirm the execution plan.

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
