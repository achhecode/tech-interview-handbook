---
title: Projects and Architecture
parent: Introduction and Preparation
nav_order: 5
---

# Projects and Architecture
{: .no_toc }

How to walk through a project and its architecture.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Walk me through the architecture of your latest project.

Medium
{: .label .label-yellow }

**Answer**

Follow this structure: **Context, Architecture, Your role, Hard problem, Result.**

1. **Context:** business problem and users.
2. **Architecture:** draw it and explain the data flow.
3. **Your role:** modules you owned and decisions you made.
4. **Hard problem:** a bottleneck or failure, and how you fixed it.
5. **Result:** metrics (latency, throughput, cost, release time).

```text
Client -> API Gateway -> Order Service -> Payment Service
                              |                |
                              v                v
                          Kafka topic       MySQL / MongoDB
                              |
                              v
                      Notification Service
```

**Key points**
- Be ready to justify every box: why Kafka, why MongoDB, why this cache.
- Know numbers: requests per second, data size, number of services, team size.

**Follow-up questions**
- What would you change if you rebuilt it today?
- How did you handle failures between services?

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
