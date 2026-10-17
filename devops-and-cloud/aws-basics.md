---
title: AWS Basics
parent: DevOps and Cloud
nav_order: 4
---

# AWS Basics
{: .no_toc }

Core AWS services for backend developers.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Which AWS services does a typical Java backend use?

Medium
{: .label .label-yellow }

**Answer**

| Need | Service |
|:--|:--|
| Compute | EC2, ECS/EKS, Lambda |
| Database | RDS, DynamoDB |
| Storage | S3 |
| Messaging | SQS, SNS, MSK (Kafka) |
| Security | IAM, Secrets Manager |
| Monitoring | CloudWatch |

Follow least-privilege IAM roles, and avoid hard-coding credentials.

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
