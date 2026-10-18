---
title: Monolith vs Microservices
parent: Microservices
nav_order: 1
---

# Monolith vs Microservices
{: .no_toc }

Trade-offs, architectural differences, advantages, disadvantages, and when to choose each approach.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

1. TOC
{:toc}
</details>

---

## What is Monolithic Architecture?

A **monolithic architecture** is a software architectural style in which an application is developed, packaged, and deployed as a single unit.

The application may contain multiple modules, such as authentication, user management, payments, orders, and notifications. However, these modules typically run within the same application process and are deployed together.

Example

Consider an e-commerce application with the following modules:

- User Management
- Product Catalog
- Order Management
- Payment Processing
- Inventory Management
- Notification Service

In a traditional monolithic application, these modules are part of one application, share the same deployment lifecycle, and may share a common database.

### Monolithic architecture diagram

```text
             Client / Browser
                    |
                    v
          +---------------------+
          |   Monolithic App    |
          |---------------------|
          | User Management     |
          | Product Catalog     |
          | Order Management    |
          | Payment Processing  |
          | Inventory Management|
          | Notifications       |
          +---------------------+
                    |
                    v
             +-------------+
             |  Database   |
             +-------------+
```

### Characteristics of a monolith

- Single deployment unit.
- Usually one application runtime.
- Modules can call one another directly within the process.
- Often uses a shared database.
- Simpler local development and debugging.
- Scaling commonly involves deploying additional instances of the entire application.
- A change to one module may require rebuilding and redeploying the application.

**Important:** A monolith does not have to be poorly designed. A modular monolith can maintain clear boundaries between business domains while retaining a single deployment unit.

## What is Microservices Architecture?

**Microservices architecture** is an architectural style that structures an application as a collection of small, autonomous services, modeled around business capabilities or domains.

Each service owns a specific business responsibility and can be developed, deployed, and scaled independently, provided its dependencies and deployment infrastructure support that independence.

For example, an e-commerce platform may have separate services for users, products, orders, payments, inventory, and notifications.

### Microservices architecture diagram

```text
                 Client / Browser
                        |
                        v
                 +-------------+
                 | API Gateway |
                 +-------------+
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
    +-----------+ +-----------+ +-----------+
    |   User    | |  Product  | |   Order   |
    |  Service  | |  Service  | |  Service  |
    +-----------+ +-----------+ +-----------+
          |             |             |
          v             v             v
    +-----------+ +-----------+ +-----------+
    | User DB   | | Product DB| | Order DB  |
    +-----------+ +-----------+ +-----------+
                                      |
                                      v
                               +-------------+
                               |   Payment   |
                               |   Service   |
                               +-------------+
                                      |
                                      v
                                +-----------+
                                | Payment DB|
                                +-----------+
```

The diagram is illustrative. Real systems may use synchronous REST or gRPC calls, asynchronous messaging, or a combination of both.

### Key characteristics of microservices

**1. Decoupling**

Services communicate through explicit interfaces and aim to minimize dependencies on each other's internal implementation.

**2. Componentization**

Services can be independently maintained, upgraded, and replaced when their contracts remain compatible.

**3. Business capabilities**

Each service focuses on a well-defined business responsibility, such as payment processing or inventory management.

**4. Autonomy**

Teams can make implementation and release decisions independently within agreed architectural standards.

**5. Independent deployment**

A change to the payment service does not necessarily require redeploying the order or user service.

**6. Independent scalability**

Services can be scaled according to their individual resource demands.

**7. Decentralized data ownership**

Each service owns its business data. Other services access that data through APIs or events rather than directly modifying its tables.

**8. Technology flexibility**

Different services can use different technologies when the benefits justify the additional operational complexity.

**9. Continuous delivery**

Automated build, testing, deployment, monitoring, and rollback pipelines enable smaller and more frequent releases.

**10. Product ownership**

Teams own the ongoing operation, reliability, and evolution of their services rather than treating delivery as a one-time project.


