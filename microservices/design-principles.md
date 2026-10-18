---
title: Design Principles
parent: Microservices
nav_order: 2
---

# Design Principles
{: .no_toc }

Principles for designing service boundaries, building resilient services, and developing maintainable, scalable distributed systems.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

1. TOC
{:toc}
</details>

---

## What Are the Design Principles of Microservices?

Microservices design principles are guidelines for creating services that are independently maintainable, deployable, scalable, and resilient.

The goal is not simply to split a monolithic application into smaller services. The goal is to create services with clear responsibilities, well-defined boundaries, and minimal unnecessary dependencies.

### 1.1 Modularity

Each service should have a clear responsibility and encapsulate the business logic associated with that responsibility.

For example, an e-commerce application may have separate services for orders, payments, inventory, and notifications.

A service should expose its functionality through a defined API or event contract rather than exposing its internal implementation.

### 1.2 Single Responsibility

A service should focus on a cohesive business capability.

For example, a Payment Service handles payment-related operations rather than also managing employee records or product catalogs.

This principle is related to the Single Responsibility Principle (SRP) from SOLID, although a service boundary usually operates at a larger architectural level than a class boundary.

### 1.3 Loose Coupling

Services should minimize dependencies on the internal implementation of other services.

Prefer stable APIs and versioned event contracts. Avoid direct access to another service's internal tables.

**Example:** Order Service requests a payment through a Payment Service API instead of directly updating payment records in the payment database.

### 1.4 High Cohesion

Related business functionality should be grouped together within a service.

For example, creating, cancelling, and retrieving orders may belong to the Order Service because they operate on the same business domain.

High cohesion makes services easier to understand and maintain.

### 1.5 Independent Scalability

Services should be scalable according to their individual workloads.

For example, a Product Catalog may need many replicas during a sale, while the Notification Service may need fewer replicas.

Independent scaling requires appropriate infrastructure, resource limits, load balancing, and consideration of downstream capacity.

### 1.6 Independent Deployment

A service should be deployable without requiring unrelated services to be redeployed.

This requires backward-compatible API changes, compatible event schemas, and automated deployment pipelines.

A service is not truly independently deployable if every release requires coordinated changes across the entire system.

### 1.7 Decentralized Data Management

Each service should own and control its business data.

For example:

- User Service owns user profiles.
- Order Service owns order records.
- Payment Service owns payment records.
- Inventory Service owns stock information.

Other services should access this information through APIs or events rather than directly modifying another service's database.

This avoids tight coupling through shared database tables.

### 1.8 Statelessness

Where practical, application instances should avoid keeping essential session state exclusively in local memory.

State can be stored in databases, distributed caches, or other suitable backing services.

Stateless application instances can generally be replaced, restarted, and scaled more easily.

**Important:** Statelessness does not mean the business domain has no state. Orders, payments, and inventory are inherently stateful business entities. The principle concerns where runtime state is held and how instances handle requests.

### 1.9 Resilience

Services should continue operating safely when dependencies become unavailable or slow.

Common techniques include:

- Timeouts.
- Circuit breakers.
- Bounded retries with backoff.
- Bulkheads.
- Fallback responses.
- Graceful degradation.
- Idempotent operations.

For example, if the recommendation service fails, the e-commerce application may still allow customers to browse products and place orders.

### 1.10 High Availability

Services should be designed to minimize downtime and recover quickly from failures.

Common techniques include:

- Multiple service replicas.
- Health checks.
- Load balancing.
- Redundant infrastructure.
- Automated recovery.
- Appropriate database replication and backups.

Multiple replicas alone do not guarantee high availability. Shared dependencies and infrastructure failures must also be considered.

### 1.11 Observability

Services should expose enough operational information to understand their internal behavior.

The three major pillars are:

- **Logs:** Records of application events and errors.
- **Metrics:** Numerical measurements such as request rate, error rate, latency, and CPU usage.
- **Distributed traces:** Show how a request travels across multiple services.

Use correlation or trace IDs to connect related events across service boundaries.

### 1.12 Automation

Automate repetitive activities such as:

- Building applications.
- Running unit and integration tests.
- Scanning dependencies for vulnerabilities.
- Building container images.
- Deploying services.
- Running database migrations.
- Checking application health.
- Rolling back unsuccessful deployments.

