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

## Distributed Tracing in Microservices

Distributed tracing is a technique used to track a request as it travels across multiple microservices in a distributed system. It helps developers understand request flow, measure latency, identify bottlenecks, and diagnose failures across service boundaries.

In a monolithic application, debugging a request is often straightforward because most operations happen within one application. In microservices, a single request may pass through several independent services, making troubleshooting more complex.

**Distributed tracing provides end-to-end visibility into the journey of a request.**

### Why Is Distributed Tracing Needed?

Consider an e-commerce application where a customer places an order.

```text
Client
  |
  v
API Gateway (20 ms)
  |
  v
Order Service (50 ms)
  |
  +------> Inventory Service (100 ms)
  |
  +------> Payment Service (800 ms)
  |
  v
Response
```

Suppose the total response time is unexpectedly high. Distributed tracing helps determine whether the delay comes from the Order Service, Inventory Service, Payment Service, or another dependency.

**Common use cases:**

- Identify performance bottlenecks.
- Find slow database queries or downstream API calls.
- Diagnose errors across multiple microservices.
- Understand dependencies between services.
- Measure end-to-end request latency.
- Investigate intermittent failures.
- Correlate service interactions during production incidents.

### Important Concepts: Trace ID and Span ID

**Trace:** Represents the complete journey of a request across multiple services.

**Trace ID:** A unique identifier used to correlate spans belonging to the same distributed request.

**Span:** Represents an individual operation within a trace, such as an HTTP request, database query, or message-processing operation.

**Span ID:** Identifies an individual span within a trace.

**Parent Span ID:** Connects a child span to the operation that initiated it, allowing the tracing system to reconstruct the request hierarchy.

Example:

```text
Trace ID: abc123
|
+-- API Gateway       Span ID: 001
    |
    +-- Order Service Span ID: 002
    |   |
    |   +-- Inventory Span ID: 003
    |
    +-- Payment Service Span ID: 004
```

All these spans belong to the same trace, but each span has its own identifier and timing information.

### How Is Distributed Tracing Implemented?

The typical workflow consists of the following stages:

**Instrumentation:** Each service is configured to create spans for relevant operations. Instrumentation can be automatic, manual, or a combination of both.

**Context propagation:** Trace context is passed between services using request headers or messaging metadata. The W3C Trace Context standard commonly uses `traceparent` and optionally `tracestate` headers.

**Collection and export:** Services send recorded spans to a collector or tracing backend. OpenTelemetry is commonly used to instrument applications and export telemetry.

**Storage and analysis:** A tracing backend stores and processes spans so developers can search traces, inspect errors, and analyze latency.

**Visualization:** A tracing UI displays the trace timeline, span hierarchy, service relationships, and timing information.

```text
Incoming Request
       |
       v
API Gateway
       |
       v
Order Service
       |
       +------> Inventory Service
       |
       +------> Payment Service
                       |
                       v
              Trace Context Propagation
                       |
                       v
             OpenTelemetry Collector
                       |
                       v
               Tracing Backend
              (Jaeger / Zipkin)
                       |
                       v
              Trace Visualization
```

The diagram is conceptual: each instrumented service exports spans, often through an OpenTelemetry Collector, to a compatible tracing backend.

### How Is Distributed Tracing Done in Spring Boot?

For a modern Spring Boot application, a common approach is to use **Micrometer Tracing with OpenTelemetry** or another supported tracing bridge.

Typical components include:

- **Micrometer Tracing:** Provides tracing abstractions in the Spring ecosystem.
- **OpenTelemetry:** Provides instrumentation APIs, SDKs, and telemetry export.
- **OpenTelemetry Collector:** Receives, processes, and exports telemetry.
- **Jaeger or Zipkin:** Stores and visualizes distributed traces, depending on the configured integration.

When properly configured, supported instrumentation can automatically create spans for HTTP requests and outbound calls, propagate trace context, and export traces. Manual spans can be added for important business operations.

Example trace-context header:

