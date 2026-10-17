---
title: Asynchronous Processing
parent: Microservices
nav_order: 5
---

# Asynchronous Processing
{: .no_toc }

Async execution inside a service.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How do you run tasks asynchronously in Spring Boot?

Medium
{: .label .label-yellow }

**Answer**

Enable with `@EnableAsync` and annotate the method with `@Async`. Return `void` or `CompletableFuture`.

```java
@Async("taskExecutor")
public CompletableFuture<Report> generate(long id) {
    return CompletableFuture.completedFuture(build(id));
}
```

**Pitfalls**
- Calling the method from the same class bypasses the proxy.
- The default executor may be unbounded; define a bounded `ThreadPoolTaskExecutor`.
- Transaction and security context do not automatically propagate to the new thread.

For durable background work across services, use a message broker instead.

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
