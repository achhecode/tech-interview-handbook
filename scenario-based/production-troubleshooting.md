---
title: Production Troubleshooting
parent: Scenario-Based Questions
nav_order: 1
---

# Production Troubleshooting
{: .no_toc }

Debugging live issues.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## An API became slow in production. How do you investigate?

Hard
{: .label .label-red }

**Answer**

1. **Scope:** is it one endpoint or all? Since when? Check recent deployments.
2. **Metrics and traces:** latency percentiles, error rate, a trace showing which hop is slow.
3. **Dependencies:** database slow queries (`EXPLAIN`), connection pool exhaustion, external API timeouts.
4. **JVM:** GC pauses, thread dump for blocked or waiting threads, CPU spikes.
5. **Fix and prevent:** roll back or hotfix, add timeouts, indexes or caching, then add alerts and a regression test.

---

## The service crashes with OutOfMemoryError. What do you do?

Hard
{: .label .label-red }

**Answer**

Capture a heap dump (`-XX:+HeapDumpOnOutOfMemoryError`), analyse it with MAT for the largest retained objects, check recent changes, and fix the cause (unbounded cache, large result sets without pagination, leaks). As a stop-gap, restart, and tune `-Xmx` only after finding the cause.

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