```http
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

This W3C header carries the trace ID, parent span ID, and trace flags. Applications should propagate valid tracing context rather than generating an unrelated trace for every downstream request.

### Distributed Tracing vs Logging vs Metrics

| Feature | Distributed Tracing | Logging | Metrics |
|---|---|---|---|
| Purpose | Track a request across services | Record individual events | Measure system behavior over time |
| Data | Traces and spans | Log entries | Numeric measurements |
| Example | Payment call took 800 ms | Payment API returned HTTP 500 | Request error rate is 2% |
| Best for | Finding where a request is slow or failing | Understanding what happened | Monitoring trends and alerting |
| Tools | Jaeger, Zipkin, Grafana Tempo | ELK, Loki | Prometheus, Grafana |

These three observability signals complement each other. Trace IDs can be included in structured logs to help correlate logs with traces.

### Distributed Tracing vs Correlation ID

A correlation ID is commonly used to associate related logs or operations. A trace ID is part of a structured tracing model that includes spans, timing data, and parent-child relationships.

A correlation ID alone does not provide a complete trace timeline. A trace ID can also serve as a correlation identifier when the system is configured consistently.

### Best Practices

- Use standard trace-context propagation.
- Instrument inbound and outbound requests.
- Propagate context through asynchronous messaging.
- Record exceptions and relevant status information.
- Avoid putting passwords, tokens, personal data, or other sensitive information in span attributes.
- Use sampling to control telemetry volume in high-traffic systems.
- Configure appropriate retention and access controls.
- Correlate traces with logs and metrics.
- Monitor the tracing pipeline so telemetry loss does not go unnoticed.

Distributed tracing is an observability tool; it does not automatically fix failures or replace health checks, metrics, logs, or alerting.

### Interview Questions

**What is distributed tracing?**

Distributed tracing tracks a request across multiple services using a trace ID and related spans. It helps diagnose latency, errors, and dependency issues in distributed systems.

**What is the difference between a trace and a span?**

A trace represents the complete request journey. A span represents one operation within that journey.

**How is trace context propagated between microservices?**

Trace context is typically propagated through HTTP headers, such as W3C `traceparent`, or through message metadata for asynchronous communication.

**What tools are used for distributed tracing?**

OpenTelemetry is widely used for instrumentation and telemetry export. Jaeger, Zipkin, and Grafana Tempo are examples of tracing backends.

**How do you find a slow microservice using distributed tracing?**

Search for the relevant trace, inspect its span timeline, and identify operations with unusually high duration, errors, or delays in downstream dependencies.

**Does distributed tracing replace centralized logging?**

No. Tracing explains the request path and timing; logs provide detailed event information. Using trace IDs to correlate the two makes troubleshooting more effective.

---

### Key Takeaway

Distributed tracing provides end-to-end visibility into requests flowing through microservices. For a Spring Boot architecture, Micrometer Tracing and OpenTelemetry can generate and propagate trace context, while a backend such as Jaeger or Zipkin can help visualize spans and identify performance bottlenecks.



---



## What are the types of fault-tolerance mechanisms in Spring Boot Microservices?

{: .no_toc }

### Interview Answer

Fault tolerance is the ability of a microservice to continue operating, recover, or degrade gracefully when one of its dependencies fails.

In Spring Boot microservices, we use different fault-tolerance mechanisms to prevent failures from cascading across the system.

The main mechanisms are:

1. **Circuit Breaker** — stops repeated calls to an unhealthy service.
2. **Retry** — retries transient failures.
3. **Timeout** — prevents a service from waiting indefinitely.
4. **Bulkhead** — isolates resources to limit the impact of failures.
5. **Fallback** — provides an alternative response when a dependency fails.
6. **Rate Limiter** — restricts the number of calls within a time period.
7. **Load Balancing** — distributes requests across service instances.
8. **Health Checks and Monitoring** — detect failures and provide operational visibility.
9. **Asynchronous Messaging** — decouples services and supports buffering and independent processing.

In modern Spring Boot applications, **Resilience4j and Spring Cloud LoadBalancer** are common choices for implementing these mechanisms.

### 1. Circuit Breaker

A circuit breaker prevents an application from repeatedly calling a dependency that is failing or responding too slowly.

It has three states:

- **CLOSED:** Requests are allowed; failures are monitored.
- **OPEN:** Calls are rejected or short-circuited for a configured period.
- **HALF_OPEN:** A limited number of test calls determine whether the dependency has recovered.

Example using Resilience4j:

```java
@CircuitBreaker(
    name = "inventoryService",
    fallbackMethod = "inventoryFallback"
)
public String getInventory() {
    return restClient.get()
        .uri("/api/inventory")
        .retrieve()
        .body(String.class);
}

public String inventoryFallback(Throwable ex) {
    return "Inventory temporarily unavailable";
}
```

**Use case:** The Inventory Service is unavailable, so the Order Service stops repeatedly calling it and returns a controlled response.

### 2. Retry

Retry automatically attempts a failed operation again when the failure is likely to be temporary.

Example:

```java
@Retry(name = "paymentService")
public String callPaymentService() {
    return paymentClient.processPayment();
}
```

Retries should be bounded and use appropriate backoff. Avoid retrying non-idempotent operations blindly, especially payments, because a timeout may occur after the payment was already processed.

**Use case:** A downstream service temporarily fails because of a network interruption.

### 3. Timeout

A timeout defines how long a service should wait for a response before treating the operation as failed.

```java
@Bean
RestClient inventoryRestClient(RestClient.Builder builder) {
    var factory = new SimpleClientHttpRequestFactory();
    factory.setConnectTimeout(Duration.ofSeconds(2));
    factory.setReadTimeout(Duration.ofSeconds(3));

    return builder
        .baseUrl("http://inventory-service")
        .requestFactory(factory)
        .build();
}
```

This configures connection and read timeouts for this client. Actual timeout behavior depends on the HTTP client and the type of operation. Configure timeouts for all relevant network calls.

**Use case:** The Inventory Service becomes slow, and the Order Service must not wait indefinitely.

### 4. Bulkhead

The bulkhead pattern isolates resources so that a failure in one dependency does not consume all resources needed by other operations.

For example, a service can limit the number of concurrent calls to a slow Reporting Service.

```java
@Bulkhead(
    name = "reportingService",
    type = Bulkhead.Type.SEMAPHORE
)
public String generateReport() {
    return reportingClient.generate();
}
```

Resilience4j supports semaphore and thread-pool bulkheads.

**Use case:** Reporting requests are slow, but login and order-processing operations must remain responsive.

### 5. Fallback

A fallback provides an alternative result or behavior when an operation fails or is rejected by a resilience mechanism.

```java
@CircuitBreaker(
    name = "productService",
    fallbackMethod = "productFallback"
)
public String getProduct() {
    return productClient.getProduct();
}