Tools may include Maven, GitHub Actions, GitLab CI/CD, Docker, Kubernetes, and observability platforms.

### 1.13 Security by Design

Security should be incorporated into every service rather than added at the end.

Important practices include:

- Authentication and authorization.
- Least-privilege access.
- Secure service-to-service communication.
- TLS encryption.
- Centralized secret management.
- Input validation.
- Dependency vulnerability scanning.
- Audit logging.
- Regular secret rotation.

A gateway can enforce common security controls, but individual services must still enforce their own authorization requirements.

### 1.14 API Contract Stability

Service contracts should be explicit and backward-compatible whenever possible.

For example, adding an optional field to a JSON response is often safer than renaming or removing an existing field that other services consume.

Use contract testing to verify compatibility between service providers and consumers.

### 1.15 Design for Failure

Assume that networks, databases, message brokers, and remote services can fail.

A robust service should define:

- What happens when a dependency times out.
- Whether a request can be safely retried.
- How partial failures are handled.
- How operations recover after a restart.
- How alerts identify production problems.

### 1.16 Right-Sized Services

Do not create a separate service for every class, database table, or small function.

A service should represent a cohesive business capability with a clear reason to exist independently.

Too many tiny services can increase network calls, deployment complexity, and operational overhead.

---

## Quick Summary of Microservices Principles

| Principle | Main objective |
|---|---|
| Modularity | Separate business responsibilities |
| Single responsibility | Keep each service focused |
| Loose coupling | Minimize dependencies |
| High cohesion | Group related functionality |
| Independent scalability | Scale according to demand |
| Independent deployment | Release services separately |
| Data ownership | Control data within each service |
| Statelessness | Make application instances easier to replace |
| Resilience | Handle failures safely |
| High availability | Minimize downtime |
| Observability | Understand production behavior |
| Automation | Reduce manual operational work |
| Security by design | Protect data and operations |
| API stability | Prevent breaking dependent services |
| Design for failure | Anticipate distributed-system failures |

---

## The 12-Factor App Methodology

### What is the 12-Factor methodology?

The **12-Factor App** is a methodology for building software-as-a-service applications that are portable, maintainable, and suitable for deployment across different environments.

It is particularly useful for microservices because it encourages externalized configuration, explicit dependencies, disposable processes, and environment consistency.

The methodology was originally published by Heroku. Its principles are not specific to Java or Spring Boot and are not mandatory rules for every architecture.

### The 12 factors at a glance

| No. | Factor | Meaning |
|---:|---|---|
| 1 | Codebase | Track one codebase per application, with multiple deployments |
| 2 | Dependencies | Explicitly declare and isolate dependencies |
| 3 | Config | Store environment-specific configuration outside the code |
| 4 | Backing Services | Treat databases and other dependencies as attached resources |
| 5 | Build, Release, Run | Separate build, release, and execution stages |
| 6 | Processes | Run the application as one or more stateless processes |
| 7 | Port Binding | Export functionality through a bound port |
| 8 | Concurrency | Scale out using additional processes or instances |
| 9 | Disposability | Start quickly and shut down gracefully |
| 10 | Dev/Prod Parity | Keep development and production environments similar |
| 11 | Logs | Treat logs as event streams |
| 12 | Admin Processes | Run administrative tasks as separate processes |

---

## Factor 1: Codebase

**Principle:** Track code in a version-control system, with one codebase per application and multiple deployments from that codebase.

For example, the same Order Service codebase can be deployed to development, staging, and production.

```text
Order Service Repository
          |
          +----> Development
          |
          +----> Staging
          |
          +----> Production
```

Each deployment may use different configuration, but the application source is maintained in version control.

### Practical implementation

- Use Git.
- Maintain a consistent branching and release strategy.
- Tag application releases.
- Track changes through pull requests.
- Promote tested artifacts between environments.

**Interview point:** One codebase does not necessarily mean one repository for the entire microservices system. Different services may have separate repositories or live in a monorepo, depending on the team's workflow.

---

## Factor 2: Dependencies

**Principle:** Explicitly declare and isolate dependencies.

