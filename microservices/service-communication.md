---
title: Synchronous and Asynchronous Communication
parent: Microservices
nav_order: 4
---

# Synchronous and Asynchronous Communication
{: .no_toc }

REST, gRPC and messaging.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Why Are Microservices Stateless?

Microservices are generally designed to keep application instances stateless so that they can be scaled, replaced, and restarted easily without losing essential business data.

A **stateless service** does not depend on information stored exclusively in its local memory from previous requests. Each request contains the necessary information, or the service retrieves the required state from an external resource such as a database, distributed cache, or message broker.

### Scalability

Stateless microservices can be scaled horizontally by adding more instances without transferring local session state between them.

For example, if an Order Service receives a large number of requests, a load balancer can distribute those requests across multiple instances.

```text
              Load Balancer
                    |
          +---------+---------+
          |         |         |
          v         v         v
      Instance 1 Instance 2 Instance 3
          |         |         |
          +---------+---------+
                    |
                    v
               Order Database
```

Any healthy instance can process a request, provided it has access to the required data and dependencies.

### Resilience and Fault Tolerance

If a stateless instance fails, another healthy instance can process subsequent requests without needing to recover the failed instance's local session state.

For example, if one Spring Boot instance crashes, Kubernetes can replace it, and the load balancer can route traffic to the remaining healthy instances.

However, statelessness alone does not guarantee resilience. Database failures, network problems, and other shared dependencies must also be handled.

### Simplified Load Balancing

A load balancer can route requests to any healthy instance instead of directing users to the specific instance that previously handled their requests.

This reduces the need for sticky sessions, in which requests from a user are repeatedly routed to the same server.

### Portability

Stateless application instances can be deployed across different servers, containers, availability zones, or cloud environments without migrating their local application state.

Environment-specific configuration and external dependencies must still be managed correctly.

### Simplicity and Maintainability

Stateless application instances are easier to restart, test, replace, and deploy because they do not need to preserve essential state in local memory between requests.

This also simplifies rolling deployments and autoscaling.

### Better Resource Utilization

When instances do not retain large amounts of user-specific state in memory, resources can be allocated more flexibly across replicas.

External state stores must still be designed and scaled appropriately.

---

## How Do Stateless Microservices Manage State?

Statelessness does not mean that the application has no state. It means that essential state is not tied exclusively to an individual application instance.

Common approaches include:

- **Database:** Store durable business data such as orders, customers, and payments.
- **Distributed cache:** Store shared, temporary data in systems such as Redis.
- **JWT:** Carry signed claims about a user or client, subject to expiry and validation.
- **Message broker:** Persist and distribute events or messages where appropriate.
- **External session store:** Store session information outside individual application instances.

### Example: Stateless Spring Boot Service

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public Order getOrder(Long orderId) {
        return orderRepository.findById(orderId)
                .orElseThrow(() ->
                    new RuntimeException("Order not found"));
    }
}
```

In this example, the service retrieves the order from a database rather than relying on an instance-level in-memory collection.

Each request can be handled by any instance that has access to the same authorized data.

---

## Stateless vs Stateful Microservices

| Aspect | Stateless Service | Stateful Service |
|---|---|---|
| Application instance | Does not retain essential request state between requests | May retain state locally or manage ongoing stateful interactions |
| Horizontal scaling | Generally simpler | May require session affinity or state coordination |
| Instance failure | Another instance can usually take over | Local state may need recovery or replication |
| Load balancing | Requests can generally go to any healthy instance | Routing may depend on where state resides |
| Deployment | Easier to replace instances | Requires careful handling of state and ownership |
| Example | REST API retrieving orders from a database | WebSocket connection manager maintaining live connection state |

**Important:** Stateful services are not inherently bad. Some workloads, such as WebSocket connections, stream processing, and certain long-running workflows, require stateful behavior. Such systems can still be made scalable and resilient through appropriate state management, partitioning, replication, and recovery mechanisms.

---

## Are All Microservices Stateless?

No. Statelessness is a recommended design approach for application instances, not an absolute requirement for every microservice.

For example:

- An authentication service may use an external session store.
- A payment service maintains durable payment records in a database.
- A WebSocket service may maintain active connection state.
- A Kafka consumer may maintain partition assignments and processing offsets.

The key distinction is between **business state**, which must be preserved appropriately, and **instance-local runtime state**, which should not become an unnecessary obstacle to scaling or recovery.

### Interview Answer

Microservices are generally designed to be stateless because stateless instances can be scaled horizontally, load-balanced, restarted, and replaced more easily. They do not depend on local session data from previous requests, so any healthy instance can handle an incoming request.

Essential state is stored in external systems such as databases, distributed caches, or message brokers. This improves scalability and simplifies deployments, but resilience still requires handling failures in shared dependencies. Some services, such as WebSocket or stream-processing services, may legitimately maintain state.

---


## How to Share a Database with Multiple Microservices?


In a microservices architecture, multiple services can connect to the same database server or database instance. However, **sharing database tables and allowing multiple services to modify the same data can create tight coupling** and reduce service independence.

The recommended approach is for each microservice to own its business data and expose it through APIs or events. Sharing a database can still be appropriate in legacy systems, transitional architectures, or systems with specific operational constraints.

### Design the Database Schema Around Service Boundaries

Organize tables and schemas according to business domains and service responsibilities.

For example, an e-commerce application might have the following structure:

```text
             PostgreSQL Server
                    |
          +---------+---------+
          |         |         |
          v         v         v
      User Schema Order Schema Payment Schema
          |         |         |
          v         v         v
      User Service Order Service Payment Service
```

This diagram illustrates logical separation within a shared database server. Separate schemas do not automatically guarantee independent ownership or eliminate all coupling.

Avoid allowing one service to modify another service's tables directly.

### Use a Database Migration Tool

Use tools such as Flyway or Liquibase to manage database schema changes in a controlled and repeatable manner.

For example, if the Order Service needs a new column, its migration should be planned so that currently running application versions remain compatible during deployment.

Recommended practices include:

- Version-control migration scripts.
- Apply migrations through controlled deployment pipelines.
- Prefer backward-compatible changes.
- Use an expand-and-contract strategy for breaking schema changes.
- Coordinate ownership when multiple services use the same database instance.

### Implement a Database Access Layer

Use repositories and data-access abstractions to separate persistence logic from business logic.

For example, Spring Boot services can use Spring Data JPA repositories to interact with their owned tables.

However, creating a shared library or centralized database-access service that exposes every table to every microservice can increase coupling. Each service should retain control over its business rules and data access.

### Use a Shared Database Instance When Necessary

A shared database server can reduce initial infrastructure costs and simplify administration.

For example, multiple services might use the same PostgreSQL server while having separate databases, schemas, users, or permissions.

The key is to establish clear ownership boundaries and avoid uncontrolled cross-service access.

### Implement Data Isolation and Access Control

Ensure that each microservice has only the permissions it needs.

Possible approaches include:

- Separate database users for each service.
- Separate schemas for different service domains.
- Separate databases where stronger isolation is required.
- Row-level security where appropriate.
- Network restrictions and encrypted connections.
- Auditing and monitoring database access.

Database permissions should enforce the intended ownership model rather than relying solely on application developers to follow conventions.

### Monitor Database Performance

When multiple services share a database, they also compete for database resources.

Monitor:

- Query execution time.
- Connection pool usage.
- CPU and memory consumption.
- Lock contention and deadlocks.
- Slow queries and missing indexes.
- Transaction duration.
- Connection limits and replication lag.

A resource-intensive query from one service can affect the performance of other services sharing the same database infrastructure.

### Consider Security and Regulatory Requirements

Applications handling financial, healthcare, or other sensitive data may require strict access controls, audit trails, encryption, and data segregation.

A shared database is not automatically prohibited, but the architecture must satisfy the applicable security and regulatory requirements.

### Example: Multiple Spring Boot Services Sharing PostgreSQL

Suppose User Service and Order Service connect to the same PostgreSQL server.

Each service should ideally use its own database user and access only the tables it owns.

**User Service configuration:**

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${USER_DB_USERNAME}
    password: ${USER_DB_PASSWORD}
```