## Monolith vs Microservices: Key Differences

| Aspect | Monolithic Architecture | Microservices Architecture |
|---|---|---|
| Structure | One application unit | Multiple independently managed services |
| Deployment | Usually deployed together | Services can be deployed independently |
| Scalability | Typically scales the entire application | Can scale individual services |
| Development | Centralized application development | Teams can work independently |
| Communication | Mostly in-process calls | Network calls and asynchronous messaging |
| Database | Often shared | Data ownership is usually per service |
| Technology | Often one primary stack | Different stacks are possible |
| Testing | Relatively straightforward integration testing | More distributed testing and contract testing |
| Debugging | Often easier to trace locally | Requires distributed tracing and correlation IDs |
| Fault isolation | A process failure can affect the whole application | Failures can be isolated, but cascading failures remain possible |
| Performance | Lower overhead for internal calls | Network communication adds overhead |
| Deployment complexity | Generally lower | Generally higher |
| Infrastructure cost | Often lower initially | Often higher due to infrastructure and operations |
| Best suited for | Small to medium systems and tightly integrated domains | Large systems with independently evolving business capabilities |

**Interview takeaway:** Microservices improve deployment and scaling independence, but introduce distributed-systems complexity. They are not automatically faster, cheaper, or more reliable.

---

## Advantages of Microservices Over Monolithic Architecture

### 4.1 Independent deployment

In a monolith, changing one module commonly requires rebuilding and redeploying the entire application.

With microservices, a payment-service update can be released independently if it does not require coordinated changes to other services.

**Example:** A bug fix in payment processing does not require redeploying the product catalog.

### 4.2 Independent scalability

Different business capabilities may have different traffic patterns.

For example:

- Product Catalog: 10 instances
- Order Service: 5 instances
- Payment Service: 3 instances

These numbers are illustrative. Actual scaling depends on CPU, memory, throughput, latency, and downstream capacity.

The main advantage is that each service can be scaled based on its own workload.

### 4.3 Technology flexibility

A team can select technologies appropriate to a service's requirements.

For example, a Java Spring Boot service may handle transactions, while another service uses Python for data processing.

However, too many technology stacks can increase maintenance, recruitment, security, and operational costs.

### 4.4 Better fault isolation

A failure in one service does not have to bring down the entire application.

For example, if the recommendation service fails, customers may still be able to browse products and place orders.

This requires proper timeouts, circuit breakers, fallbacks, bulkheads, and graceful degradation. Microservices alone do not guarantee fault isolation.

### 4.5 Team autonomy

Different teams can own separate services and release them on different schedules.

This is particularly useful when an organization has many teams working on different business capabilities.

### 4.6 Easier evolution of large systems

Services can evolve independently as business requirements change, provided API contracts, event schemas, and data ownership boundaries are managed carefully.

### 4.7 More focused codebases

A well-designed service has a limited business responsibility, which can make its code easier to understand and maintain.

However, poorly defined service boundaries can create excessive inter-service dependencies and negate this benefit.

### 4.8 Improved release frequency

With automated CI/CD, teams can test and deploy smaller changes more frequently, reducing the scope of individual releases.



## Challenges of Monolithic Architecture

### 5.1 Limited independent scalability

If only the payment module experiences heavy traffic, the entire application may need additional instances.

This can waste resources on modules that do not need additional capacity.

### 5.2 Tightly coupled modules

Poorly structured monoliths can develop dependencies between business modules, making changes risky.

A small modification may unexpectedly affect multiple parts of the application.

### 5.3 Large deployment scope

Even a small change may require the full application to be rebuilt, tested, and deployed.

### 5.4 Slower builds and tests

As an application grows, compilation, automated testing, and deployment can become slower.

This is not inevitable; good modularization, caching, and test strategies can mitigate the problem.

### 5.5 Team coordination challenges

Multiple teams changing the same codebase may encounter merge conflicts, shared release dependencies, and coordination overhead.

### 5.6 Technology migration constraints