An application should not rely on undeclared libraries installed manually on a developer's machine.

In Java, dependencies can be declared in `pom.xml` or `build.gradle`.

### Maven example

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Maven resolves declared dependencies and their transitive dependencies.

### Best practices

- Declare dependencies explicitly.
- Pin or manage versions through a dependency-management strategy.
- Use reproducible builds.
- Scan libraries for known vulnerabilities.
- Avoid relying on globally installed tools at runtime.

**Interview point:** Explicit dependencies improve reproducibility, portability, and maintainability.

---

## Factor 3: Config

**Principle:** Store deployment-specific configuration outside application code.

Configuration includes:

- Database URLs.
- API endpoints.
- Port numbers.
- Feature flags.
- Message broker addresses.
- Environment-specific settings.

Credentials and secrets should be managed using appropriate secret-management mechanisms, not committed to source control.

### Spring Boot example

```yaml
server:
  port: ${SERVER_PORT:8080}

spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

Example environment variables:

```bash
export SERVER_PORT=8081
export DB_URL=jdbc:postgresql://localhost:5432/orders
export DB_USERNAME=order_user
export DB_PASSWORD='replace-with-a-secret'
```

The example password is a placeholder. In production, inject secrets through a secure secret manager or deployment platform.

### Why is this important?

The same application artifact can run in different environments without modifying its source code.

**Interview point:** Configuration is externalized; secrets require stronger controls than ordinary configuration values.

---

## Factor 4: Backing Services

**Principle:** Treat backing services as attached resources.

A backing service is an external resource that an application uses to perform its work.

Examples include:

- PostgreSQL and MySQL.
- MongoDB.
- Redis.
- Apache Kafka.
- RabbitMQ.
- SMTP services.
- External payment APIs.
- Object storage.

The application accesses these resources through configuration and defined interfaces.

For example, an Order Service may use PostgreSQL for order data and Kafka for publishing order events.

```text
Order Service
     |
     +----> PostgreSQL
     |
     +----> Kafka
     |
     +----> Payment API
```

This approach makes it easier to change resource endpoints or environments without rewriting the application's business logic.

**Interview point:** Treating a backing service as an attached resource does not mean that replacing a database or broker is always trivial. Data migration, compatibility, and operational concerns still matter.

---

## Factor 5: Build, Release, Run

**Principle:** Strictly separate the build, release, and run stages.

### Build

Compile the source code, resolve dependencies, execute build-time checks, and create an artifact or container image.

```bash
mvn clean package
```

### Release

Combine a specific build artifact with the configuration needed for a particular deployment.

For example, release version `1.2.0` of the Order Service to staging.

### Run

Execute the released artifact in the target environment.

```bash
java -jar target/order-service.jar
```

The exact commands depend on the project and deployment platform.

### Typical CI/CD flow

```text
Source Code
     |
     v
   Build
     |
     v
   Tests
     |
     v
Versioned Artifact
     |
     v
   Release
     |
     v
     Run
```

**Interview point:** The build produces the artifact; the release combines the artifact with deployment configuration; the run stage executes it.

---

## Factor 6: Processes

**Principle:** Execute the application as one or more stateless processes.

Application instances should not depend on local memory or local files as the sole source of durable business state.

For example, if an Order Service restarts, its orders must remain available from durable storage.

### Incorrect approach

```java
private final Map<String, Order> orders = new HashMap<>();
```

Using an in-memory map as the sole persistent order store is unsafe because the data is lost when the process restarts and is not shared between replicas.

### Better approach

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public Order save(Order order) {
        return orderRepository.save(order);
    }
}
```

The repository persists the order in the configured database.

**Important:** In-memory caches, temporary computation, and local working state are still acceptable. Durable business state must not depend exclusively on an individual process.

---

## Factor 7: Port Binding

**Principle:** Export the application's service functionality through a port.

A Spring Boot web application typically starts an embedded web server and listens on a configured port.

```yaml
server:
  port: 8080
```

Clients or gateways can then access the application over HTTP.

For example:

```text
http://localhost:8080/api/orders
```

In containerized environments, the application may listen on port 8080 inside the container while the platform exposes it through a Service, load balancer, or ingress.