**Order Service configuration:**

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${ORDER_DB_USERNAME}
    password: ${ORDER_DB_PASSWORD}
```

The connection URL can point to the same PostgreSQL server, while database roles and grants restrict each service to its authorized objects.

The credentials should be supplied securely through environment configuration or a secret manager, not committed to source control.

For example:

- User Service can read and update user tables.
- Order Service can read and update order tables.
- Order Service retrieves user information through a User Service API rather than directly updating user records.

This approach allows shared database infrastructure while preserving logical data ownership.

### Why Is Sharing a Database Discouraged in Microservices?

Sharing tables across services can introduce several problems.

**Tight coupling:** A schema change made for one service can break another service.

**Independent deployment difficulties:** Multiple services may need coordinated releases when they depend on the same table structure.

**Data ownership conflicts:** Different services may implement competing business rules for the same records.

**Performance contention:** Expensive queries and long-running transactions can affect multiple services.

**Security risks:** Broad database permissions can allow a compromised service to access data outside its intended responsibility.

**Scaling limitations:** Services may need different database configurations or scaling strategies, but a shared database can constrain those choices.


### Recommended Approach: Database per Service

The database-per-service pattern gives each microservice exclusive ownership of its business data.

```text
             +----------------+
             |  User Service  |
             +----------------+
                     |
                     v
                +----------+
                |  User DB |
                +----------+

             +----------------+
             | Order Service  |
             +----------------+
                     |
                     v
                +----------+
                | Order DB |
                +----------+

             +----------------+
             | Payment Service|
             +----------------+
                     |
                     v
               +-------------+
               |  Payment DB |
               +-------------+
```

Each service owns its database and exposes the data required by other services through APIs or events.

The databases do not necessarily need separate physical servers. They can share a database server or managed database cluster while maintaining appropriate logical isolation.

### How Do Services Share Data Without Sharing Tables?

Common approaches include:

- **REST APIs:** A service requests data from the service that owns it.
- **gRPC:** Services communicate through defined remote procedure contracts.
- **Event-driven communication:** Services publish and consume events through Kafka or RabbitMQ.
- **Local read models:** A service maintains a local copy of selected data for querying, updated through events.
- **CQRS:** Separate models can be used for commands and queries when the application's complexity justifies it.

For example, when an order is created, the Order Service can publish an `OrderCreated` event. The Notification Service consumes the event and sends a confirmation without directly accessing the Order Service's database.

### Shared Database vs. Database per Service

| Aspect | Shared Database | Database per Service |
|---|---|---|
| Infrastructure | Simpler initially | May require more setup |
| Data ownership | Can become ambiguous | Clearer service ownership |
| Schema changes | Can affect multiple services | More isolated |
| Independent deployment | Harder if tables are shared | Generally easier |
| Data consistency | Easier for shared local transactions | Often requires eventual consistency |
| Cross-service queries | May be easier initially | May require APIs or read models |
| Security isolation | Requires careful permissions | Can provide stronger logical isolation |
| Scaling | Shared resource can become a bottleneck | Database resources can be tailored per service |


### Interview Questions

**Can multiple microservices use the same database?**

Yes. Multiple services can use the same database server or instance. However, sharing tables and directly modifying each other's data creates tight coupling. Clear data ownership and access restrictions are essential.

**Is a separate physical database server required for every microservice?**

No. Services can have logically separate databases or schemas on the same database server. The key principle is independent data ownership, not necessarily separate physical infrastructure.

**How do you maintain data consistency when each microservice has its own database?**

Use patterns such as Saga, transactional outbox, idempotent consumers, and event-driven communication. Depending on the workflow, eventual consistency may be acceptable.

**Can we use joins across microservices' databases?**

Direct cross-service database joins are generally discouraged because they couple services to each other's schemas and database availability. Use APIs, events, or purpose-built read models instead.

**When would you accept a shared database?**

A shared database can be a reasonable transitional choice for a legacy system, a small team, or an application with operational constraints. Enforce clear ownership, least-privilege access, and controlled schema changes, and reassess the architecture as requirements evolve.

### Key Takeaway

A shared database instance is not inherently incompatible with microservices. The bigger architectural concern is **shared ownership of data and direct dependencies on other services' tables**.

Prefer database-per-service ownership when independent deployment, scalability, and autonomy are important. Where sharing is necessary, isolate permissions, define schema ownership, manage migrations carefully, and monitor the impact of shared resources.


---



## How to Connect Internal and External Services in Microservices?

In a microservices architecture, services communicate with external clients and with other internal services. The communication mechanism depends on security requirements, latency, reliability, scalability, and whether the interaction needs an immediate response.

The two primary communication styles are:

- **Synchronous communication:** One service sends a request and waits for a response, such as REST or gRPC.
- **Asynchronous communication:** A service publishes a message or event that another service processes independently, such as Kafka or RabbitMQ.

A production system commonly combines both approaches.

---

### Internal vs External Service Communication

#### External Communication

External communication occurs between clients outside the backend system and the microservices platform.

Examples include:

- A web application calling a backend API.
- A mobile application retrieving customer details.
- A third-party payment provider sending a webhook.
- An external partner integrating with an order API.

An **API Gateway** is commonly used as the entry point for external client requests.

#### Internal Communication

Internal communication occurs between services within the backend system.

Examples include:

- Order Service calling Inventory Service.
- Order Service requesting payment processing.
- User Service publishing a user-registration event.
- Notification Service consuming an order-created event.

Internal communication can use REST, gRPC, messaging systems, or a combination of these.

#### Architecture Diagram

```text
                  External Clients
                         |
                         v
                  +--------------+
                  | API Gateway  |
                  +--------------+
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
        +---------+ +---------+ +----------+
        |  User   | |  Order  | | Product  |
        | Service | | Service | | Service  |
        +---------+ +---------+ +----------+
             |           |
             |           +-------> Inventory Service
             |
             +-------------------> Kafka / RabbitMQ
                                          |
                                          v
                                  Notification Service
```

This diagram illustrates one possible architecture. A gateway is not required for every internal call, and asynchronous messaging may be used for many workflows.


#### API Gateway Pattern

An API Gateway acts as a common entry point for external requests and routes them to the appropriate backend services.

Instead of exposing every microservice directly to clients, the system can expose a single public API endpoint.

#### Responsibilities

- Request routing.
- Authentication integration.
- Rate limiting and throttling.
- TLS termination.
- Request and response transformation.
- API versioning.
- Request logging and correlation IDs.
- Caching where appropriate.

Example

```text
Mobile App
    |
    v
