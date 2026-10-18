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


## Which Microservice Design Patterns Have You Used and Why?

In a microservices architecture, different design patterns help address challenges such as service discovery, fault tolerance, communication, data consistency, and scalability.

For a Java and Spring Boot microservices application, the following patterns are particularly useful.

### Service Registry and Service Discovery

A Service Registry maintains information about available service instances, allowing services to discover each other dynamically instead of relying on hardcoded IP addresses.

**Why use it?**
- Supports dynamic scaling.
- Simplifies service-to-service communication.
- Helps locate available service instances.
- Works with load balancing and health-check mechanisms.

**Technologies:** Netflix Eureka, Consul, Kubernetes DNS.

Example: Order Service discovers Inventory Service using its logical service name rather than a hardcoded host and port.

### Circuit Breaker

The Circuit Breaker pattern prevents a service from repeatedly calling an unhealthy or unresponsive dependency.

It generally has three states:

- **Closed:** Calls are allowed and failures are monitored.
- **Open:** Calls are rejected or handled through a fallback.
- **Half-open:** Limited test calls determine whether the dependency has recovered.

**Why use it?**

It helps prevent cascading failures, limits unnecessary calls to unhealthy dependencies, and improves system resilience.

**Technology:** Resilience4j with Spring Boot.

Example: If Payment Service repeatedly fails, Order Service temporarily stops making normal calls to it rather than allowing requests to accumulate.

### API Gateway

An API Gateway provides a common entry point for external clients accessing backend microservices.

**Responsibilities:**
- Request routing.
- Authentication integration.
- Rate limiting.
- TLS termination.
- Request logging.
- API versioning and transformations.

**Why use it?**

It simplifies client integration and centralizes appropriate cross-cutting concerns.

**Technologies:** Spring Cloud Gateway, Kong, NGINX, cloud-managed API gateways.

Example: A mobile application calls the gateway, which routes requests to User Service, Order Service, or Product Service.

### Event-Driven Architecture

In Event-Driven Architecture, services communicate by publishing and consuming events rather than relying exclusively on direct synchronous calls.

**Technologies:** Apache Kafka, RabbitMQ.

**Why use it?**
- Reduces direct runtime dependencies.
- Enables asynchronous processing.
- Supports multiple independent consumers.
- Helps absorb traffic spikes.
- Allows services to evolve independently.

Example: When an order is placed, Order Service publishes an `OrderCreated` event. Inventory Service updates stock, and Notification Service sends a confirmation.

Consumers should handle duplicate events safely, and asynchronous workflows must account for eventual consistency.

### CQRS (Command Query Responsibility Segregation)

CQRS separates operations that modify data (commands) from operations that retrieve data (queries).

**Why use it?**
- Allows read and write workloads to be optimized independently.
- Supports different models for updates and queries.
- Can improve read performance for complex query requirements.

Example: An e-commerce application writes order data to a transactional model while using a separate read model to display order history.

CQRS does not necessarily require separate databases or microservices. It introduces additional complexity and is most useful when the read and write requirements justify it.

### Other Useful Patterns

- **Database per Service:** Each service owns its data and controls access through its APIs or events.
- **Saga:** Coordinates a business transaction across multiple services using local transactions and compensating actions.
- **Retry:** Retries transient failures using bounded attempts and backoff.
- **Bulkhead:** Isolates resource pools or operations to limit the impact of failures.
- **Strangler Fig:** Gradually replaces a legacy system by routing functionality to new services.
- **Transactional Outbox:** Stores a business change and its corresponding event record in one local database transaction, enabling reliable event publication.

### Sample Interview Answer

**Question: Which microservice design patterns have you used so far, and why?**

In a typical Spring Boot microservices architecture, I would discuss patterns based on the actual project implementation.

For example:

"I use an API Gateway to provide a common entry point for external clients and centralize concerns such as routing and rate limiting. For internal communication, I use REST or service discovery where appropriate, and asynchronous messaging with Kafka when services need to process business events independently.

For resilience, I use Resilience4j Circuit Breaker, timeouts, and bounded retries to reduce the impact of downstream failures. For data management, I prefer Database per Service to maintain service ownership. When a business transaction spans multiple services, I consider the Saga pattern to coordinate local transactions and handle failures."

In an interview, mention only the patterns you have genuinely implemented, and be prepared to explain the problem, implementation, trade-offs, and results.

---

## Which Design Patterns Are Used for Database Design in Microservices?
{: .no_toc }

Database design in microservices focuses on data ownership, consistency, transaction management, query performance, and service independence.

### Database per Service

Each microservice owns its database or logically isolated data store. Other services access that data through APIs or events rather than directly querying the owning service's tables.

**Benefits:**
- Independent schema evolution.
- Clear data ownership.
- Reduced coupling between services.
- Freedom to select appropriate storage technologies.

**Trade-off:** Cross-service queries and transactions become more complex.

### Shared Database

Multiple microservices access the same database instance or schema.

This can simplify legacy integration and reporting, but shared tables create tight coupling when multiple services read and write each other's data.

A shared database instance with separate schemas, permissions, and clear ownership is different from allowing every service unrestricted access to all tables.

