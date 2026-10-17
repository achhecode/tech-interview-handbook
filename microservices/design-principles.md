---
title: Design Principles
parent: Microservices
nav_order: 2
---

# Design Principles
{: .no_toc }

Principles for service boundaries and robust services.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What are the key design principles of microservices?

Medium
{: .label .label-yellow }

**Answer**

- Single responsibility around a business capability (bounded context from DDD)
- Database per service, no shared tables
- Loose coupling and high cohesion; communicate via APIs or events
- Design for failure: timeouts, retries, circuit breakers
- Stateless services with externalised configuration (12-factor)
- API versioning and backward compatibility
- Observability: logs, metrics, traces

---

## How do you decide service boundaries?

Hard
{: .label .label-red }

**Answer**

Model the business domain, identify bounded contexts, and split along them (orders, payments, inventory). Avoid splitting by technical layer. A good boundary has a small public API, owns its data, and changes independently.

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