API Gateway
    |
    +----> User Service
    |
    +----> Order Service
    |
    +----> Product Service
```

#### Common technologies

- Spring Cloud Gateway.
- Kong.
- NGINX.
- Cloud-managed API gateways.

**Important:** The API Gateway should not become a centralized implementation of all business logic. Backend services must still enforce their own authorization and business rules.


#### Service Discovery

Service discovery allows a service or client to locate the current network address of another service without hardcoding individual instance addresses.

This is useful when services run multiple replicas and instances are created, replaced, or removed dynamically.

#### How It Works

```text
Order Service
      |
      v
Service Discovery
      |
      v
Inventory Service Instances
   |          |          |
   v          v          v
Instance 1  Instance 2  Instance 3
```

Depending on the implementation, instances register with a registry, or the platform provides service addresses through DNS and orchestration.

#### Types of service discovery

**Client-side discovery:** The calling service queries a registry and selects an instance.

**Server-side discovery:** A proxy, load balancer, or platform routes requests to a healthy instance.

#### Common technologies

- Netflix Eureka.
- Consul.
- Kubernetes Services and DNS.
- Cloud-native service discovery.

#### Spring Boot example

A service may call another service using its logical service name:

```java
@FeignClient(name = "inventory-service")
public interface InventoryClient {

    @GetMapping("/api/inventory/{productId}")
    InventoryResponse getInventory(
        @PathVariable("productId") Long productId
    );
}
```

The logical name is resolved through the configured discovery or load-balancing mechanism. The annotation alone does not configure service discovery; the necessary Spring Cloud dependencies and configuration must also be present.

**Interview point:** Service discovery removes the need to hardcode dynamically changing instance addresses.


### Load Balancer

A load balancer distributes incoming requests across multiple instances of a service.

#### Why is it needed?

- Distributes traffic across replicas.
- Helps avoid overloading a single instance.
- Supports horizontal scaling.
- Can route requests away from unhealthy instances when health information is available.

Example

```text
                 Load Balancer
                       |
             +---------+---------+
             |         |         |
             v         v         v
          Instance 1 Instance 2 Instance 3
```

Load balancing can be implemented by an external load balancer, an API gateway, a service mesh, or a client-side load-balancing library.

In Kubernetes, a Service can provide a stable endpoint and distribute traffic to eligible pods.

**Important:** Load balancing does not automatically provide fault tolerance. Health checks, timeouts, and suitable failure-handling mechanisms are also required.

### Synchronous Communication: REST and gRPC

In synchronous communication, the calling service waits for a response from the receiving service.

#### REST

REST APIs commonly use HTTP and JSON.

Example:

```text
Order Service
      |
      | GET /api/inventory/101
      v
Inventory Service
      |
      | 200 OK
      v
Available Stock
```

Use REST when services need straightforward request-response communication and interoperability.

#### gRPC

gRPC uses Protocol Buffers and supports strongly typed service contracts and multiple communication patterns.

It can be useful for efficient service-to-service communication, particularly in systems with high request volumes or strict latency requirements.

#### Challenges of synchronous communication

- Network latency.
- Timeouts.
- Dependency failures.
- Cascading failures.
- Increased coupling between request lifecycles.

Use explicit timeouts and avoid unbounded retries.

### Asynchronous Communication: Event-Driven Architecture

In asynchronous communication, services exchange messages or events without requiring the sender to wait for the receiver to finish processing.

Common technologies include Apache Kafka and RabbitMQ.

#### Example: Order Created Event

```text
Order Service
      |
      | Publish OrderCreated
      v
 Message Broker
      |
      +--------> Inventory Consumer
      |
      +--------> Notification Consumer
      |
      +--------> Analytics Consumer
```

The Order Service publishes an event. Other services consume it independently according to their responsibilities.

#### Benefits

- Reduces direct runtime dependencies.
- Supports independent processing.
- Helps absorb temporary traffic spikes.
- Allows multiple consumers to react to the same event.
- Supports eventual consistency.

#### Challenges

- Duplicate message delivery.
- Event ordering.
- Delayed processing.
- Schema compatibility.
- Dead-letter handling.
- Eventual consistency.

Consumers should be designed to handle duplicate deliveries safely. A message broker alone does not guarantee end-to-end exactly-once business effects.


### Circuit Breaker Pattern

A circuit breaker protects a service from repeatedly calling a dependency that is failing or responding too slowly.

It is not primarily a mechanism for automatically switching to another healthy instance. Instance selection and load balancing are separate responsibilities.

#### Circuit breaker states

- **Closed:** Calls are allowed, and failures are monitored.
- **Open:** Calls are rejected or handled through a fallback without repeatedly invoking the failing dependency.
- **Half-open:** A limited number of test calls determine whether the dependency has recovered.

Example

```text
Order Service
      |
      v
Circuit Breaker
      |
      v