**Interview point:** Port binding makes the service self-contained with respect to its web-serving runtime. It does not eliminate the need for networking, routing, or access controls.

---

## Factor 8: Concurrency

**Principle:** Scale out using the process model.

Instead of relying exclusively on one increasingly powerful server, run multiple instances or processes to handle additional workload.

```text
                Load Balancer
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       Instance 1  Instance 2  Instance 3
```

### Example

An Order Service initially runs one replica. When demand increases, Kubernetes can run more replicas.

The number of replicas should be determined by workload, resource limits, latency, and downstream capacity.

### Important considerations

- Keep application instances replaceable.
- Use external storage for durable state.
- Make event consumers safe against duplicate delivery.
- Configure connection pools appropriately.
- Ensure the database can handle additional concurrent connections.
- Use autoscaling based on meaningful metrics.

**Interview point:** Horizontal scaling increases capacity only when the application and its dependencies can handle the additional load.

---

## Factor 9: Disposability

**Principle:** Maximize robustness through fast startup and graceful shutdown.

Application instances should start quickly and stop cleanly so that deployment platforms can replace them safely.

### Graceful shutdown in Spring Boot

```yaml
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 20s
```

This configures Spring Boot's graceful shutdown behavior. The timeout should be selected according to the application's request durations and shutdown requirements.

### Good practices

- Stop accepting new requests during shutdown.
- Allow in-flight requests to finish within a bounded period.
- Release database connections and other resources.
- Handle termination signals.
- Avoid losing messages during consumer shutdown.
- Make startup and recovery safe to repeat.

**Interview point:** Graceful shutdown reduces interrupted work during rolling deployments and infrastructure maintenance.

---

## Factor 10: Dev/Prod Parity

**Principle:** Keep development, staging, and production as similar as practical.

Differences between environments can cause bugs that are difficult to reproduce.

For example, developing against an in-memory database while production uses PostgreSQL may hide SQL compatibility or transaction-related problems.

### Best practices

- Use containers for local dependencies where practical.
- Keep database engines and versions compatible.
- Use automated integration tests.
- Manage configuration through environment-specific deployment settings.
- Keep dependencies and runtime versions consistent.
- Avoid manual production-only configuration changes.

**Interview point:** Environments do not need to be identical, but differences should be deliberate, documented, and controlled.

---

## Factor 11: Logs

**Principle:** Treat logs as event streams.

An application should write logs to standard output or another suitable logging interface, leaving collection, storage, indexing, and retention to the platform or logging infrastructure.

### Spring Boot example

```java
private static final Logger log =
        LoggerFactory.getLogger(OrderService.class);

public void processOrder(String orderId) {
    log.info("Processing orderId={}", orderId);
}
```

### Logging best practices

- Prefer structured logs in production.
- Include correlation IDs or trace IDs.
- Log meaningful errors and business events.
- Avoid logging passwords, access tokens, payment credentials, or unnecessary personal information.
- Use appropriate log levels.
- Centralize log collection where required.

Common logging platforms include the Elastic Stack, Grafana Loki, and cloud-native logging services.

**Interview point:** Logs are records of events, not a substitute for metrics or distributed tracing.

---

## Factor 12: Admin Processes

**Principle:** Run administrative or management tasks as one-off processes using the same application codebase and configuration as the main application.

Examples include:

- Database migrations.
- Data imports and exports.
- One-time cleanup jobs.
- Reconciliation jobs.
- Administrative maintenance commands.

For example, a Flyway migration may run as part of a controlled deployment process rather than being executed manually against production.

### Best practices

- Automate repeatable administrative tasks.
- Use appropriate access controls.
- Make jobs safe to rerun where possible.
- Log execution outcomes.
- Avoid running unbounded or duplicate jobs.
- Use a dedicated scheduler or job system for recurring workloads when appropriate.

**Interview point:** One-off administrative work should use the same deployment assumptions and configuration discipline as the application.

---

## Applying the 12 Factors to Spring Boot Microservices

Consider an Order Service built using Java, Spring Boot, PostgreSQL, Docker, and Kubernetes.