public String productFallback(Throwable ex) {
    return "Product details temporarily unavailable";
}
```

A fallback might return cached data, a safe default, or a meaningful error response, depending on the business requirement.

**Important:** A fallback is not a substitute for a circuit breaker, retry, or timeout. It is a recovery strategy that can be used with them.

### 6. Rate Limiter

A rate limiter restricts how many calls can be made within a specified time interval.

```java
@RateLimiter(name = "externalApi")
public String callExternalApi() {
    return externalApiClient.fetchData();
}
```

**Use case:** Preventing excessive requests to a third-party API with a usage quota.

A rate limiter can protect a dependency, but it does not automatically distribute traffic among service instances.

### 7. Load Balancing

Load balancing distributes requests across multiple instances of a service.

For example:

```text
              Order Service
                    |
                    v
             Load Balancer
              /     |     \
             v      v      v
        Inventory Inventory Inventory
         Instance 1 Instance 2 Instance 3
```

Spring Cloud LoadBalancer provides client-side load-balancing capabilities and can integrate with service discovery.

**Use case:** If one instance becomes unavailable, requests can be routed to other healthy instances, provided the discovery and load-balancing setup identifies unhealthy instances correctly.

Load balancing improves availability, but it does not replace circuit breakers, timeouts, or retries.

### 8. Health Checks and Monitoring

Spring Boot Actuator provides endpoints for health information and metrics.

Common endpoints include:

- `/actuator/health` — application health.
- `/actuator/metrics` — available application metrics.
- `/actuator/prometheus` — Prometheus-formatted metrics when the required integration is configured.

Monitoring tools such as Prometheus and Grafana can help identify rising latency, elevated error rates, resource exhaustion, and repeated circuit-breaker openings.

**Use case:** Detecting a failing service before it causes a larger outage.

### 9. Asynchronous Messaging

Message brokers such as Kafka and RabbitMQ allow services to communicate without requiring the consumer to respond during the original request.

```text
Order Service
      |
      | Publish event
      v
 Message Broker
      |
      v
Notification Service
```

If the Notification Service is temporarily unavailable, messages can remain queued for later processing, depending on broker configuration and delivery guarantees.

For reliable processing, consider durable messaging, acknowledgments, bounded retries, dead-letter handling, and idempotent consumers.

### Comparison of Fault-Tolerance Mechanisms

| Mechanism | Main purpose |
|---|---|
| Circuit Breaker | Stop repeated calls to a failing dependency |
| Retry | Recover from transient failures |
| Timeout | Bound how long an operation can wait |
| Bulkhead | Isolate resources and limit failure impact |
| Fallback | Provide alternative behavior |
| Rate Limiter | Restrict request frequency |
| Load Balancer | Distribute requests across service instances |
| Health Checks | Report service health |
| Monitoring | Detect and diagnose failures |
| Asynchronous Messaging | Decouple processing between services |

### Which libraries and frameworks are used?

| Technology | Purpose |
|---|---|
| Resilience4j | Circuit Breaker, Retry, Bulkhead, Rate Limiter |
| Spring Retry | Declarative retry behavior |
| Spring Cloud LoadBalancer | Client-side load balancing |
| Spring Boot Actuator | Health checks and application metrics |
| Micrometer | Metrics instrumentation and integration |
| Kafka / RabbitMQ | Asynchronous messaging |
| Spring Cloud Netflix Hystrix | Legacy circuit-breaker solution; Hystrix is in maintenance mode |
| Netflix Ribbon | Legacy client-side load balancer; superseded in modern Spring Cloud setups by Spring Cloud LoadBalancer |

### Quick Interview Revision

**Question:** What fault-tolerance mechanisms have you used or would you implement in Spring Boot microservices?

**Answer:** I would combine timeouts, retries with backoff, circuit breakers, bulkheads, and appropriate fallback strategies using Resilience4j. For traffic distribution, I would use Spring Cloud LoadBalancer, and for health checks and metrics, Spring Boot Actuator with Micrometer. Where asynchronous processing is appropriate, I would use Kafka or RabbitMQ. The goal is to prevent cascading failures, isolate unhealthy dependencies, and allow the system to recover gracefully.

**Interview tip:** These mechanisms solve different problems. A timeout limits waiting, a retry repeats an operation, a circuit breaker stops calls to a failing dependency, a bulkhead isolates resources, and a fallback defines alternative behavior. They are most effective when configured together according to the business requirements.

