---
title: Transactions and ACID
parent: SQL, JPA and Hibernate
nav_order: 3
---

# Transactions and ACID
{: .no_toc }

ACID properties and isolation levels.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What are ACID properties and isolation levels?

Medium
{: .label .label-yellow }

**Answer**

**ACID:** Atomicity, Consistency, Isolation, Durability.

| Isolation level | Prevents |
|:--|:--|
| Read uncommitted | Nothing |
| Read committed | Dirty reads |
| Repeatable read | Dirty and non-repeatable reads |
| Serializable | Dirty, non-repeatable reads and phantom reads |

Higher isolation means more locking or conflicts and less throughput.

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
