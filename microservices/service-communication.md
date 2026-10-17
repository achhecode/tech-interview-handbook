---
title: Synchronous and Asynchronous Communication
parent: Microservices
nav_order: 4
---

# Synchronous and Asynchronous Communication
{: .no_toc }

REST, gRPC and messaging.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## When do you use synchronous vs asynchronous communication?

Medium
{: .label .label-yellow }

**Answer**

**Synchronous (REST, gRPC):** the caller needs an immediate answer (read a price, validate a user). Simple but couples availability and latency. **Asynchronous (Kafka, RabbitMQ):** fire-and-forget or event-driven flows (notifications, order processing). Better decoupling and resilience, but eventual consistency and harder debugging.

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
