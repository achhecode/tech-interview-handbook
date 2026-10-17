---
title: Microservice Patterns
parent: Microservices
nav_order: 3
---

# Microservice Patterns
{: .no_toc }

Saga, circuit breaker, API gateway and outbox.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Explain the Saga pattern.

Hard
{: .label .label-red }

**Answer**

A saga manages a distributed business transaction as a sequence of local transactions, each with a **compensating action** to undo it on failure.

- **Choreography:** services react to each other's events (simple, harder to follow as it grows).
- **Orchestration:** a central orchestrator tells each service what to do (clearer flow, one more component).

Example: Order created, Payment reserved, Inventory reserved. If inventory fails, refund the payment and cancel the order.

---

## Explain the Circuit Breaker pattern.

Hard
{: .label .label-red }

**Answer**

It stops calls to a failing dependency to prevent cascading failures.

- **Closed:** calls pass, failures are counted.
- **Open:** calls fail fast (or use a fallback) after the failure threshold.
- **Half-open:** a few trial calls test recovery.

```java
@CircuitBreaker(name = "payment", fallbackMethod = "fallback")
public PaymentResponse pay(PaymentRequest req) { return client.pay(req); }

PaymentResponse fallback(PaymentRequest req, Throwable t) { return PaymentResponse.pending(); }
```

---

## What is the API Gateway pattern?

Medium
{: .label .label-yellow }

**Answer**

A single entry point (e.g. Spring Cloud Gateway) that handles routing, authentication, rate limiting, request aggregation and cross-cutting concerns, so clients do not call each service directly.

---

## What is the Outbox pattern?

Hard
{: .label .label-red }

**Answer**

It avoids the dual-write problem. Write the business change and an event row to an **outbox table** in the same local transaction, then a relay (poller or CDC such as Debezium) publishes the events to Kafka. This guarantees events are not lost when the database commits.

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