**Best practice:** Prefer independent ownership, and use a shared database only when the trade-offs are understood and justified.

### Event Sourcing

Event Sourcing stores state changes as an append-only sequence of events instead of relying exclusively on the latest database state.

For example:

```text
OrderCreated
     |
     v
OrderPaid
     |
     v
OrderShipped
```

The current state can be reconstructed by applying the events.

**Benefits:** Audit history, replayability, and a detailed record of changes.

**Trade-offs:** Event schema evolution, projections, storage growth, and operational complexity.

Event Sourcing is not required for every microservice.

### CQRS

CQRS separates command processing from query processing. A write model handles state changes, while a read model is optimized for retrieval.

It is useful when read and write requirements differ significantly.

CQRS can be combined with Event Sourcing, but the two patterns are independent and do not have to be implemented together.

### Saga Pattern

Saga manages a business transaction spanning multiple services. Each service executes a local transaction, and compensating actions handle failures when a business operation needs to be reversed.

Example: Order Service creates an order, Payment Service charges the customer, and Inventory Service reserves stock.

If stock reservation fails after payment, the workflow can initiate a refund or another appropriate compensation.

### Materialized View

A Materialized View maintains a precomputed representation of data to support frequent or expensive queries.

In microservices, a read model can combine data obtained from several services or events.

**Example:** A customer dashboard displays order summaries, payment status, and delivery information without querying each source independently for every page load.

The view must be updated as source data changes, so it may be eventually consistent.

### API Composition

API Composition combines responses from multiple services to construct a result for a client.

Example: A dashboard API retrieves customer details from User Service and order summaries from Order Service, then combines them.

**Trade-off:** It can increase latency and introduce runtime dependencies on the services being called.

### Read Replicas

Read replicas are database copies used to serve read queries, reducing read pressure on a primary database.

They are useful when read traffic is high, but replication lag can cause stale reads. Read replicas improve database scalability; they do not, by themselves, establish microservice data ownership.

### Comparison Table

| Pattern | Main Purpose | Main Trade-off |
|---|---|---|
| Database per Service | Data ownership and isolation | Cross-service queries become harder |
| Shared Database | Simplifies shared-data access | Risk of tight coupling |
| Event Sourcing | Preserve state-change history | More operational complexity |
| CQRS | Optimize reads and writes independently | Synchronization and model maintenance |
| Saga | Manage multi-service business transactions | Compensation and workflow complexity |
| Materialized View | Speed up complex reads | Projection maintenance and stale data |
| API Composition | Combine data from multiple services | Latency and runtime dependencies |
| Read Replicas | Scale read workloads | Replication lag and stale reads |

---

## What Is the Saga Microservice Pattern?
{: .no_toc }

The Saga pattern is a distributed transaction pattern used when a business operation involves multiple microservices, each managing its own local database transaction.

In a monolithic application, several database updates may be coordinated through a single ACID transaction. In microservices, each service owns its data, so one ordinary local database transaction generally cannot atomically cover all participating services.

Saga addresses this by dividing a business transaction into a sequence of local transactions. If a later step fails, the workflow can execute compensating transactions to address the effects of earlier successful steps.

### Example: E-Commerce Order Processing

Consider a customer placing an order.

```text
1. Order Service
   Create order as PENDING
          |
          v
2. Payment Service
   Charge customer
          |
          v
3. Inventory Service
   Reserve stock
          |
          v
4. Order Service
   Mark order CONFIRMED
```

If all steps succeed, the order is confirmed.

Now suppose payment succeeds but inventory reservation fails. The system cannot simply roll back the Payment Service's committed database transaction. Instead, the Saga initiates a compensating action, such as refunding the payment, and updates the order status accordingly.

```text
Payment succeeds
       |
       v
Inventory reservation fails
       |
       v
Initiate payment refund
       |
       v
Mark order as FAILED or COMPENSATION_PENDING
```

The exact final state depends on the business requirements and whether compensation succeeds.

### Types of Saga Pattern

**Choreography-Based Saga**

Each service publishes events and reacts to events from other services. There is no central coordinator directing every step.

Example:

```text
Order Service
     |
     | OrderCreated
     v
Payment Service
     |
     | PaymentCompleted
     v
Inventory Service
     |
     | InventoryReserved
     v
Order Service
```

**Advantages:**
- No central workflow coordinator.
- Services communicate through events.
- Suitable for relatively simple event-driven workflows.

**Disadvantages:**
- Workflow behavior can be difficult to understand across many services.
- Event dependencies can become complex.
- Monitoring and failure recovery require careful design.

**Orchestration-Based Saga**

A central orchestrator coordinates the sequence of steps, tracks workflow state, and tells each service which action to execute.

```text
             Saga Orchestrator
              /      |      \
             v       v       v
           Order   Payment  Inventory
           Service Service  Service
```

**Advantages:**
- Workflow logic is centralized.
- Easier to inspect the sequence of operations.
- Explicit handling of retries, timeouts, and compensation.

**Disadvantages:**
- The orchestrator adds complexity.
- It must be designed for recovery and availability.
- Poorly designed orchestration can create excessive coupling.