| Factor | Example implementation |
|---|---|
| Codebase | Git repository with version-controlled code |
| Dependencies | Maven `pom.xml` |
| Config | Environment variables and Spring configuration |
| Backing Services | PostgreSQL, Kafka, Redis |
| Build, Release, Run | Maven, container image, deployment pipeline |
| Processes | Replaceable Spring Boot instances |
| Port Binding | Embedded web server on port 8080 |
| Concurrency | Multiple Kubernetes replicas |
| Disposability | Startup probes and graceful shutdown |
| Dev/Prod Parity | Docker Compose for compatible local dependencies |
| Logs | Structured logs collected centrally |
| Admin Processes | Flyway migrations and controlled maintenance jobs |

The methodology complements microservices architecture, but it does not define service boundaries, solve distributed transactions, or guarantee resilience on its own.

---

## Other Important Architectural Principles and Patterns

The 12-Factor methodology focuses on application development and deployment. Additional principles and patterns help solve microservices-specific design problems.

### 17.1 Domain-Driven Design (DDD)

Domain-Driven Design helps identify business boundaries and model software around the business domain.

A **bounded context** defines a boundary within which a particular domain model and its terminology are consistent.

For example, the meaning of a customer in billing may differ from the meaning of a customer in marketing.

Bounded contexts can help identify service boundaries, but a bounded context does not automatically have to be a separately deployed microservice.

### 17.2 SOLID Principles

SOLID is a set of object-oriented design principles that improve maintainability and extensibility.

- **S — Single Responsibility Principle:** A class should have one cohesive reason to change.
- **O — Open/Closed Principle:** Software entities should be open for extension and closed for modification.
- **L — Liskov Substitution Principle:** Subtypes should be usable wherever their base types are expected.
- **I — Interface Segregation Principle:** Clients should not depend on interfaces they do not use.
- **D — Dependency Inversion Principle:** High-level policy should not depend directly on low-level implementation details.

SOLID primarily guides code and module design, while microservices principles also address deployment, network communication, and distributed data.

### 17.3 API Gateway Pattern

An API Gateway provides a common entry point for client requests.

It may handle routing, authentication integration, rate limiting, and request transformation.

However, it should not become a centralized implementation of all business logic.

### 17.4 Circuit Breaker Pattern

A circuit breaker temporarily stops calls to a failing dependency to prevent repeated failures from exhausting resources.

Typical states include:

- Closed: Requests are allowed.
- Open: Requests fail fast or use an appropriate fallback.
- Half-open: Limited requests test whether the dependency has recovered.

### 17.5 Saga Pattern

A Saga coordinates a business workflow across multiple services using local transactions and compensating actions when required.

For example, if a payment succeeds but order fulfillment subsequently fails, the workflow may initiate a refund or another business-specific compensation.

### 17.6 Transactional Outbox Pattern

The transactional outbox pattern records a business change and its corresponding event in the same local database transaction.

A separate publisher then sends the event to a message broker.

This reduces the risk of updating a database successfully but losing the corresponding event.

### 17.7 Idempotency

An operation is idempotent when repeating the same operation produces the same intended effect as executing it once.

This is particularly important for payment requests, retries, message consumers, and distributed workflows.

An idempotency key or durable deduplication record can prevent duplicate business operations.

### 17.8 Twelve-Factor App vs. Microservices Principles

| Aspect | 12-Factor App | Microservices design principles |
|---|---|---|
| Primary focus | Application portability and deployment | Service boundaries and distributed architecture |
| Configuration | Externalized configuration | Externalized configuration and secure secret management |
| State | Stateless processes | Resilient state management and service-owned data |
| Scaling | Process-based scaling | Independent service scaling |
| Communication | Port binding | APIs, messaging, and explicit contracts |
| Reliability | Disposability and environment practices | Timeouts, circuit breakers, bulkheads, and recovery |
| Data ownership | Not its primary focus | A key architectural concern |
| Observability | Logs as event streams | Logs, metrics, traces, and service-level monitoring |

---

## Common Interview Questions

### Q1. What are the most important microservices design principles?

Loose coupling, high cohesion, clear business boundaries, independent deployment, independent scalability, service-owned data, resilience, observability, automation, and security by design.

### Q2. What is the difference between a stateless service and a stateful service?

A stateless service instance does not rely on its own local memory to retain essential client or business state between requests.

