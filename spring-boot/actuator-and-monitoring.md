---
title: Actuator and Monitoring
parent: Spring Boot
nav_order: 7
---

# Actuator and Monitoring
{: .no_toc }

Health checks, metrics and production readiness.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is Spring Boot Actuator?

Medium
{: .label .label-yellow }

**Answer**

Actuator exposes production-ready endpoints such as `/actuator/health`, `/actuator/metrics` and `/actuator/info`. Add `spring-boot-starter-actuator` and expose endpoints with `management.endpoints.web.exposure.include`. Metrics are published through Micrometer to systems like Prometheus. Secure sensitive endpoints and never expose all of them publicly.

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