The orchestrator coordinates the workflow but does not make independent databases behave like one atomic database transaction.

### What Is a Compensating Transaction?

A compensating transaction is a business operation that counteracts the effect of a previously completed local transaction.

Examples:

| Completed Action | Possible Compensation |
|---|---|
| Payment charged | Refund payment |
| Inventory reserved | Release inventory |
| Hotel room booked | Cancel booking |
| Order created | Cancel order |

Compensation is not always an exact rollback. For example, a refund may take time to settle, and a shipment that has already been dispatched may require a return workflow.

### Important Saga Implementation Considerations

- **Idempotency:** Reprocessing the same command should not create duplicate payments or reservations.
- **Retries:** Retry transient failures with bounded attempts and backoff.
- **Timeouts:** Detect steps that take too long.
- **Durable state:** Persist workflow progress so the Saga can resume after a restart.
- **Compensation:** Define what happens when each step fails.
- **Outbox pattern:** Reliably record events alongside local database changes.
- **Observability:** Track Saga IDs, events, failures, and compensation progress.
- **Dead-letter handling:** Provide a way to inspect and recover messages that cannot be processed.
- **Consistency:** Accept that services may temporarily observe intermediate states.

A Saga does not guarantee that every workflow will finish successfully. Some failures require manual intervention, and the system must represent these states clearly.

### Saga vs Two-Phase Commit

| Aspect | Saga | Two-Phase Commit (2PC) |
|---|---|---|
| Approach | Sequence of local transactions | Coordinated prepare and commit phases |
| Consistency | Eventual consistency is common | Aims for atomic distributed commit |
| Rollback | Business compensation | Abort before commit when possible |
| Availability | Can support independently progressing services | Coordinator and resource constraints may block progress |
| Complexity | Workflow and compensation logic | Transaction coordination and participant support |
| Best fit | Long-running business workflows | Environments where supported distributed atomic transactions are appropriate |

Saga is often preferred for long-running microservice workflows because it avoids requiring one global database transaction. However, it introduces eventual consistency and compensation logic, which must be handled deliberately.

### Saga Interview Questions

**Why do we need the Saga pattern?**

Because a business transaction may span several microservices with independent databases. Saga coordinates local transactions and defines recovery actions when a later step fails.

**What is the difference between choreography and orchestration?**

Choreography uses events and service reactions to coordinate the workflow. Orchestration uses a coordinator to direct and track each step.

**Does Saga provide complete rollback?**

No. It uses compensating business operations. These may not reverse every external effect exactly, and some failures require retries or manual intervention.

**How do you prevent duplicate payments in a Saga?**

Use idempotency keys or stable operation identifiers, persist transaction state, and ensure retries cannot charge the same business operation more than once.

**How does Saga handle service failures?**

The workflow can retry transient failures, execute compensating transactions, persist its progress, and expose unresolved failures for recovery.

**Can Saga be implemented using Kafka?**

Yes. Kafka can carry the events or commands used in a Saga. The implementation still needs durable workflow state, idempotent consumers, failure handling, and compensation logic. Kafka alone does not implement the entire Saga pattern.


### Final Revision

- **Service Registry:** Discovers service instances.
- **Circuit Breaker:** Limits repeated calls to unhealthy dependencies.
- **API Gateway:** Provides a common entry point for external clients.
- **Event-Driven Architecture:** Supports asynchronous, decoupled communication.
- **CQRS:** Separates command and query models.
- **Database per Service:** Establishes independent data ownership.
- **Event Sourcing:** Records state changes as events.
- **Saga:** Coordinates transactions across independent services.
- **Materialized View:** Provides a precomputed read model.
- **API Composition:** Combines data from multiple service APIs.
- **Transactional Outbox:** Helps reliably publish events associated with database changes.


---



## Explain the CQRS Concept

**CQRS (Command Query Responsibility Segregation)** is a design pattern that separates operations that modify data from operations that retrieve data.

- **Command:** Changes the state of the system, such as creating an order, updating a user, or cancelling a booking.
- **Query:** Retrieves data without changing the system's state, such as fetching order details or displaying a customer profile.

The main objective of CQRS is to allow read and write operations to be designed, optimized, and potentially scaled independently.

### How CQRS Works

Consider an e-commerce application with thousands of users browsing products but comparatively fewer users updating product information.

Without CQRS, the same application model and database access path may handle both reads and writes.

With CQRS, the architecture can use separate models for commands and queries.

```text
                  Client
                    |
              +-----+-----+
              |           |
              v           v
         Command API   Query API
              |           |
              v           v
         Write Model   Read Model
              |           ^
              v           |
        Write Database    |
              |           |
              +--> Events-+
```

This is a conceptual architecture. The read and write models may use the same database initially or separate data stores when the requirements justify them. If separate read models are updated asynchronously, they may temporarily lag behind the write model.

### Example Using Spring Boot

Suppose an application manages employee records.

**Command operation:**

```java
public record CreateEmployeeCommand(
    String name,
    String department
) {}
```

The command handler validates the request and creates the employee.

