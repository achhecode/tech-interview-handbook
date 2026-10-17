---
title: Caching Strategies
parent: System Design
nav_order: 2
---

# Caching Strategies
{: .no_toc }

Where and how to cache.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Explain common caching strategies and eviction policies.

Medium
{: .label .label-yellow }

**Answer**

- **Cache-aside:** app reads cache, on miss loads from the DB and fills the cache (most common).
- **Write-through:** write to cache and DB together.
- **Write-back:** write to cache, flush to DB later (fast, risk of loss).

**Eviction:** LRU, LFU, TTL. **Problems:** stale data, cache stampede (mitigate with locking or jittered TTLs), and cache penetration (cache null results).

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
