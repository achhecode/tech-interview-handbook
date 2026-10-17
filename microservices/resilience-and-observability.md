---
title: Resilience and Observability
parent: Microservices
nav_order: 8
---

# Resilience and Observability
{: .no_toc }

Retries, timeouts, tracing and monitoring.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What resilience techniques do you apply to remote calls?

Medium
{: .label .label-yellow }

**Answer**

Timeouts on every call, limited **retries** with exponential backoff and jitter (only for idempotent operations), **circuit breakers**, **bulkheads** to isolate thread pools, **rate limiting**, and fallbacks. Resilience4j provides most of these for Spring Boot.

---

## What are the three pillars of observability?

Medium
{: .label .label-yellow }

**Answer**

**Logs** (structured, with correlation IDs), **metrics** (Micrometer plus Prometheus and Grafana) and **traces** (OpenTelemetry / Micrometer Tracing with Zipkin or Jaeger) to follow one request across services.

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