```java
@Service
public class EmployeeCommandService {

    private final EmployeeRepository repository;

    public EmployeeCommandService(
            EmployeeRepository repository) {
        this.repository = repository;
    }

    public Long createEmployee(
            CreateEmployeeCommand command) {

        Employee employee = new Employee();
        employee.setName(command.name());
        employee.setDepartment(command.department());

        return repository.save(employee).getId();
    }
}
```

**Query operation:**

```java
@Service
public class EmployeeQueryService {

    private final EmployeeRepository repository;

    public EmployeeQueryService(
            EmployeeRepository repository) {
        this.repository = repository;
    }

    public Employee getEmployee(Long id) {
        return repository.findById(id)
            .orElseThrow(() ->
                new RuntimeException("Employee not found"));
    }
}
```

These examples separate command and query responsibilities at the application-service level. They do not, by themselves, implement a fully independent CQRS architecture with separate databases or independently deployed services.

### Benefits of CQRS

- Independent optimization of read and write workloads.
- Potential for independent scaling of query and command components.
- Read models can be designed for specific query requirements.
- Complex business rules can be concentrated in command handling.
- Supports event-driven architectures and specialized projections.

### Disadvantages of CQRS

- Increased architectural complexity.
- Additional work to keep read models updated.
- Possible eventual consistency between read and write models.
- More infrastructure, monitoring, and testing requirements.
- Often unnecessary for simple CRUD applications.

### When Should You Use CQRS?

CQRS can be useful when:

- The application has significantly different read and write workloads.
- Read queries involve complex joins, aggregations, or reporting.
- Read and write models have different scaling requirements.
- Different teams need to evolve query and command models independently.
- The business domain has complex validation and state transitions.

**Interview point:** CQRS does not automatically mean two databases, two microservices, or Event Sourcing. Those are architectural choices that can be combined with CQRS when useful.

---

## Which Microservice Pattern Will You Use for Read-Heavy and Write-Heavy Applications?
{: .no_toc }

The choice depends on the workload, consistency requirements, data model, and scaling constraints. CQRS can help when read and write paths need different optimization strategies, while Event Sourcing is useful when maintaining a durable history of state changes is a business requirement.

### Read-Heavy Applications

For read-heavy systems, consider:

**CQRS:** Separates query handling from command handling so the read path can be optimized independently.

**Caching:** Reduces repeated database reads. Examples include Redis and application-level caches.

**Read Replicas:** Offload read traffic from the primary database.

**Materialized Views:** Precompute frequently requested results, such as dashboards or reporting summaries.

Example: An e-commerce product catalog may receive thousands of product-detail requests while product updates happen relatively infrequently.

A suitable design could use a read-optimized query model, caching, and read replicas. Full CQRS with separate databases is appropriate only if the added complexity provides a meaningful benefit.

### Write-Heavy Applications

For write-heavy systems, consider:

**Efficient transactional writes:** Optimize indexes, transactions, database connections, and batching.

**Partitioning and sharding:** Distribute data and write load when a single database cannot meet throughput requirements.

**Asynchronous processing:** Use Kafka or RabbitMQ to buffer work and decouple producers from consumers where the business workflow permits it.

**Event Sourcing:** Store state changes as an ordered history of events when auditability, reconstruction, or historical analysis is important.

Event Sourcing is not automatically the best pattern for every write-heavy application. It can introduce additional storage, projection, and operational overhead.

### Comparison

| Requirement | Suitable approach |
|---|---|
| Very high read volume | Caching, read replicas, CQRS where justified |
| Complex reporting | Materialized views or specialized query models |
| High write throughput | Database optimization, batching, partitioning |
| Durable history of every business change | Event Sourcing where justified |
| Independent asynchronous processing | Kafka or RabbitMQ |
| Different read and write scaling needs | CQRS with independently scalable components |

**Sample interview answer:**

"For a read-heavy application, I would first evaluate caching, database indexing, and read replicas. If the read and write models have significantly different requirements, I would consider CQRS so each side can be optimized independently. For write-heavy applications, I would focus on database throughput, batching, partitioning, and asynchronous processing where appropriate. I would use Event Sourcing when a complete history of business changes is required, rather than choosing it solely because the write volume is high."

---

## Explain the Choreography Pattern in Microservices
{: .no_toc }

Choreography is a distributed coordination pattern in which microservices communicate through events or messages without a central orchestrator directing every step.

Each service owns its business logic, performs its local operation, and publishes events that other services can consume.

### How Choreography Works

Consider an e-commerce order workflow.

```text
Order Service
     |
     | OrderCreated
     v
Payment Service
     |
     | PaymentCompleted
     v
Inventory Service
     |
     | InventoryReserved
     v
Notification Service
```

Each service reacts to events according to its own responsibilities. The arrows represent a possible event-driven workflow, not direct method calls between services.

For example:

- Order Service creates an order and publishes `OrderCreated`.
- Payment Service consumes the event, processes payment, and publishes `PaymentCompleted` or `PaymentFailed`.
- Inventory Service responds to the relevant payment event and attempts to reserve stock.
- Notification Service sends the appropriate customer notification.

In a real system, the workflow must define what happens when payment fails, stock is unavailable, or a message is processed more than once.

### Implementation Technologies

