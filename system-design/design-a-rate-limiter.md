---
title: Design a Rate Limiter
parent: System Design
nav_order: 4
---

# Design a Rate Limiter
{: .no_toc }

Algorithms and placement.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How would you design a rate limiter?

Hard
{: .label .label-red }

**Answer**

| Algorithm | Idea |
|:--|:--|
| Fixed window | Count per time window (simple, burst at boundary) |
| Sliding window | Smooths the boundary issue |
| Token bucket | Tokens refill at a rate; allows bursts |
| Leaky bucket | Constant outflow rate |

Place it at the API gateway. For multiple instances, keep counters in Redis using atomic operations (`INCR` with expiry or a Lua script). Return `429 Too Many Requests` with a `Retry-After` header.

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
