---
title: API Design Scenarios
parent: Scenario-Based Questions
nav_order: 2
---

# API Design Scenarios
{: .no_toc }

Correctness and scale in APIs.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How do you prevent duplicate processing when a client retries a payment API?

Hard
{: .label .label-red }

**Answer**

Make the endpoint **idempotent**. The client sends an `Idempotency-Key` header; the server stores the key with the result (unique constraint plus status). A retry with the same key returns the stored response instead of charging again. Also use a database transaction and unique constraints as the final safety net.

---

## How do you return a very large result set from an API?

Medium
{: .label .label-yellow }

**Answer**

Use **pagination** (page/size, or keyset/cursor pagination for large tables), limit page size, index the sort column, and stream or export asynchronously for huge reports.

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