- Apache Kafka.
- RabbitMQ.
- Spring Boot messaging integrations.
- Transactional Outbox pattern for reliable event publication.

### Advantages

- No central orchestrator is required.
- Services remain relatively independent.
- Supports asynchronous processing.
- New consumers can subscribe to events without requiring the producer to call them directly.
- Can support independent deployment and scaling.

### Disadvantages

- The complete workflow can be difficult to understand across many services.
- Event dependencies may become complex.
- Debugging requires distributed tracing and good event observability.
- Duplicate delivery and eventual consistency must be handled.
- Failure recovery and compensating actions need careful design.

### Choreography vs Orchestration

| Aspect | Choreography | Orchestration |
|---|---|---|
| Coordination | Services react to events | A coordinator directs the workflow |
| Central controller | Not required | Required |
| Workflow visibility | Distributed across services | Centralized in the orchestrator |
| Coupling | Event-contract dependencies remain | Services depend on coordinator commands or contracts |
| Best suited for | Relatively simple event-driven workflows | Complex workflows with explicit steps and recovery |

**Important:** Choreography does not mean there is no coordination at all. Coordination emerges from the events and the behavior of participating services. It is also different from service discovery, which helps locate service instances, and an API Gateway, which routes client requests.

### Interview Answer

"Choreography is a microservice coordination pattern in which services communicate through events without a central orchestrator controlling the entire workflow. For example, Order Service publishes an `OrderCreated` event, Payment Service processes it and publishes a payment result, and Inventory Service reacts to the relevant event. This reduces direct runtime coupling, but it requires reliable event delivery, idempotent consumers, observability, and explicit failure-handling logic."

### Quick Revision

- **CQRS:** Separates command responsibilities from query responsibilities.
- **Command:** Changes application state.
- **Query:** Retrieves data without changing application state.
- **Event Sourcing:** Persists state changes as events.
- **Choreography:** Services coordinate by reacting to events.
- **Orchestration:** A coordinator directs a multi-service workflow.
- **Saga:** Manages distributed business transactions through local transactions and compensating actions.
- **Read-heavy optimization:** Consider caching, read replicas, materialized views, and CQRS.
- **Write-heavy optimization:** Consider database tuning, batching, partitioning, and asynchronous processing.

The key interview takeaway is to explain not just what a pattern does, but **which problem it solves, why it fits the workload, and what complexity it introduces**.

---


## What Is the Circuit Breaker Pattern? What Are Its Examples?

The **Circuit Breaker pattern** is a resilience design pattern used in distributed systems to prevent repeated calls to an unhealthy or unresponsive service. It helps reduce cascading failures and allows a system to fail fast instead of wasting resources waiting for a dependency that may not recover immediately.

It works like an electrical circuit breaker: when failures exceed a configured threshold, the circuit opens and temporarily stops normal calls to the failing dependency.

### Why Do We Need a Circuit Breaker?

Consider an e-commerce application where Order Service calls Payment Service.

```text
Order Service
      |
      v
Payment Service
      |
      v
Payment Database
```

Suppose Payment Service becomes slow because of database problems. Without appropriate timeouts and resilience controls, requests can accumulate, threads can remain blocked, and Order Service may also become unavailable.

A circuit breaker detects repeated failures and temporarily blocks calls to Payment Service, helping protect the calling service.

### States of a Circuit Breaker

A circuit breaker generally has three states:

<box gap={3}>
  <row align=start gap={3}>
    <box background="rgba(34,197,94,0.13)" radius="lg" padding={3} align=center justify=center minWidth="104px">
      <title size="md" color="success">CLOSED</title>
    </box>
    <box flex="1" gap={1}>
      **Normal operation**

      Requests are allowed through. The circuit breaker monitors outcomes and records failures.
    </box>
  </row>
  <divider color="subtle" />
  <row align=start gap={3}>
    <box background="rgba(239,68,68,0.12)" radius="lg" padding={3} align=center justify=center minWidth="104px">
      <title size="md" color="danger">OPEN</title>
    </box>
    <box flex="1" gap={1}>
      **Fail fast**

      Normal calls are blocked temporarily. The application may return a fallback response or an appropriate error.
    </box>
  </row>
  <divider color="subtle" />
  <row align=start gap={3}>
    <box background="rgba(234,179,8,0.14)" radius="lg" padding={3} align=center justify=center minWidth="104px">
      <title size="md" color="warning">HALF-OPEN</title>
    </box>
    <box flex="1" gap={1}>
      **Test recovery**

      After a configured wait period, a limited number of calls are allowed. Success can close the circuit; renewed failures can reopen it.
    </box>
  </row>
</box>

```text
       Failures exceed threshold
CLOSED ----------------------------> OPEN
   ^                                  |
   |                                  |
   | Successful test calls            | Wait duration expires
   |                                  v
   +----------------------------- HALF-OPEN
                      Failed test calls
                              |
                              v
                             OPEN
```

The exact thresholds, permitted test calls, and transition rules depend on the circuit breaker configuration.

### Real-World Examples

**Payment Service:** If the payment provider repeatedly times out, Order Service temporarily stops making normal payment calls. The application can show a payment-unavailable message or mark the order for later processing.

