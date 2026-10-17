---
title: Messaging with Kafka
parent: Microservices
nav_order: 6
---

# Messaging with Kafka
{: .no_toc }

Core concepts and delivery guarantees.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Explain the core concepts of Kafka.

Medium
{: .label .label-yellow }

**Answer**

- **Topic:** named stream of records, split into **partitions**.
- **Offset:** position of a record in a partition; ordering is guaranteed **per partition**.
- **Producer** sends records; the key decides the partition.
- **Consumer group:** each partition is consumed by one consumer in the group, so parallelism is limited by partition count.
- **Broker / cluster / replication:** partitions are replicated for fault tolerance, with a leader per partition.

---

## How do you avoid message loss and duplicates in Kafka?

Hard
{: .label .label-red }

**Answer**

- **Producer:** `acks=all`, `enable.idempotence=true`, retries.
- **Broker:** replication factor 3 and `min.insync.replicas=2`.
- **Consumer:** disable auto-commit, commit offsets after processing, and make processing **idempotent** (dedupe by message id).
- Use a dead-letter topic for poison messages.

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