Changing a major framework, runtime, or programming language may require migrating much of the application together.

### 5.7 Larger failure impact

A memory leak, resource exhaustion, or critical defect in the shared application process can affect multiple business capabilities.

A modular monolith can reduce logical coupling, but modules still share the process and deployment lifecycle.



## Disadvantages of Microservices Architecture

### 6.1 Increased architectural complexity

Instead of managing one application, the organization must manage multiple services, interfaces, deployments, and dependencies.

### 6.2 Network latency

Communication between services introduces serialization, network transmission, and additional failure points.

For example, an order request might call the inventory and payment services before it can finish processing.

### 6.3 Distributed transactions

A business operation may span several services and databases. Maintaining consistency requires patterns such as Saga, transactional outbox, idempotency, and compensating transactions.

### 6.4 Difficult debugging

A single request may travel through several services. Distributed tracing, centralized logs, metrics, and correlation IDs are often necessary to identify the root cause.

### 6.5 More complex testing

Testing must cover individual services, API contracts, integrations, asynchronous events, and end-to-end workflows.

### 6.6 Operational overhead

Each service may require deployment configuration, health checks, secrets management, dashboards, alerts, scaling policies, and incident-response procedures.

### 6.7 Data consistency challenges

When services own separate databases, updates may propagate asynchronously. Consumers must account for eventual consistency and delayed events.

### 6.8 Cascading failures

A slow downstream service can consume threads, connections, or other resources in upstream services, causing a broader outage.

Timeouts, circuit breakers, bulkheads, bounded retries, and load shedding help mitigate this risk.

### 6.9 Higher infrastructure costs

Multiple runtimes, replicas, message brokers, observability systems, and orchestration infrastructure can increase the total cost of ownership.

### 6.10 Distributed security

Every service-to-service interaction must be secured appropriately, including authentication, authorization, transport encryption, and secret rotation.



## When Should You Choose a Monolith?

A monolithic architecture is often the better choice when:

- The application is small or has a limited number of business capabilities.
- The product is in its early stages and requirements change frequently.
- The development team is small.
- There is limited DevOps or platform-engineering capacity.
- Low-latency in-process communication is important.
- Most modules must change and release together.
- The domain boundaries are not yet well understood.
- Simplicity, delivery speed, and low operational cost are priorities.

### Example

A startup is building an internal employee-management application with authentication, employee profiles, leave requests, and reports.

A well-structured Spring Boot modular monolith may be sufficient. It can be deployed as one application and later split into services if real business or scaling requirements justify the change.



## When Should You Choose Microservices?

Microservices may be appropriate when:

- The system has multiple independently evolving business capabilities.
- Different teams need independent release schedules.
- Specific components have substantially different scaling requirements.
- The organization has mature CI/CD, monitoring, security, and incident-response practices.
- Services have clear ownership and manageable interfaces.
- Independent deployment provides measurable business value.
- The organization can manage distributed data and failure scenarios.

### Example

A large e-commerce platform has separate teams for catalog, orders, payments, inventory, shipping, and notifications.

Product browsing receives heavy traffic during sales, while payment processing requires strict security and careful transaction handling. Independent ownership, scaling, and deployment may justify a microservices architecture.



## When Are Microservices Not Preferable?

Microservices may be a poor fit under the following circumstances.

| Situation | Why microservices may be unsuitable |
|---|---|
| Small application | Distributed architecture adds unnecessary complexity |
| Limited budget | Infrastructure and operations may cost more |
| Small team | Service ownership and on-call responsibilities can overwhelm the team |
| Unclear domain boundaries | Poor service decomposition creates excessive dependencies |
| Strongly coupled workflows | Frequent cross-service coordination can negate autonomy |
| Strict latency requirements | Network calls may add unacceptable overhead |
| Immature DevOps practices | Independent deployment and operational reliability become difficult |
| Simple CRUD application | A single application and database may be sufficient |
| Insufficient observability | Distributed failures become difficult to diagnose |