**Inventory Service:** If Inventory Service is unavailable, Product Service may return product details without current stock information only if that behavior is acceptable. It should not falsely claim stock is available.

**Third-party shipping API:** If a shipping provider fails repeatedly, the application can stop making unnecessary requests and temporarily show that shipping estimates are unavailable.

**Notification Service:** If an email provider becomes unavailable, notifications may be queued for later processing rather than repeatedly blocking order creation.

A fallback must preserve business correctness. Returning a default success response for a failed payment, for example, would be unsafe.

### Circuit Breaker vs Retry vs Timeout

| Pattern | Purpose | Example |
|---|---|---|
| Circuit Breaker | Temporarily blocks calls to an unhealthy dependency | Stop normal payment requests after repeated failures |
| Retry | Repeats an operation after a potentially transient failure | Retry a request after a temporary network error |
| Timeout | Limits how long a call can wait | Stop waiting for a payment response after 2 seconds |
| Bulkhead | Isolates resources to limit the impact of failures | Limit concurrent calls to Payment Service |

These patterns are complementary. Retries should be bounded and use backoff; aggressive retries can make an overloaded dependency even worse.

### How to Implement a Circuit Breaker in Spring Boot

For modern Spring Boot applications, **Resilience4j** is a common choice. Netflix Hystrix, used in many older examples, is in maintenance mode and is not the preferred choice for a new implementation.

Add the appropriate Resilience4j Spring Boot integration dependency to your project, along with Spring AOP support as required by your setup.

#### Service Example

```java
@Service
public class PaymentClientService {

    private final PaymentService paymentService;

    public PaymentClientService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @CircuitBreaker(
        name = "paymentService",
        fallbackMethod = "paymentFallback"
    )
    public String processPayment() {
        return paymentService.doSomething();
    }

    public String paymentFallback(
            Throwable exception) {
        return "Payment service is temporarily unavailable";
    }
}
```

Import the annotation:

```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
```

In this example, Resilience4j monitors eligible failures from `paymentService.doSomething()`. When the configured circuit breaker opens, calls can be rejected and the fallback method can be invoked.

The fallback signature must match the original method's parameters, if any, followed by an appropriate exception parameter. Depending on the application's needs, you may want to handle specific exceptions rather than using a broad `Throwable`.

#### Configuration Example

In `application.yml`:

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 3
```

This configuration means:

- The breaker evaluates a sliding window of up to 10 calls.
- It requires at least 5 calls before evaluating the failure rate.
- A failure rate of 50% or higher can open the circuit.
- The open state lasts 10 seconds before the breaker transitions to half-open.
- Up to 3 test calls are permitted in the half-open state.

Actual behavior also depends on exception classification and other Resilience4j settings.

**Important:** A circuit breaker does not automatically switch to another healthy service instance. Load balancing handles instance selection, while a circuit breaker controls whether calls should proceed. A fallback is also not the same as a guaranteed retry or failover.

### How Does Hystrix Work?

In older Spring Cloud applications, Netflix Hystrix provided circuit breaker functionality through annotations such as `@HystrixCommand` and configuration such as `@EnableCircuitBreaker`.

For example, older code might look like this:

```java
@HystrixCommand(fallbackMethod = "defaultResponse")
public String callService() {
    return myService.doSomething();
}

public String defaultResponse() {
    return "Service is temporarily unavailable";
}
```

This illustrates the historical approach. Hystrix is no longer the recommended library for new Spring Boot applications; use a supported resilience library such as Resilience4j instead.

### Best Practices

- Configure timeouts for remote calls.
- Use bounded retries with exponential backoff and jitter where appropriate.
- Tune the failure threshold and sliding window based on actual traffic.
- Define business-safe fallback behavior.
- Monitor circuit breaker state changes and failure rates.
- Avoid retrying non-idempotent operations without suitable safeguards.
- Consider bulkheads to isolate resource consumption.
- Test open, half-open, recovery, and fallback behavior.

### Interview Questions

**What problem does the Circuit Breaker pattern solve?**

It reduces cascading failures by temporarily stopping calls to a dependency that is failing repeatedly.

**What are the three states of a circuit breaker?**

Closed, Open, and Half-open.

**What is the difference between Circuit Breaker and Retry?**

Retry attempts an operation again after a failure. Circuit Breaker temporarily prevents normal calls when the dependency is considered unhealthy.

**Does a circuit breaker guarantee service recovery?**

No. It limits the impact of failures and periodically permits recovery checks, but the dependency must actually recover for successful calls to resume.

**Which circuit breaker library would you use in Spring Boot?**

For a new application, I would generally choose Resilience4j because it integrates with the Spring ecosystem and supports circuit breakers, retries, rate limiting, and bulkheads.

---

### Key Takeaway

The Circuit Breaker pattern improves resilience by detecting repeated failures and temporarily blocking calls to an unhealthy dependency. In modern Spring Boot applications, Resilience4j is a practical choice, usually combined with timeouts, carefully controlled retries, and business-safe fallback handling.


---


## Explain the Annotations Used to Implement Circuit Breaker in Spring Boot

In Spring Boot, a circuit breaker can be implemented using libraries such as **Resilience4j**. Older applications may use Netflix Hystrix, but Hystrix is in maintenance mode and is not recommended for new implementations.

Resilience4j integrates with Spring Boot and provides annotations and configuration options for circuit breakers, retries, rate limiting, and bulkheads.

### Important Circuit Breaker Annotations

#### `@CircuitBreaker`

The `@CircuitBreaker` annotation applies circuit breaker behavior to a method.

It monitors eligible failures and can invoke a fallback method when the circuit is open or a call fails.

```java
@CircuitBreaker(
    name = "paymentService",
    fallbackMethod = "paymentFallback"
)
public String callDependency() {
    return paymentService.processPayment();
}