Payment Service
```

If Payment Service repeatedly fails, the circuit breaker may open temporarily. Order Service can then fail fast or return an appropriate response rather than allowing requests to accumulate.

#### Common technology

Resilience4j is commonly used with Spring Boot to implement circuit breakers, retries, rate limiters, and bulkheads.

**Interview point:** Use timeouts with circuit breakers. A circuit breaker cannot prevent long waits if remote calls have no suitable timeout.


### Secure Internal and External Communication

Communication between services must be secured regardless of whether the services are publicly accessible.

#### External communication

- Use HTTPS/TLS.
- Authenticate clients using appropriate mechanisms such as OAuth 2.0 or OpenID Connect.
- Apply authorization and rate limits.
- Validate request data.
- Protect credentials and API keys.
- Verify third-party webhook signatures where supported.

#### Internal communication

- Authenticate service identities.
- Apply least-privilege authorization.
- Use TLS or mutual TLS where appropriate.
- Store credentials in a secret manager.
- Restrict network access.
- Avoid trusting a request solely because it originated inside the network.

An API Gateway can enforce shared controls, but it should not be the only layer responsible for application authorization.


### Internal vs External Communication: Comparison

| Aspect | External Communication | Internal Communication |
|---|---|---|
| Participants | Clients, partners, third parties | Backend microservices |
| Common entry point | API Gateway or public load balancer | Service DNS, discovery, proxy, or broker |
| Protocols | HTTPS, REST, webhooks, WebSocket | REST, gRPC, Kafka, RabbitMQ |
| Security | Client authentication, authorization, TLS | Service identity, authorization, TLS |
| Traffic controls | Public rate limiting and abuse protection | Capacity limits, quotas, and backpressure |
| Reliability | Timeouts, retries, API error handling | Timeouts, retries, circuit breakers, messaging recovery |
| Observability | API access logs and traces | Distributed traces, metrics, logs, message monitoring |


### How to Choose the Right Communication Pattern

| Requirement | Suitable approach |
|---|---|
| Expose APIs to web or mobile clients | API Gateway + REST |
| Find dynamically changing service instances | Service discovery or platform DNS |
| Distribute requests across replicas | Load balancer |
| Obtain an immediate response from another service | REST or gRPC |
| Process work independently | Kafka or RabbitMQ |
| Notify multiple consumers of a business event | Event-driven messaging |
| Prevent repeated calls to an unhealthy dependency | Circuit breaker |
| Protect services from excessive traffic | Rate limiting and backpressure |
| Track a request across multiple services | Distributed tracing |
| Secure service-to-service communication | Service identity, authorization, TLS/mTLS |

These patterns are complementary. A single architecture can use an API Gateway for external traffic, REST for synchronous internal requests, Kafka for asynchronous events, and Resilience4j for resilience.


### Interview Questions

**How do microservices communicate with each other?**

They commonly communicate through synchronous protocols such as REST and gRPC or asynchronous messaging systems such as Kafka and RabbitMQ.

**What is the difference between an API Gateway and service discovery?**

An API Gateway is an entry point that routes and may apply common policies to client requests. Service discovery helps locate service instances dynamically.

**What is the difference between a load balancer and a circuit breaker?**

A load balancer distributes requests across instances. A circuit breaker temporarily stops calls to a failing dependency to reduce cascading failures.

**Should every internal service call go through the API Gateway?**

No. Internal service-to-service communication can use service discovery, internal load balancing, or a service mesh. Routing every internal request through the external gateway may add unnecessary latency and create a bottleneck.

**When should you use Kafka instead of REST?**

Use Kafka when services need asynchronous event processing, independent consumers, or buffering of workloads. REST is often more suitable when the caller needs an immediate response.

**How do you handle a downstream service failure?**

Configure timeouts, appropriate retries with backoff, circuit breakers, and bulkheads. Use fallbacks only when they preserve correct business behavior, and monitor failures through logs, metrics, and traces.

**How do you prevent duplicate processing in event-driven microservices?**

Use idempotent consumers, stable event identifiers, durable deduplication where needed, and appropriate transaction boundaries.

**How do you trace a request across multiple microservices?**

Propagate trace context and correlation identifiers through HTTP requests and messaging metadata, and use distributed tracing tools to visualize the complete request path.

### Quick Revision Notes

- **API Gateway:** Common entry point for external API traffic.
- **Service discovery:** Locates service instances dynamically.
- **Load balancer:** Distributes requests across service instances.
- **REST:** Synchronous HTTP request-response communication.
- **gRPC:** Contract-based remote procedure calls using Protocol Buffers.
- **Kafka/RabbitMQ:** Asynchronous messaging.
- **Circuit breaker:** Reduces repeated calls to failing dependencies.
- **Timeout:** Limits how long a caller waits for a response.
- **Retry:** Repeats a failed operation under controlled conditions.
- **Bulkhead:** Limits resource exhaustion across independent operations.
- **Service mesh:** Can provide infrastructure-level routing, identity, encryption, and observability.
- **Distributed tracing:** Tracks requests across service boundaries.

### Final Recommendation

For a typical Java and Spring Boot microservices system, use an API Gateway for external clients, service discovery or platform DNS for internal routing, REST or gRPC for synchronous requests, and Kafka or RabbitMQ for asynchronous workflows.

Add secure service identity, bounded timeouts, suitable circuit breakers, centralized observability, and idempotent message processing to make the architecture production-ready.


---


## How do two microservices communicate with each other?

Two microservices can communicate using **synchronous communication** or **asynchronous communication**. The choice depends on the business requirement, response-time expectations, reliability, and coupling between services.

The most common approaches are:

1. **REST APIs over HTTP:** One microservice calls another using HTTP requests and receives a response.
2. **Message queues:** Services exchange messages through brokers such as RabbitMQ or Apache Kafka.
3. **Event-driven communication:** A service publishes an event, and interested services consume it independently.
4. **gRPC:** Services communicate using a high-performance remote procedure call framework, often with Protocol Buffers.

In Spring Boot microservices, I would typically use **REST APIs for synchronous request-response operations** and **Kafka or RabbitMQ for asynchronous communication** when services need to process events independently.

Regardless of the approach, communication should be secured and designed to handle failures, timeouts, retries, and duplicate messages.

### 1. REST API Communication

<text color="secondary" size="sm">Type: Synchronous</text>

One service directly invokes another service's REST endpoint using HTTP.

Example: An Order Service calls the Inventory Service to check stock availability before creating an order.

```text
Client
  |
  v
Order Service
  |
  | HTTP GET /api/inventory/101
  v
Inventory Service
  |
  | Returns stock availability
  v
Order Service
```

**Spring Boot example**

Inventory Service:

```java
@RestController
@RequestMapping("/api/inventory")
public class InventoryController {

    @GetMapping("/{productId}")
    public String checkStock(
            @PathVariable Long productId) {
        return "Product " + productId + " is in stock";
    }
}
```

Order Service using `RestClient`:

```java
@Service
public class OrderService {

    private final RestClient restClient;

    public OrderService(RestClient.Builder builder) {
        this.restClient = builder
                .baseUrl("http://inventory-service")
                .build();
    }

    public String checkInventory(Long productId) {
        return restClient.get()
                .uri("/api/inventory/{id}", productId)
                .retrieve()
                .body(String.class);
    }
}
```

This example assumes the hostname `inventory-service` is resolvable in the deployment environment. In production, the base URL can come from configuration or service discovery.

**Advantages**
- Simple to implement and debug.
- Immediate response from the called service.
- Suitable for queries and operations that require an immediate result.

**Disadvantages**
- The caller depends on the downstream service being available.
- Slow downstream calls can increase latency.
- Failures can propagate across multiple services.

### 2. Message Queue Communication

<text color="secondary" size="sm">Type: Asynchronous</text>

Services communicate through a message broker instead of calling each other directly. The sender publishes a message, and the receiver processes it independently.

Common technologies include:
- RabbitMQ
- Apache Kafka
- Amazon SQS

```text
Order Service
     |
     | Publish message
     v
Message Broker
     |
     | Consume message
     v
Notification Service
```

For example, after an order is created, the Order Service publishes an `OrderCreated` message. The Notification Service consumes it and sends an email without making the order request wait for the email to be sent.

**Advantages**
- Reduces direct dependencies between services.
- Supports asynchronous processing and buffering.
- Helps services operate independently during temporary slowdowns.

**Disadvantages**
- Requires broker infrastructure and monitoring.
- Messages may be delivered more than once.
- Debugging and tracing asynchronous flows can be more complex.

### 3. Event-Driven Architecture

In an event-driven architecture, a service publishes an event describing something that has already happened. Other services subscribe to the event and react to it.

Example: When an order is placed, multiple services can independently respond to the same event.

```text
                  Order Service
                       |
                       | OrderCreated event
                       v
                 Kafka / Event Bus
                  /      |       \
                 v       v        v
          Inventory   Payment  Notification
           Service    Service    Service
```

For example:
- Inventory Service reserves stock.
- Payment Service initiates payment processing.
- Notification Service sends an order confirmation.

The event producer does not need to know which services consume the event.

**Important:** Event-driven architecture is a communication style; Kafka and RabbitMQ are technologies that can help implement it. A message queue and an event-driven architecture are related, but they are not exactly the same concept.

### 4. gRPC Communication

gRPC is a remote procedure call framework that commonly uses HTTP/2 and Protocol Buffers for efficient communication.

```text
Order Service
      |
      | gRPC request
      v