**Important:** None of these factors automatically rules out microservices. The decision depends on measurable requirements, the team's capabilities, and the costs of each alternative.



## What Is a Modular Monolith?

A modular monolith is a single deployable application divided into clearly defined modules with controlled dependencies.

For example:

```text
              Spring Boot Application
                       |
          +------------+------------+
          |            |            |
          v            v            v
      +--------+   +--------+   +---------+
      |  User  |   | Order  |   | Payment |
      | Module |   | Module |   | Module  |
      +--------+   +--------+   +---------+
          |            |            |
          +------------+------------+
                       |
                Shared Runtime
                       |
                 Application DB
```

A modular monolith can use separate packages, module interfaces, domain boundaries, and even separate schemas while retaining a single deployment unit.

### Why is it useful?

- Simpler deployment and operations than microservices.
- Clearer separation of business responsibilities than an unstructured monolith.
- Easier local development and debugging.
- Provides a foundation for extracting services when independent deployment becomes valuable.

**Recommended principle:** Start with a well-modularized monolith when the need for distributed deployment is uncertain. Extract services when concrete business or technical constraints justify it.



## Can We Migrate From Monolith to Microservices?

Yes. This is commonly done incrementally rather than rewriting the entire system at once.

A useful approach is the **Strangler Fig pattern**, where new functionality is introduced as separate services while existing functionality remains in the monolith.

### Migration steps

1. Understand the business domains and existing dependencies.
2. Identify a capability that benefits from independent deployment or scaling.
3. Define a clear API or event contract.
4. Extract the capability into a separate service.
5. Establish data ownership and a migration strategy.
6. Route relevant requests to the new service.
7. Add monitoring, automated tests, security, and rollback mechanisms.
8. Repeat only where the benefits justify the cost.

### Example

Suppose an existing e-commerce monolith handles user management, orders, and notifications.

Notifications can be a useful extraction candidate because they may be processed asynchronously and have a distinct operational profile.

However, the best first service depends on the application's actual dependencies and bottlenecks, not simply which module appears easiest to extract.



## Real-World Example: E-Commerce Order Processing

Consider a customer placing an order.

### Monolithic implementation

```text
Customer Request
      |
      v
Monolithic Application
      |
      +--> Validate Customer
      |
      +--> Check Inventory
      |
      +--> Create Order
      |
      +--> Process Payment
      |
      +--> Send Notification
      |
      v
Response to Customer
```

All operations may execute within one application, using direct method calls and potentially a single database transaction for operations that share the same database.

This can be simpler to implement and debug.

### Microservices implementation

```text
Customer Request
      |
      v
   API Gateway
      |
      v
  Order Service
      |
      +----> Inventory Service
      |
      +----> Payment Service
      |
      +----> Message Broker
                    |
                    v
             Notification Service
```

The exact flow depends on the business requirements. For example, order creation and payment processing may be coordinated asynchronously using a Saga rather than a single distributed transaction.

### Important design considerations

- Use timeouts for remote calls.
- Use idempotency keys for operations such as payment requests.
- Avoid unbounded retries.
- Handle payment failures and inventory reservation failures explicitly.
- Use asynchronous messaging for suitable background tasks.
- Track requests using correlation IDs and distributed traces.
- Design compensating actions for workflows that span multiple services.



## Frequently Asked Interview Questions

### Q1. What is the main difference between monolithic and microservices architecture?

A monolith is generally deployed as one application unit, whereas microservices divide the system into independently managed services that communicate through explicit interfaces.

### Q2. Are microservices always better than monoliths?

No. Microservices provide independent deployment and scaling, but introduce network failures, distributed data management, security, observability, and operational complexity.

### Q3. Can a monolith scale?

Yes. A monolith can scale vertically by increasing resources or horizontally by running multiple instances behind a load balancer. Its main limitation is that individual modules generally cannot be scaled independently.

### Q4. Do microservices require separate databases?