public String paymentFallback(Throwable ex) {
    return "Payment service is temporarily unavailable";
}
```

**Important attributes:**

- `name`: Identifies the circuit breaker instance.
- `fallbackMethod`: Specifies the method to invoke when fallback handling is triggered.

The fallback method must have compatible parameters and return type. If the original method has parameters, the fallback should accept those parameters followed by an appropriate exception parameter.

#### `@Retry`

The `@Retry` annotation retries a method when a configured failure occurs.

```java
@Retry(name = "paymentService")
public String callPaymentService() {
    return paymentService.processPayment();
}
```

**Use case:** Retry a transient network failure or temporary service error.

Retries should be bounded and used carefully. Repeatedly retrying an overloaded dependency can make the failure worse.

#### `@TimeLimiter`

The `@TimeLimiter` annotation limits how long an asynchronous operation is allowed to run before a timeout is reported.

It is commonly used with `CompletionStage`, `CompletableFuture`, or supported reactive return types, depending on the integration.

```java
@TimeLimiter(name = "paymentService")
public CompletableFuture<String> callPaymentAsync() {
    return CompletableFuture.supplyAsync(
        () -> paymentService.processPayment()
    );
}
```

A time limiter does not necessarily terminate the underlying operation when its timeout expires. Configure the underlying HTTP client's connection and response timeouts as well.

#### `@Bulkhead`

The `@Bulkhead` annotation limits concurrent execution or isolates resources to help prevent one dependency from exhausting the application's resources.

```java
@Bulkhead(name = "paymentService")
public String callPaymentService() {
    return paymentService.processPayment();
}
```

Depending on configuration, Resilience4j can use a semaphore-based or thread-pool-based bulkhead.

**Use case:** Limit the number of concurrent requests to a slow external service.

#### `@RateLimiter`

The `@RateLimiter` annotation limits how frequently an operation can be called within a configured period.

```java
@RateLimiter(name = "paymentService")
public String callPaymentService() {
    return paymentService.processPayment();
}
```

**Use case:** Protect a third-party API with request quotas or control the rate of outgoing calls.

These annotations can be combined when necessary, but the order of the resilience decorators and their interaction should be understood and tested.

### Complete Circuit Breaker Example Using Resilience4j

#### Service Class

```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import org.springframework.stereotype.Service;

@Service
public class PaymentClientService {

    private final PaymentService paymentService;

    public PaymentClientService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @CircuitBreaker(
        name = "paymentService",
        fallbackMethod = "paymentFallback"
    )
    public String callDependency() {
        return paymentService.processPayment();
    }

    public String paymentFallback(Throwable ex) {
        return "Payment service is temporarily unavailable";
    }
}
```

#### Configuration in `application.yml`

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        failureRateThreshold: 50
        waitDurationInOpenState: 5s
        permittedNumberOfCallsInHalfOpenState: 3
```

**Configuration explained:**

| Property | Meaning |
|---|---|
| `slidingWindowType` | Defines whether the failure window is count-based or time-based |
| `slidingWindowSize` | Number of calls or time-window measurements used for evaluation |
| `minimumNumberOfCalls` | Minimum calls required before the failure rate is evaluated |
| `failureRateThreshold` | Failure percentage that can open the circuit |
| `waitDurationInOpenState` | How long the circuit stays open before recovery testing |
| `permittedNumberOfCallsInHalfOpenState` | Number of test calls permitted during recovery |

For this example, the breaker can open when at least five calls have been recorded and the configured failure rate reaches 50%. Actual behavior depends on exception classification and the other Resilience4j settings.

### What Are `@HystrixCommand` and `@HystrixProperty`?

These annotations belong to **Netflix Hystrix**, a legacy circuit breaker library.

#### `@HystrixCommand`

Wraps a method with Hystrix command behavior and optionally specifies a fallback.

```java
@HystrixCommand(fallbackMethod = "fallback")
public String callDependency() {
    return paymentService.processPayment();
}

public String fallback() {
    return "Service is temporarily unavailable";
}
```

#### `@HystrixProperty`

