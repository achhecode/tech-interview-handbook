---
title: Design a URL Shortener
parent: System Design
nav_order: 3
---

# Design a URL Shortener
{: .no_toc }

A classic system design problem.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Design a URL shortening service like bit.ly.

Hard
{: .label .label-red }

**Answer**

1. **Requirements:** create short URL, redirect, optional expiry, high read-to-write ratio.
2. **API:** `POST /urls`, `GET /{code}` returns a 301/302 redirect.
3. **Code generation:** a unique ID (counter or distributed ID generator) encoded in **Base62**; 7 characters gives about 3.5 trillion combinations.
4. **Storage:** key-value or relational table `code -> longUrl, createdAt, expiry`.
5. **Scale reads:** cache hot codes in Redis, put a CDN or load balancer in front, shard by code.
6. **Extras:** analytics through async events, rate limiting, abuse checks.

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