A stateful business domain can still persist information in a database or another external resource.

### Q3. Why should each microservice own its database?

Data ownership prevents services from depending on each other's internal schemas and allows each service to evolve its persistence model independently.

Separate databases are a common implementation, but the essential principle is controlled ownership and access rather than the physical number of database servers.

### Q4. Why is loose coupling important?

It reduces the need for coordinated changes. When one service changes its internal implementation without breaking its contract, dependent services can continue working without being modified.

### Q5. What is the difference between high availability and resilience?

High availability focuses on maintaining service availability and minimizing downtime.

Resilience focuses on handling failures, degrading gracefully, and recovering safely. Resilience contributes to availability, but the concepts are not identical.

### Q6. How do you prevent cascading failures?

Use bounded timeouts, circuit breakers, bulkheads, carefully limited retries, load shedding, rate limiting, and appropriate fallbacks.

Monitor resource consumption and downstream health as well.

### Q7. How do you externalize configuration in Spring Boot?

Use environment variables, configuration files supplied at deployment time, or a configuration management system. Store sensitive credentials in a suitable secret manager rather than source control.

### Q8. How do the 12 factors help microservices?

They encourage explicit dependencies, portable deployments, externalized configuration, disposable processes, process-based scaling, environment parity, and consistent logging.

They improve application deployability but do not eliminate distributed-system failure modes.

### Q9. What is the difference between Factor 5 and Factor 9?

Factor 5 separates building an artifact, creating a release, and running it.

Factor 9 focuses on making processes disposable through fast startup and graceful shutdown.

### Q10. Is the 12-Factor methodology mandatory for every microservice?

No. It is a methodology rather than a mandatory specification. Its principles are useful defaults, but implementation choices should account for the application, platform, security requirements, and operational environment.

### Q11. Does every microservice need a separate repository?

No. A system can use a monorepo or multiple repositories. The important consideration is whether ownership, builds, testing, and deployments can be managed effectively.

### Q12. How do you make microservices independently deployable?

Define stable contracts, maintain backward compatibility, automate builds and tests, manage database changes safely, and avoid release-time dependencies between unrelated services.

### Q13. How would you design a reliable payment service?

Use idempotency keys, durable transaction records, strict timeouts, secure credential handling, reconciliation workflows, appropriate retry rules, audit logging, and observability.

Avoid blindly retrying a payment when the outcome is unknown. First establish whether the original operation succeeded.

### Q14. How do you choose microservice boundaries?

Start with business capabilities and domain boundaries. Group cohesive functionality, define ownership, and minimize chatty communication and cross-service transactions. Validate the boundaries against actual business workflows.

---

## Quick Revision Notes

- **Loose coupling:** Minimize dependencies between services.
- **High cohesion:** Keep related business functionality together.
- **Service autonomy:** Enable teams to develop and deploy independently.
- **Data ownership:** Each service controls its business data.
- **Stateless processes:** Avoid relying exclusively on local memory for durable state.
- **Resilience:** Handle failures safely and recover predictably.
- **Observability:** Use logs, metrics, and traces.
- **Externalized configuration:** Keep environment-specific values outside application code.
- **Disposability:** Support fast startup and graceful shutdown.
- **Port binding:** Expose service functionality through a network port.
- **Process-based scaling:** Add instances to handle additional workload.
- **Domain-Driven Design:** Use domain models and bounded contexts to guide boundaries.
- **Circuit breaker:** Limit repeated calls to failing dependencies.
- **Saga:** Coordinate workflows across service-owned transactions.
- **Transactional outbox:** Reliably publish events associated with local database changes.
- **Idempotency:** Prevent duplicate business effects during retries.

## Final Recommendation

When designing production-grade Java and Spring Boot microservices, prioritize clear domain boundaries, stable contracts, service-owned data, secure configuration, and independent deployment.

Then build operational reliability around those boundaries with automated CI/CD, health checks, structured logging, metrics, distributed tracing, bounded retries, timeouts, and graceful shutdown.

The most important lesson is that **microservices are not simply small applications; they are independently managed components of a distributed system.** Good design must account for both software structure and the realities of networks, failures, data consistency, security, and operations.