Configures individual Hystrix command or circuit breaker properties.

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    commandProperties = {
        @HystrixProperty(
            name = "execution.isolation.thread.timeoutInMilliseconds",
            value = "2000"
        ),
        @HystrixProperty(
            name = "circuitBreaker.requestVolumeThreshold",
            value = "5"
        ),
        @HystrixProperty(
            name = "circuitBreaker.errorThresholdPercentage",
            value = "50"
        ),
        @HystrixProperty(
            name = "circuitBreaker.sleepWindowInMilliseconds",
            value = "5000"
        )
    }
)
public String callDependency() {
    return paymentService.processPayment();
}
```

The settings configure a 2-second Hystrix command timeout, a request-volume threshold of five, a 50% error threshold, and a 5-second circuit-breaker sleep window.

This is legacy Hystrix syntax, not Resilience4j syntax. Do not mix Hystrix annotations and Resilience4j configuration in the same example as if they were interchangeable.

### Quick Revision

| Annotation | Purpose | Library |
|---|---|---|
| `@CircuitBreaker` | Prevents repeated calls to an unhealthy dependency | Resilience4j |
| `@Retry` | Retries configured failures | Resilience4j |
| `@TimeLimiter` | Applies a timeout to supported asynchronous operations | Resilience4j |
| `@Bulkhead` | Limits concurrent execution or isolates resources | Resilience4j |
| `@RateLimiter` | Limits call frequency | Resilience4j |
| `@HystrixCommand` | Wraps a method in Hystrix command behavior | Legacy Hystrix |
| `@HystrixProperty` | Configures Hystrix command and circuit breaker properties | Legacy Hystrix |

#### Interview Answer

"For a new Spring Boot microservice, I would generally use Resilience4j. The `@CircuitBreaker` annotation protects calls to failing dependencies, `@Retry` handles appropriate transient failures, `@TimeLimiter` limits asynchronous operation duration, `@Bulkhead` isolates resources, and `@RateLimiter` controls request frequency. I would configure the circuit breaker in `application.yml` and combine it with HTTP client timeouts, monitoring, and business-safe fallback handling. `@HystrixCommand` and `@HystrixProperty` are associated with the older Netflix Hystrix library."


---


## Which library have you used to implement Circuit Breaker in Spring Boot?

### Interview Answer

In my Spring Boot microservices, I would use **Resilience4j** to implement the Circuit Breaker pattern. It is lightweight, modular, and integrates well with Spring Boot and Spring Cloud.

Resilience4j helps prevent cascading failures when a downstream service becomes slow or unavailable. It can also be combined with Retry, TimeLimiter, Bulkhead, and RateLimiter to improve fault tolerance.

Previously, Netflix Hystrix was widely used for circuit breaking in Java microservices. However, **Hystrix is in maintenance mode**, so Resilience4j is generally the preferred choice for new Spring Boot applications.

### Popular Circuit Breaker Libraries

| Library | Language / Platform | Description |
|---|---|---|
| Resilience4j | Java | Lightweight fault-tolerance library; commonly used with Spring Boot |
| Netflix Hystrix | Java | Legacy fault-tolerance library, now in maintenance mode |
| Polly | .NET | Provides resilience strategies such as circuit breaking, retries, and timeouts |
| go-hystrix | Go | Go implementation inspired by Netflix Hystrix |
| Ruby Circuit Breaker | Ruby | Circuit breaker implementations for Ruby applications |
| Elixir circuit-breaker libraries | Elixir | Libraries providing circuit breaker functionality |

### Why Resilience4j?

- **Lightweight and modular:** Use only the resilience features your application needs.
- **Spring Boot integration:** Supports annotations and configuration for circuit breakers.
- **Fallback methods:** Return a controlled response when a downstream call fails.
- **Monitoring:** Exposes metrics that can be integrated with Micrometer and monitoring systems.
- **Multiple resilience patterns:** Supports circuit breakers, retries, time limits, bulkheads, and rate limiting.

### Example Using Resilience4j

```java
@Service
public class PaymentService {

    @CircuitBreaker(
        name = "paymentService",
        fallbackMethod = "paymentFallback"
    )
    public String processPayment() {
        return callPaymentProvider();
    }

    private String callPaymentProvider() {
        // Call the external payment service
        return "Payment processed successfully";
    }

    public String paymentFallback(Throwable ex) {
        return "Payment service is temporarily unavailable";
    }
}
```

The `@CircuitBreaker` annotation monitors failures from `processPayment()`. When the circuit is open, calls are short-circuited and directed to the fallback method instead of repeatedly calling the failing service.

The fallback response shown here is illustrative. For actual payment processing, avoid reporting a payment as failed or successful without checking its status; a timeout may leave the payment outcome unknown.

### Important Distinction

- **Spring Cloud Circuit Breaker** provides an abstraction for circuit breaker implementations and can integrate with supported providers, including Resilience4j.
- **Resilience4j** is the underlying resilience library that implements the circuit breaker functionality.
- **Netflix Hystrix** is a legacy option and is not recommended for new projects.

### Quick Interview Revision

**Question:** Which circuit breaker library have you used or would you choose in Spring Boot?

**Answer:** I would choose Resilience4j because it is lightweight, integrates well with Spring Boot, supports annotation-based configuration and fallback methods, and offers additional resilience patterns such as Retry, TimeLimiter, and Bulkhead. Hystrix was popular in older Netflix-based microservices, but Resilience4j is the more appropriate choice for new applications.


---



