---
title: Service Discovery and Configuration
parent: Microservices
nav_order: 7
---

# Service Discovery and Configuration
{: .no_toc }

Registry, config server and gateway.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How do services find each other in a microservices system?

Medium
{: .label .label-yellow }

**Answer**

A **service registry** (Eureka, Consul, or Kubernetes DNS) records instances. Clients resolve a logical name and a load balancer picks an instance (Spring Cloud LoadBalancer). Central configuration comes from Spring Cloud Config, Consul or Kubernetes ConfigMaps/Secrets. On Kubernetes, the platform's Service and DNS often replace Eureka.

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
