---
title: Transaction Management
parent: Spring Framework
nav_order: 6
---

# Transaction Management
{: .no_toc }

How @Transactional works, propagation and common pitfalls.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How does @Transactional work and what are the common pitfalls?

Hard
{: .label .label-red }

**Answer**

Spring wraps the bean in a proxy. The proxy starts a transaction through the `PlatformTransactionManager`, calls your method, then commits, or rolls back on a `RuntimeException` or `Error` by default.

| Propagation | Behaviour |
|:--|:--|
| `REQUIRED` (default) | Join existing transaction or create one |
| `REQUIRES_NEW` | Always start a new one, suspend the outer |
| `SUPPORTS` | Join if present, else non-transactional |
| `NOT_SUPPORTED` / `NEVER` / `MANDATORY` | Run without / forbid / require a transaction |

**Pitfalls**
- Self-invocation bypasses the proxy.
- Works only on `public` methods called through the proxy.
- Checked exceptions do not roll back unless `rollbackFor` is set.
- Swallowing the exception in a catch block prevents rollback.

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
