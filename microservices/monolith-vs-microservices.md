---
title: Monolith vs Microservices
parent: Microservices
nav_order: 1
---

# Monolith vs Microservices
{: .no_toc }

Trade-offs and when to choose which.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Monolith vs microservices: what are the trade-offs?

Medium
{: .label .label-yellow }

**Answer**

| | Monolith | Microservices |
|:--|:--|:--|
| Deployment | One unit | Independent per service |
| Scaling | Whole app | Per service |
| Complexity | Simple operations | Distributed-system complexity |
| Data | Shared DB, ACID | Database per service, eventual consistency |
| Team fit | Small teams | Many autonomous teams |

{: .tip }
Mention the **modular monolith** as a sensible first step. Microservices solve organisational scaling as much as technical scaling.

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