Not necessarily in every implementation, but each service should own its data and control how it is accessed. Separate databases or schemas can help enforce these boundaries.

Multiple services sharing tables and directly modifying each other's data can create tight coupling.

### Q5. How do microservices communicate?

Common approaches include:

- REST APIs over HTTP.
- gRPC for efficient remote procedure calls.
- Messaging systems such as Apache Kafka and RabbitMQ.
- Event-driven communication for asynchronous workflows.

### Q6. How do you handle failures between microservices?

Use suitable timeouts, circuit breakers, bulkheads, bounded retries with backoff, idempotent operations, fallbacks, and monitoring.

Do not retry every failure indiscriminately. Retries can worsen overload and duplicate non-idempotent operations.

### Q7. How do you maintain data consistency across microservices?

Common approaches include Saga workflows, transactional outbox, idempotent event consumers, and eventual consistency.

A distributed transaction should not be the default solution for every cross-service workflow.

### Q8. What is the Strangler Fig pattern?

It is a migration approach in which functionality is gradually moved from a legacy application to new services while the existing application continues to operate.

### Q9. What is a modular monolith?

It is a single deployable application containing well-separated modules with defined responsibilities and controlled dependencies.

### Q10. What is the biggest disadvantage of microservices?

There is no single universal disadvantage. A common major challenge is the operational and distributed-systems complexity introduced by network communication, independent data ownership, deployment, and failure handling.

### Q11. How do you decide the right size of a microservice?

Design services around cohesive business capabilities and clear ownership, rather than arbitrary code size or the number of classes.

A service should be independently understandable and deployable without excessive coordination with other services.

### Q12. Can microservices use the same technology stack?

Yes. Microservices do not require different programming languages. Using a common stack such as Java, Spring Boot, and PostgreSQL can simplify development and operations.

### Q13. Does microservices architecture guarantee fault tolerance?

No. It can enable better isolation, but only if the system is designed with appropriate failure-handling mechanisms and resilient dependencies.

### Q14. What is the difference between SOA and microservices?

Both architectures organize software around services. Microservices commonly emphasize smaller, independently deployable services, decentralized ownership, and autonomous data management. Service-oriented architecture (SOA) is broader and may use shared enterprise services, centralized governance, or an enterprise service bus.

These are tendencies rather than absolute rules.

### Q15. What would you choose for a new Spring Boot application?

I would generally start with a modular monolith unless there are clear requirements for independent deployment, different scaling profiles, or autonomous team ownership. I would introduce microservices when their benefits outweigh the additional operational complexity.



## Quick Revision Notes

- **Monolith:** One application, generally one deployment unit.
- **Microservices:** Multiple independently managed and potentially independently deployable services.
- **Main microservices benefit:** Independent deployment, scaling, and ownership.
- **Main microservices challenge:** Distributed-systems and operational complexity.
- **Main monolith benefit:** Simpler development, testing, deployment, and debugging.
- **Main monolith limitation:** Modules generally share the application's deployment and scaling lifecycle.
- **Modular monolith:** One deployment unit with clear internal module boundaries.
- **Strangler Fig:** Incremental migration from a legacy application to new services.
- **Saga:** Coordinates a business transaction across services using local transactions and compensating actions where necessary.
- **Circuit breaker:** Stops repeated calls to a failing dependency temporarily to reduce cascading failures.
- **API Gateway:** Provides a common entry point for client requests to backend services.
- **Service discovery:** Helps clients or infrastructure locate service instances dynamically.
- **Observability:** Uses logs, metrics, and traces to understand system behavior.
- **Transactional outbox:** Helps reliably publish events associated with local database transactions.

## Final Recommendation

Choose architecture based on business needs rather than trends.

A well-designed monolith is often the simplest way to build a reliable application. Microservices become valuable when independent deployment, scaling, ownership, or fault isolation solve real problems that outweigh their costs.

For Java and Spring Boot development, understand both approaches deeply: start with sound modular design, and learn the distributed-systems patterns needed when an application genuinely requires microservices.





