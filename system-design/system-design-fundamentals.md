---
title: System Design Fundamentals
parent: System Design
nav_order: 1
---

# System Design Fundamentals
{: .no_toc }

Scalability, CAP theorem and load balancing.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Vertical vs horizontal scaling, and what is the CAP theorem?

Medium
{: .label .label-yellow }

**Answer**

**Vertical** scaling adds more power to one machine; **horizontal** scaling adds more machines (needs stateless services and load balancing).

**CAP:** during a network partition a distributed store can guarantee only one of **Consistency** or **Availability**. Banking systems usually choose consistency; social feeds often choose availability.

---

## How does a load balancer distribute traffic?

Medium
{: .label .label-yellow }

**Answer**

Common algorithms: round robin, weighted round robin, least connections, IP hash (sticky). Layer 4 balances on TCP; Layer 7 can route on URL or headers. Health checks remove unhealthy instances.

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