Inventory Service
      |
      | gRPC response
      v
Order Service
```

**Advantages**
- Efficient binary serialization.
- Strongly typed service contracts.
- Supports unary and streaming communication patterns.

**Disadvantages**
- Requires service definitions and generated client/server code.
- Browser-facing integration can require additional tooling, such as gRPC-Web.
- Can be less convenient than REST for simple public APIs.

### Comparison of Communication Methods

| Feature | REST | Message Queue / Events | gRPC |
|---|---|---|---|
| Communication | Usually synchronous | Usually asynchronous | Usually synchronous; streaming also supported |
| Protocol | HTTP | Broker-specific protocols | HTTP/2 in typical deployments |
| Data format | Commonly JSON | Depends on message schema | Commonly Protocol Buffers |
| Response | Usually immediate | Usually processed later | Usually immediate for unary calls |
| Best suited for | Queries and direct operations | Background processing and event propagation | Efficient internal service-to-service RPC |
| Example technology | Spring `RestClient` | Kafka, RabbitMQ | gRPC Java |

### How do you choose the right approach?

- Use **REST** when the caller needs an immediate response, such as retrieving customer details.
- Use **Kafka or RabbitMQ** when a task can be processed asynchronously, such as sending notifications or propagating business events.
- Use **gRPC** when strongly typed, efficient internal RPC or streaming is beneficial.
- Use **event-driven architecture** when multiple services need to react independently to business events.

### Important Production Considerations

Regardless of the communication mechanism:

- **Security:** Use TLS where appropriate, authentication, and authorization between services.
- **Timeouts:** Set sensible request timeouts for synchronous calls.
- **Resilience:** Use circuit breakers, bounded retries, and bulkheads where appropriate.
- **Idempotency:** Ensure retrying a request or processing a duplicate message does not cause unintended side effects.
- **Observability:** Use correlation IDs, distributed tracing, metrics, and structured logs.
- **Reliability:** For event publishing, consider the transactional outbox pattern; for consumers, use acknowledgments, retry policies, and dead-letter handling as appropriate.

### Quick Interview Revision

**Question:** How do two microservices communicate with each other?

**Answer:** Microservices communicate synchronously through REST APIs or gRPC, or asynchronously through message brokers such as Kafka and RabbitMQ. I would use REST when an immediate response is required and asynchronous messaging when services need to be loosely coupled and process work independently. In production, I would also implement appropriate security, timeouts, fault tolerance, observability, and idempotency.

---



## How do you restrict one microservice from calling other microservices?

### Interview Question

Suppose we have five microservices: A, B, C, D, and E. We want to allow Service A to communicate with Service B but prevent it from calling Services C, D, and E. How would you implement this restriction?

### Interview Answer

I would implement **defense in depth using service-to-service authentication, authorization policies, and network-level restrictions**.

For example, if Service A is allowed to call only Service B, I would:

1. **Implement service-level authorization:** Give Service A an identity and configure authorization policies so it can invoke only the permitted APIs of Service B.
2. **Use an API Gateway where appropriate:** If internal traffic is routed through a gateway, configure policies to restrict which services can access which endpoints. However, a gateway alone is insufficient if services can bypass it.
3. **Apply network policies:** Use Kubernetes NetworkPolicies, firewall rules, or service-mesh authorization policies to block Service A from reaching Services C, D, and E.
4. **Enforce authorization in the target services:** Services C, D, and E must reject requests from Service A even if it somehow reaches their endpoints.

This combination provides multiple layers of protection instead of relying on a single access-control mechanism.

### 1. Understand the Required Access

<box border radius="lg" padding={3} gap={3}>
  <box background="surface-secondary" radius="md" padding={3} align="center" gap={1}>
    <icon name="server" size="xl" />
    **Service A**
    <text color="secondary" size="sm">Caller</text>
  </box>
  <box align="center" gap={1}>
    <icon name="arrow-down" size="lg" color="success" />
    <text color="success" weight="medium" size="sm">Allowed</text>
  </box>
  <box border={{size:1,color:"#16A34A"}} radius="md" padding={3} align="center" gap={1}>
    **Service B**
    <text color="secondary" size="sm">Authorized destination</text>
  </box>
  <divider />
  <text color="danger" weight="medium">Blocked destinations</text>
  <grid columns={3} gap={2}>
    <grid-item>
      <box border={{size:1,color:"#DC2626"}} radius="md" padding={2} align="center" gap={1}>
        <icon name="shield-ban" color="danger" size="lg" />
        **Service C**
        <text color="danger" size="xs">Denied</text>
      </box>
    </grid-item>
    <grid-item>
      <box border={{size:1,color:" #DC2626"}} radius="md" padding={2} align="center" gap={1}>
        <icon name="shield-ban" color="danger" size="lg" />
        **Service D**
        <text color="danger" size="xs">Denied</text>
      </box>
    </grid-item>
    <grid-item>
      <box border={{size:1,color:"#DC2626"}} radius="md" padding={2} align="center" gap={1}>
        <icon name="shield-ban" color="danger" size="lg" />
        **Service E**
        <text color="danger" size="xs">Denied</text>
      </box>
    </grid-item>
  </grid>
</box>

The goal is not merely to hide the URLs of Services C, D, and E. Service A must be technically unable or unauthorized to access them.

### 2. Approach 1: Authentication and Authorization

Each microservice should have its own identity. When Service A calls another service, it presents a verifiable credential, such as an OAuth 2.0 access token.

The receiving service validates the caller's identity and checks its permissions.

Example access policy:

| Caller | Destination | Permission |
|---|---|---|
| Service A | Service B | Allow |
| Service A | Service C | Deny |
| Service A | Service D | Deny |
| Service A | Service E | Deny |

With OAuth 2.0, the token can contain scopes or other authorization claims. The target service must validate those claims and enforce the policy. An API key alone is generally weaker because it does not inherently provide fine-grained authorization.

**Spring Boot implementation concept**

A service can validate JWT access tokens using Spring Security's OAuth 2.0 Resource Server support. Authorization rules should then restrict access based on trusted scopes or claims representing the calling service.

Illustrative configuration for a target service:

```java
@Bean
SecurityFilterChain securityFilterChain(
        HttpSecurity http) throws Exception {

    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/internal/**")
            .hasAuthority("SCOPE_inventory.read")
        .anyRequest().authenticated()
    );

    http.oauth2ResourceServer(oauth ->
        oauth.jwt(Customizer.withDefaults())
    );

    return http.build();
}
```

This example checks a permission named `inventory.read`. It does **not** by itself prove that the caller is Service A or Service B. For a strict service-to-service policy, validate a trusted service identity claim as well, or use a dedicated authorization mechanism. Also configure the JWT issuer and other validation settings for your environment.

### 3. Approach 2: API Gateway

An API Gateway can enforce access rules when service traffic passes through it.

```text
Service A
    |
    v
API Gateway
    |
    +----> Service B  (Allowed)
    |
    +----X Service C  (Denied)
    |
    +----X Service D  (Denied)
    |
    +----X Service E  (Denied)
```

For example, Spring Cloud Gateway can apply authentication and authorization filters before forwarding a request.

However, **an API Gateway is not sufficient on its own for internal microservice security**. If Service A can directly access Service C over the internal network, it might bypass the gateway. Restrict direct network access or enforce authorization at the destination as well.

### 4. Approach 3: Kubernetes NetworkPolicy

If the microservices run in Kubernetes, NetworkPolicy can restrict which workloads can communicate over the network.

For example, the policy for Service A can allow egress only to Service B and explicitly required infrastructure, such as DNS. Other traffic can be denied by default.

Conceptual policy:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-service-a
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: service-a
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: service-b
      ports:
        - protocol: TCP
          port: 8080
```

**Important:** This is an illustrative starting point, not a complete production policy. It assumes Service B's destination pods are selected correctly and listen on TCP port 8080. You may also need to allow DNS and other required destinations, and account for namespace boundaries and your cluster's networking implementation. Kubernetes NetworkPolicy enforcement depends on a supporting network plugin.

Network policies operate at the network layer. They do not replace authentication and authorization inside the application.

### 5. Approach 4: Service Mesh

A service mesh such as Istio can enforce workload identity and service-to-service authorization centrally, often using mutual TLS (mTLS).

You could define a policy allowing Service A to access Service B while denying access to Services C, D, and E.

This approach is especially useful when an organization has many microservices and wants consistent security policies without implementing all communication controls separately in each application.

### Which approach should you choose?

| Environment | Recommended controls |
|---|---|
| Basic Spring Boot application | Spring Security and explicit authorization |
| Services exposed through a gateway | Gateway authorization plus protection against direct access |
| Kubernetes | NetworkPolicy plus application-level authorization |
| Large microservices platform | Service mesh authorization, workload identity, and network controls |

### Quick Interview Revision

**Question:** How would you prevent Service A from calling Services C, D, and E while allowing it to call Service B?

**Answer:** I would use a least-privilege approach. I would assign Service A a unique identity, allow only the required permissions for Service B, and deny its access to Services C, D, and E. At the infrastructure layer, I would enforce Kubernetes NetworkPolicies or service-mesh authorization rules. I would also secure the target services using Spring Security and validate service identities and permissions. An API Gateway can provide another layer of protection, but I would not rely on it alone because internal services might bypass it.


---



## How do you securely store usernames and passwords in a Spring Boot microservices application?

{: .no_toc }

### Interview Answer

The best practice is to **never hardcode credentials in the source code or commit them to Git**. Sensitive information such as database usernames, passwords, API keys, and service credentials should be stored in a secure secrets-management system and retrieved at runtime.

For a production Spring Boot microservices application, I would choose a solution based on the deployment environment:

- **AWS:** AWS Secrets Manager.
- **Google Cloud:** Google Cloud Secret Manager.
- **Azure:** Azure Key Vault.
- **Cloud-independent infrastructure:** HashiCorp Vault.
- **Local development:** Environment variables or an uncommitted local configuration file.

I would also enforce least-privilege access, encryption in transit and at rest, credential rotation, and appropriate audit logging.

### 1. Different Ways to Store Credentials

| Approach | Suitable for | Recommendation |
|---|---|---|
| Environment variables | Local development and simpler deployments | Useful, but access to the process environment must be protected |
| External configuration files | Configuration that is not sensitive | Do not store plaintext passwords in files |
| Encrypted configuration | Environments with controlled encryption and key management | Ensure encryption keys are stored separately |
| HashiCorp Vault | Multi-cloud or self-managed environments | Strong option for centralized secrets management |
| AWS Secrets Manager | AWS-hosted applications | Recommended for AWS deployments |
| Google Cloud Secret Manager | Google Cloud applications | Recommended for GCP deployments |
| Azure Key Vault | Azure applications | Recommended for Azure deployments |
| Kubernetes Secrets | Kubernetes workloads | Use appropriate encryption, RBAC, and external secret management where needed |

**Important distinction:** Google Cloud KMS is primarily a key-management and cryptographic service; Google Cloud Secret Manager is the service intended for storing and managing application secrets.

### 2. Example: Using Environment Variables in Spring Boot

Instead of hardcoding a database password in `application.properties`:

```properties
spring.datasource.username=admin
spring.datasource.password=mySecretPassword123
```

Use environment-variable placeholders:

```properties
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

Set the variables in the deployment environment:

```bash
export DB_USERNAME='app_user'
export DB_PASSWORD='your-secret-password'
```

Spring Boot resolves the placeholders at startup.

This avoids committing the actual credentials to source control. However, environment variables are not automatically secure: they may be exposed through process inspection, diagnostic dumps, or misconfigured deployment systems. For production, inject them through a controlled secrets-management mechanism.

### 3. Example: Using Google Cloud Secret Manager

Suppose a Spring Boot application is deployed on Google Cloud and needs database credentials.

The recommended flow is:

<box border radius="lg" padding={3} align="center" gap={2}>
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **Google Cloud Secret Manager**
    <text color="secondary" size="sm">Stores database username and password</text>
  </box>
  <icon name="arrow-down" size="lg" />
  <box border radius="md" padding={3} width="100%" align="center">
    **Workload Identity / Service Account**
    <text color="secondary" size="sm">Authorizes access to specific secrets</text>
  </box>
  <icon name="arrow-down" size="lg" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **Spring Boot Microservice**
    <text color="secondary" size="sm">Retrieves credentials when needed</text>
  </box>
</box>

Implementation steps:

1. Store the database credentials in Google Cloud Secret Manager.
2. Assign the application workload a dedicated service account or workload identity.
3. Grant that identity permission to access only the required secrets.
4. Configure the application to retrieve secrets using the official Google Cloud client or an appropriate Spring integration.
5. Avoid logging secret values, and establish a rotation procedure.

The service should not use a broadly privileged service account simply to make secret retrieval easier.

### 4. Example: Using HashiCorp Vault

HashiCorp Vault provides centralized secret storage, access control, auditing, and secret lifecycle management. It can also generate dynamic database credentials in supported configurations.

For example, the secret might be stored at a Vault path such as:

```text
secret/data/payment-service/database
```

The application authenticates to Vault using an appropriate workload identity or authentication method, then retrieves only the secrets it is authorized to access.

For Spring Boot, Vault can be integrated through Spring Cloud Vault. This allows configuration values to be supplied from Vault instead of being committed to the repository.

### 5. What About Database Passwords and User Passwords?

These are two different security scenarios.

**A. Database or service credentials**

For database passwords, API keys, and service-to-service credentials:

- Store them in a secrets manager.
- Restrict access to authorized workloads.
- Rotate credentials regularly or when compromise is suspected.
- Prefer short-lived credentials when supported.
- Use separate credentials for development, testing, and production.

**B. Passwords belonging to application users**

If users register and log in to your application, do not store their original passwords, even in a secrets manager.

Instead, store a **salted, computationally expensive password hash**, using an established password-hashing algorithm such as Argon2id, bcrypt, or PBKDF2. In Spring Security, use an appropriate `PasswordEncoder` implementation.

Example:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

When a user registers:

```java
String passwordHash =
        passwordEncoder.encode(rawPassword);

// Save passwordHash, never rawPassword.
userRepository.save(
        new User(username, passwordHash)
);
```

During login, Spring Security verifies the submitted password against the stored hash. The application should never need to retrieve the user's original password.

### 6. Best Practices for Production Microservices

- **Never hardcode secrets:** Keep credentials out of Java classes, YAML files, Git repositories, and container images.
- **Apply least privilege:** Each microservice should access only the secrets it needs.
- **Encrypt communications:** Use TLS for connections to secret stores and databases where supported.
- **Rotate credentials:** Support rotation and revoke compromised credentials promptly.
- **Avoid logging secrets:** Never print passwords, access tokens, or secret values in logs or exception messages.
- **Separate environments:** Do not reuse production credentials in development or testing.
- **Audit access:** Track secret access and monitor suspicious activity.
- **Use managed identities:** Prefer workload identity or platform-native authentication over embedding long-lived cloud credentials in the application.
- **Protect deployment pipelines:** CI/CD systems must also prevent secrets from appearing in build logs or artifacts.

### Quick Interview Revision

**Question:** What is the best practice for storing usernames and passwords in Spring Boot microservices?

**Answer:** I would never hardcode credentials or commit them to Git. In production, I would use a centralized secrets manager such as Google Cloud Secret Manager, AWS Secrets Manager, Azure Key Vault, or HashiCorp Vault. Each microservice would authenticate using its own workload identity and receive access only to the secrets it requires. I would also implement credential rotation, encryption, auditing, and secure logging. For application users' passwords, I would store salted password hashes using Spring Security's `PasswordEncoder`, never plaintext passwords.



---


## REST API Interview Questions and Answers

### 1. What are the HTTP methods in REST?

HTTP methods define the action a client wants to perform on a resource exposed by a REST API.

| Method | Purpose | Example |
|---|---|---|
| `GET` | Retrieve a resource | `GET /api/users/101` |
| `POST` | Create a resource or submit data for processing | `POST /api/users` |
| `PUT` | Replace a resource at a known URI | `PUT /api/users/101` |
| `PATCH` | Partially update a resource | `PATCH /api/users/101` |
| `DELETE` | Delete a resource | `DELETE /api/users/101` |
| `HEAD` | Retrieve response headers without the response body | `HEAD /api/users/101` |
| `OPTIONS` | Discover communication options supported by a resource | `OPTIONS /api/users` |

#### Spring Boot example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id) {
        return "User: " + id;
    }

    @PostMapping
    public String createUser(@RequestBody User user) {
        return "User created";
    }

    @PutMapping("/{id}")
    public String replaceUser(
            @PathVariable Long id,
            @RequestBody User user) {
        return "User replaced";
    }

    @PatchMapping("/{id}")
    public String updateUser(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        return "User partially updated";
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(
            @PathVariable Long id) {
        return ResponseEntity.noContent().build();
    }
}
```

The example illustrates endpoint mappings; a production controller should validate input, apply business rules, and persist the changes.

### 2. What are idempotent methods in REST?

An HTTP method is **idempotent** if making the same request multiple times has the same intended effect on the server as making it once.

| Method | Idempotent? | Explanation |
|---|---|---|
| `GET` | Yes | Intended to retrieve data without changing server state |
| `HEAD` | Yes | Like GET, but without the response body |
| `PUT` | Yes | Repeated requests establish the same resource state |
| `DELETE` | Yes | Repeated requests leave the resource deleted |
| `OPTIONS` | Yes | Intended to retrieve communication options |
| `POST` | Not guaranteed | Repeated requests may create multiple resources or repeat an action |
| `PATCH` | Depends on the operation | Some patches are idempotent; others are not |

**Example:**

```http
PUT /api/users/101
Content-Type: application/json

{
  "name": "Danish",
  "email": "danish@example.com"
}
```

Sending this request repeatedly should leave user 101 with the same specified state.

By contrast, repeatedly sending `POST /api/orders` may create multiple orders unless the API implements an idempotency mechanism.

**Important interview point:** Idempotency does not mean the response must be identical every time. For example, the first DELETE might return `204 No Content`, while a repeated DELETE might return `404 Not Found`. The intended server-state effect is still idempotent.

### 3. What standards should you follow when building a REST API?

Follow REST architectural principles and established HTTP conventions.

#### 1. Use resource-oriented URLs

Use nouns rather than action-oriented endpoint names.

Good:

```text
GET    /api/users
GET    /api/users/101
POST   /api/users
PUT    /api/users/101
DELETE /api/users/101
```

Avoid unnecessarily verb-oriented URLs such as:

```text
/api/getUsers
/api/createUser
/api/deleteUser/101
```

#### 2. Use appropriate HTTP status codes

| Status code | Meaning | Typical use |
|---|---|---|
| `200 OK` | Request succeeded | Successful GET or update returning a body |
| `201 Created` | Resource created | Successful POST |
| `202 Accepted` | Accepted for processing | Asynchronous operation |
| `204 No Content` | Success without a response body | Successful DELETE |
| `400 Bad Request` | Invalid request | Malformed input |
| `401 Unauthorized` | Authentication required or invalid | Missing or invalid credentials |
| `403 Forbidden` | Access denied | Authenticated caller lacks permission |
| `404 Not Found` | Resource not found | Unknown resource |
| `409 Conflict` | State conflict | Duplicate or conflicting operation |
| `429 Too Many Requests` | Rate limit exceeded | Excessive client requests |
| `500 Internal Server Error` | Unexpected server failure | Unhandled server-side error |
| `503 Service Unavailable` | Temporarily unavailable | Maintenance or overload |

#### 3. Keep APIs stateless

Each request should contain the information needed to process it, including relevant authentication credentials. Do not depend on server-side conversational session state to interpret the request.

Statelessness helps with scalability and load balancing.

#### 4. Use consistent request and response formats

JSON is common for REST APIs. Use consistent field names, error structures, validation messages, and pagination conventions.

Example response:

```json
{
  "id": 101,
  "name": "Danish",
  "email": "danish@example.com"
}
```

#### 5. Implement validation and error handling

Use Bean Validation annotations such as `@NotBlank`, `@Email`, and `@Size`. Use `@RestControllerAdvice` and `@ExceptionHandler` to return consistent error responses.

#### 6. Secure the API

Use HTTPS, authentication, authorization, input validation, rate limiting, and appropriate logging. Do not expose passwords, access tokens, or internal exception details.

#### 7. Use pagination, filtering, and sorting

For collection endpoints, provide consistent query parameters.

```http
GET /api/users?page=0&size=20&sort=name,asc
```

#### 8. Version APIs when necessary

For example:

```text
/api/v1/users
/api/v2/users
```

Choose a versioning strategy and define how backward compatibility and deprecation will be handled.

#### 9. Support caching and content negotiation where appropriate

Use HTTP caching headers such as `ETag` and `Cache-Control` where useful. Support `Accept` and `Content-Type` correctly when clients and services negotiate representations.

#### 10. Consider HATEOAS where appropriate

HATEOAS allows responses to include links to related resources and available actions. It is part of REST's uniform-interface constraints, but not every API needs a full hypermedia-driven design.

---

### 4. What is the difference between POST and PUT?

| Feature | POST | PUT |
|---|---|---|
| Typical purpose | Create a resource in a collection or process a submission | Create or replace a resource at a known URI |
| Resource URI | Commonly assigned by the server | Specified by the client |
| Idempotency | Not guaranteed | Idempotent |
| Repeated identical requests | May create multiple resources | Should establish the same intended state |
| Example | `POST /api/users` | `PUT /api/users/101` |

#### POST example

```http
POST /api/users
Content-Type: application/json

{
  "name": "Danish"
}
```

The server might create a new user and return:

```http
HTTP/1.1 201 Created
Location: /api/users/101
```

#### PUT example

```http
PUT /api/users/101
Content-Type: application/json

{
  "name": "Danish",
  "email": "danish@example.com"
}
```

PUT generally replaces the resource's state at the specified URI. Whether it creates a resource when one does not exist depends on the endpoint's contract and implementation.

**Interview tip:** PUT is idempotent; POST is not inherently idempotent. PUT does not mean “update only,” and POST does not exclusively mean “create.”

---

### 5. What is sent in HTTP headers? Can we intercept them?

HTTP headers carry metadata about a request or response.

Common request headers include:

| Header | Purpose |
|---|---|
| `Authorization` | Authentication credentials or access token |
| `Content-Type` | Format of the request body |
| `Accept` | Response formats the client accepts |
| `Accept-Language` | Preferred response language |
| `User-Agent` | Information about the client |
| `X-Request-ID` | Application-defined request correlation identifier |
| `If-Match` | Conditional request, often for concurrency control |

Response headers include `Cache-Control`, `ETag`, `Location`, and `Content-Type`.

#### How do you intercept headers in Spring Boot?

You can intercept requests or responses using filters, interceptors, or HTTP-client interceptors.

**A. Using `HandlerInterceptor`**

```java
@Component
public class RequestLoggingInterceptor
        implements HandlerInterceptor {

    @Override
    public boolean preHandle(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler) {

        String requestId = request.getHeader("X-Request-ID");

        System.out.println("Request ID: " + requestId);

        return true;
    }
}
```

Register it:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final RequestLoggingInterceptor interceptor;

    public WebConfig(RequestLoggingInterceptor interceptor) {
        this.interceptor = interceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(interceptor);
    }
}
```

A `HandlerInterceptor` is useful for MVC request handling, logging, and request-specific checks.

**B. Using `OncePerRequestFilter`**

A servlet filter runs earlier in the servlet processing pipeline and is commonly used for cross-cutting request processing, security-related checks, and header inspection.

For authentication and authorization, prefer Spring Security's security filter chain rather than implementing your own token validation from scratch.

**C. Intercepting outgoing HTTP requests**

For outgoing requests made using `RestClient`, `RestTemplate`, or `WebClient`, use the appropriate client interceptor or filter to add or inspect headers.

**Security note:** Never log the `Authorization` header or other secrets. Also, do not trust a client-supplied identity header unless a trusted component validates or establishes it.

---

### 6. How do you secure a REST API?

REST API security requires multiple layers.

#### 1. Authentication

Authentication verifies who the caller is.

Common approaches include:

- OAuth 2.0 access tokens.
- OpenID Connect for identity-related authentication.
- JWT access tokens, when appropriate.
- Mutual TLS for certain service-to-service scenarios.
- API keys for suitable use cases, usually with additional controls.

Basic authentication can be used over HTTPS in limited scenarios, but it requires careful credential handling and is not generally the preferred standalone strategy for modern distributed systems.

#### 2. Authorization

Authorization determines what the authenticated caller can do.

For example, Spring Security can restrict an endpoint to a specific role:

```java
@PreAuthorize("hasRole('ADMIN')")
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteUser(
        @PathVariable Long id) {

    userService.deleteUser(id);
    return ResponseEntity.noContent().build();
}
```

Enable method security with `@EnableMethodSecurity` where required.

#### 3. Use HTTPS

HTTPS protects data in transit. It does not replace authentication or authorization.

#### 4. Validate input

Validate request bodies, query parameters, path variables, and uploaded content. Use parameterized database queries and avoid constructing unsafe SQL.

#### 5. Apply rate limiting

Limit requests by appropriate dimensions, such as client, user, or token. Return `429 Too Many Requests` when a client exceeds the configured limit.

#### 6. Apply least privilege

Users, services, and database accounts should have only the permissions they need.

#### 7. Handle CORS correctly

Configure Cross-Origin Resource Sharing for the intended browser origins. CORS is not authentication and does not prevent non-browser clients from calling an API.

#### 8. Log and audit safely

Log relevant security events, access decisions, and request identifiers. Do not log passwords, full tokens, or sensitive personal data unnecessarily.

#### 9. Follow relevant security guidance

Use the OWASP API Security Top 10 as a practical checklist for risks such as broken object-level authorization, broken authentication, unrestricted resource consumption, and security misconfiguration. Apply industry-specific requirements such as PCI DSS when they are relevant.

---

### 7. How do you pass parameters in an HTTP request: URL or JSON body?

The appropriate location depends on the HTTP method and the meaning of the data.

#### A. Path variables

Use path variables to identify a particular resource.

```http
GET /api/users/101
```

Spring Boot:

```java
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
}
```

#### B. Query parameters

Use query parameters for filtering, sorting, searching, and pagination.

```http
GET /api/users?name=Danish&age=30
```

Spring Boot:

```java
@GetMapping
public List<User> searchUsers(
        @RequestParam String name,
        @RequestParam(required = false) Integer age) {

    return userService.search(name, age);
}
```

#### C. Request body

Use a JSON request body for structured data submitted to create or update a resource.

```http
POST /api/users
Content-Type: application/json

{
  "name": "Danish",
  "age": 30
}
```

Spring Boot:

```java
@PostMapping
public ResponseEntity<User> createUser(
        @Valid @RequestBody CreateUserRequest request) {

    User user = userService.create(request);

    return ResponseEntity.status(HttpStatus.CREATED)
            .body(user);
}
```

#### D. Request headers

Use headers for metadata such as authentication, content negotiation, and correlation identifiers.

```http
GET /api/users/101
Authorization: Bearer <access-token>
Accept: application/json
X-Request-ID: request-123
```

#### Which should you use?

| Data | Preferred location |
|---|---|
| Resource identifier | Path variable |
| Search/filter criteria | Query parameter |
| Pagination and sorting | Query parameter |
| New or updated resource data | JSON request body |
| Authentication token | Authorization header |
| Request tracing metadata | Header |

Avoid passing passwords, access tokens, or other secrets in query parameters because URLs may be retained in logs, browser history, and intermediary systems. HTTPS alone does not prevent URL exposure through those channels.

---

### Quick Interview Revision

- **HTTP methods:** GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS.
- **Idempotent methods:** GET, HEAD, PUT, DELETE, OPTIONS, and TRACE. PATCH may or may not be idempotent depending on the operation.
- **POST vs PUT:** POST is not inherently idempotent; PUT is idempotent.
- **REST standards:** Resource-oriented URLs, suitable HTTP methods and status codes, statelessness, validation, consistent errors, and appropriate caching and versioning.
- **Header interception:** Use servlet filters, Spring MVC interceptors, or HTTP-client interceptors depending on where interception is needed.
- **API security:** Authentication, authorization, HTTPS, input validation, rate limiting, least privilege, and safe auditing.
- **Request parameters:** Path variables identify resources, query parameters filter collections, request bodies carry structured data, and headers carry metadata.

---