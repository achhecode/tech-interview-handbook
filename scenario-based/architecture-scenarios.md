---
title: Architecture Scenarios
parent: Scenario-Based Questions
nav_order: 3
---

# Architecture Scenarios
{: .no_toc }

Migration and scaling decisions.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How would you migrate a monolith to microservices?

Hard
{: .label .label-red }

**Answer**

Use the **Strangler Fig** approach: put a gateway in front, extract one bounded context at a time (start with a low-risk, loosely coupled module), give it its own database, keep the monolith running, sync data with events, and retire the old code gradually. Invest in CI/CD and observability before splitting.

---

## Traffic will increase tenfold. What do you do?

Hard
{: .label .label-red }

**Answer**

Measure first and find the bottleneck. Then: scale stateless services horizontally behind a load balancer, add caching, add read replicas or partition the database, move heavy work to asynchronous queues, apply rate limiting, run load tests, and set autoscaling and alerts.

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
