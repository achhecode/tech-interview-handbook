---
title: Docker and Kubernetes
parent: DevOps and Cloud
nav_order: 2
---

# Docker and Kubernetes
{: .no_toc }

Containers and orchestration.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Docker image vs container, and how do you containerise a Spring Boot app?

Medium
{: .label .label-yellow }

**Answer**

An **image** is a read-only template; a **container** is a running instance of it.

```dockerfile
FROM eclipse-temurin:21-jdk AS build
WORKDIR /app
COPY . .
RUN ./mvnw -q package -DskipTests

FROM eclipse-temurin:21-jre
COPY --from=build /app/target/app.jar /app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

Multi-stage builds keep the final image small.

---

## What are Pod, Deployment and Service in Kubernetes?

Medium
{: .label .label-yellow }

**Answer**

A **Pod** is the smallest unit running one or more containers. A **Deployment** manages replicas and rolling updates of Pods. A **Service** gives a stable network name and load balancing for Pods. Use **readiness** probes to control traffic and **liveness** probes to restart unhealthy containers.

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
