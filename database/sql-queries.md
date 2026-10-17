---
title: SQL Queries
parent: SQL, JPA and Hibernate
nav_order: 1
---

# SQL Queries
{: .no_toc }

The classic SQL interview queries.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How do you find the maximum and the Nth highest salary?

Easy
{: .label .label-green }

**Answer**

```sql
-- Highest
SELECT MAX(salary) FROM employee;

-- Second highest
SELECT MAX(salary) FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);

-- Nth highest (works in most databases)
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employee
) t
WHERE rnk = 3;
```

---

## How do you find and remove duplicate rows?

Medium
{: .label .label-yellow }

**Answer**

```sql
-- Find duplicates
SELECT email, COUNT(*) FROM employee GROUP BY email HAVING COUNT(*) > 1;

-- Delete duplicates, keep the lowest id
DELETE FROM employee WHERE id IN (
    SELECT id FROM (
        SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
        FROM employee
    ) t
    WHERE rn > 1
);
```

---

## How do you get the highest salary in each department?

Medium
{: .label .label-yellow }

**Answer**

```sql
SELECT d.name, MAX(e.salary) AS max_salary
FROM employee e
JOIN department d ON d.id = e.dept_id
GROUP BY d.name;
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
