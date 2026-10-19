---
title: System Design Fundamentals
parent: System Design
nav_order: 1
---

# System Design Fundamentals
{: .no_toc }

Scalability, distributed systems, caching, databases, reliability, and performance.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

# System Design Fundamentals: A Complete Guide

System Design is the process of deciding how software components, databases, networks, and infrastructure work together to satisfy functional and non-functional requirements.

For example, designing a URL shortener requires decisions about:

- How users create and access short URLs.
- How millions of requests are processed.
- How short codes are generated without collisions.
- How data is stored and retrieved efficiently.
- How the system handles traffic spikes and server failures.
- How the application maintains availability and acceptable response times.

A strong System Design solution is not simply a collection of technologies. It is a set of architectural decisions justified by requirements, constraints, and trade-offs.

This guide covers the fundamentals required for backend engineering, distributed systems, and senior software engineering interviews, with practical Java examples.

## 1. Understanding System Design

System Design defines the architecture, components, interfaces, data models, communication patterns, and operational behavior of an application.

Consider an online shopping application.

A simplified architecture looks like this:

```text
                 +------------------+
                 | Web / Mobile App |
                 +--------+---------+
                          |
                          v
                 +------------------+
                 | Load Balancer    |
                 +--------+---------+
                          |
                          v
                 +------------------+
                 | Spring Boot API  |
                 +--------+---------+
                          |
              +-----------+-----------+
              |                       |
              v                       v
       +-------------+         +-------------+
       | Redis Cache |         | PostgreSQL  |
       +-------------+         +-------------+
```

The client sends a request. The load balancer forwards it to an application instance. The application executes business logic, retrieves data from the cache or database, and returns a response.

In a production system, additional components might include an API gateway, message broker, authentication service, distributed tracing, and a content delivery network (CDN).

### What does a system designer decide?

1. **Architecture:** Monolith, modular monolith, microservices, or event-driven architecture.
2. **Data storage:** SQL, NoSQL, object storage, or search indexes.
3. **Scalability:** Vertical scaling, horizontal scaling, replication, and sharding.
4. **Performance:** Caching, indexing, batching, and asynchronous processing.
5. **Reliability:** Timeouts, retries, circuit breakers, redundancy, and disaster recovery.
6. **Security:** Authentication, authorization, encryption, and secret management.
7. **Operations:** Logging, metrics, tracing, deployment, and alerting.

**Interview principle:** Every major design choice should answer a specific requirement or solve an identified problem.

## 2. Functional and Non-Functional Requirements

Before designing a system, identify what it must do and how well it must perform.

### Functional requirements

Functional requirements describe the features provided by the system.

For an online shopping application:

- Users can register and log in.
- Users can search for products.
- Users can add products to a cart.
- Users can place orders.
- Users can make payments.
- Users can view order history.

### Non-functional requirements

Non-functional requirements describe system qualities and constraints.

| Requirement | Meaning | Example |
|---|---|---|
| Scalability | Handles increasing load | Supports increasing traffic by adding instances |
| Availability | Remains usable | Target 99.99% availability |
| Latency | Time required to respond | 95% of product reads complete within 200 ms |
| Throughput | Requests processed per unit of time | 10,000 requests per second |
| Durability | Data survives failures | Confirmed orders are not lost |
| Consistency | Reads obey defined data guarantees | Payment status is read according to the required consistency model |
| Security | Protects data and operations | Only authorized users can access their orders |
| Maintainability | Easy to modify and operate | Clear modules, tests, and deployment processes |

### Availability calculation

Availability is commonly expressed as:

\[
Availability = \frac{Uptime}{Total\ Time} \times 100
\]

For a 30-day month:

| Availability | Approximate downtime per 30 days |
|---|---:|
| 99% | 7 hours 12 minutes |
| 99.9% | 43 minutes 12 seconds |
| 99.99% | 4 minutes 19 seconds |
| 99.999% | 26 seconds |

These are approximate values, assuming continuous operation over a 30-day period.

### SLA, SLO, and SLI

- **SLI (Service Level Indicator):** A measured value, such as the percentage of successful requests.
- **SLO (Service Level Objective):** An internal target, such as 99.9% successful requests over 30 days.
- **SLA (Service Level Agreement):** A formal agreement that may define consequences if specified service commitments are missed.

For example, an API might have an SLO requiring 99.9% successful requests and a p95 latency below 200 ms.

## 3. Capacity Estimation

Capacity estimation helps determine whether a proposed architecture can handle the expected workload.

You do not need exact numbers during an interview. Reasonable assumptions and transparent calculations are sufficient.

### Example: Estimate requests per second

Suppose an application has 1 million daily active users.

Assume each user makes 20 API requests per day.

Total daily requests:

\[
1,000,000 \times 20 = 20,000,000
\]

Average requests per second:

\[
RPS = \frac{20,000,000}{86,400}
\approx 231
\]

If peak traffic is five times the average:

\[
Peak\ RPS \approx 231 \times 5 = 1,155
\]

The system should be tested against approximately 1,155 requests per second under these assumptions, with additional headroom based on latency and availability targets.

### Storage estimation

Suppose the application stores 10 million records, each averaging 2 KB.

\[
Storage = 10,000,000 \times 2\ KB
\]

Approximately 20 GB of raw data is required using decimal units.

The actual storage requirement will be higher after accounting for indexes, metadata, replication, backups, and growth.

### Bandwidth estimation

If the average response is 50 KB and the application serves 1,000 responses per second:

\[
Bandwidth = 1,000 \times 50\ KB
\]

Approximately 50 MB/s, or 400 Mbps, of outbound application payload is required before protocol overhead.

### Important capacity terms

- **Average load:** Typical traffic over a period.
- **Peak load:** Maximum traffic expected during a busy interval.
- **Headroom:** Extra capacity reserved for spikes and failures.
- **Hotspot:** A component or data partition receiving disproportionately high traffic.
- **Bottleneck:** The component limiting overall system performance.

## 4. Core Components of a Distributed System

### 4.1 Client

A client is an application that sends requests to a backend.

Examples include:

- Web browsers.
- Android and iOS applications.
- Desktop applications.
- Other backend services.

Clients should not be trusted to enforce security or business rules. The server must validate input, authenticate users, and authorize operations.

### 4.2 DNS

The Domain Name System translates a domain name into information needed to reach a server.

For example:

```text
api.example.com
        |
        v
      DNS
        |
        v
Load balancer IP or endpoint
```

DNS can direct users toward regional endpoints or CDN infrastructure. DNS caching means changes may not take effect immediately everywhere.

### 4.3 CDN

A Content Delivery Network serves cached content from geographically distributed edge locations.

Typical CDN content includes:

- Images.
- CSS and JavaScript files.
- Videos.
- Public, cacheable API responses.

A CDN reduces the distance between the user and cached content. It does not automatically accelerate every dynamic API request.

### 4.4 Load balancer

A load balancer distributes incoming traffic across healthy application instances.

```text
                 Client Requests
                        |
                        v
                +---------------+
                | Load Balancer |
                +-------+-------+
                        |
              +---------+---------+
              |         |         |
              v         v         v
          +-------+ +-------+ +-------+
          | App 1 | | App 2 | | App 3 |
          +-------+ +-------+ +-------+
```

Benefits:

- Distributes traffic.
- Supports horizontal scaling.
- Removes unhealthy instances from rotation.
- Can terminate TLS connections.
- Can provide session affinity when required.

A load balancer does not make an application highly available if all application instances depend on one unavailable database or one unreplicated infrastructure component.

### 4.5 API gateway

An API gateway is a managed entry point for backend APIs.

Common responsibilities include:

- Authentication integration.
- Request routing.
- Rate limiting.
- TLS termination.
- Request and response transformation.
- API version routing.
- Centralized access policies.

A gateway and a load balancer can coexist.

| Component | Primary responsibility |
|---|---|
| Load balancer | Distributes traffic among healthy targets |
| API gateway | Applies API-level routing and policies |
| Reverse proxy | Forwards requests to upstream servers |
| Service discovery | Helps services locate other service instances |

Their features can overlap depending on the product and architecture.

### 4.6 Application server

The application server executes business logic.

A Spring Boot application might:

1. Validate a request.
2. Authenticate and authorize the user.
3. Execute business rules.
4. Access a cache or database.
5. Publish an event if necessary.
6. Return an HTTP response.

Stateless application instances are generally easier to scale because any healthy instance can process a request.

Avoid storing essential session state only in an individual instance when requests may be routed to different instances.

### 4.7 Database

A database stores persistent application data.

Examples:

- PostgreSQL for orders and financial transactions.
- MongoDB for document-oriented data.
- Cassandra for suitable large-scale distributed workloads.
- DynamoDB for managed key-value and document access patterns.

Database selection depends on access patterns, consistency requirements, query complexity, operational expertise, and cost—not simply whether the system is large.

### 4.8 Cache

A cache stores reusable data in a faster storage layer.

Redis is commonly used as a distributed cache. An in-process cache such as Caffeine can provide even faster access within an individual application instance.

### 4.9 Message broker

A message broker enables asynchronous communication.

Examples include Apache Kafka and RabbitMQ.

```text
Order Service
      |
      | Publish OrderCreated
      v
   Message Broker
      |
      +-----------> Email Consumer
      |
      +-----------> Inventory Consumer
      |
      +-----------> Analytics Consumer
```

The order service can publish an event without waiting for every consumer to complete its work.

Asynchronous communication reduces coupling, but it introduces delivery guarantees, retries, duplicate processing, ordering, and monitoring requirements.

## 5. Request Lifecycle

Consider a user requesting product details.

```text
Client
  |
  v
DNS / CDN (when applicable)
  |
  v
Load Balancer / API Gateway
  |
  v
Spring Boot Application
  |
  v
Authorization and Business Logic
  |
  v
Redis Cache
  |
  +---- Cache hit ------> Return cached result
  |
  +---- Cache miss -----> Query database
                               |
                               v
                         Store in cache
                               |
                               v
                         Return response
```

A typical cache-aside read works as follows:

1. The client sends `GET /api/products/101`.
2. The load balancer selects a healthy application instance.
3. The application validates access permissions.
4. The application checks Redis for product `101`.
5. If the key exists, the cached value is returned.
6. If the key does not exist, the application queries the database.
7. The application stores the result in Redis with a suitable expiration time.
8. The application returns the response.

A write usually follows a different sequence: update the authoritative database and then invalidate or refresh the cache.

The correct sequence depends on the consistency requirements and failure modes.

## 6. Monolithic vs. Microservices Architecture

### Monolithic architecture

A monolithic application packages its business functionality into one deployable application.

```text
+-----------------------------------+
|       Spring Boot Application     |
|                                   |
|  User Module                      |
|  Product Module                   |
|  Order Module                     |
|  Payment Module                   |
+----------------+------------------+
                 |
                 v
             Database
```

Advantages:

- Simple development and deployment.
- Easy local testing.
- Straightforward transactions within one database.
- Lower operational overhead.
- Convenient refactoring across modules.

Disadvantages:

- The application may become difficult to maintain without modular boundaries.
- A deployment can affect the entire application.
- Individual modules cannot be independently deployed as separate services.
- Scaling the whole application may waste resources.

A monolith can still be horizontally scaled by running multiple application instances behind a load balancer.

### Microservices architecture

Microservices split an application into independently deployable services organized around business capabilities.

```text
                 API Gateway
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
     User Service Product Service Order Service
          |           |           |
          v           v           v
       User DB     Product DB    Order DB
                                  |
                                  v
                              Event Broker
```

Advantages:

- Independent deployment.
- Independent scaling.
- Team ownership around business capabilities.
- Technology choices can vary when justified.
- Failure isolation can be improved with appropriate boundaries.

Disadvantages:

- Network failures and latency.
- Distributed transactions become more difficult.
- More complex debugging and observability.
- Service discovery and configuration management.
- Increased deployment and infrastructure costs.
- Versioning and backward compatibility requirements.

**Important:** Microservices do not automatically make a system scalable or reliable. Poor service boundaries can create a distributed monolith.

### Modular monolith

A modular monolith keeps one deployable application while enforcing clear internal boundaries.

It is often an effective starting point because it offers simpler operations while allowing the codebase to evolve.

### When should you choose each?

| Situation | Suitable starting point |
|---|---|
| Small team and early product | Modular monolith |
| Strong domain boundaries and independent release needs | Microservices |
| Different modules need very different scaling | Microservices may help |
| Cross-module transactions dominate the workflow | A monolith may be simpler |
| High operational maturity and independent teams | Microservices become more practical |

## 7. Vertical and Horizontal Scaling

### Vertical scaling

Vertical scaling means increasing the resources of one machine.

For example:

```text
Before:
4 CPU cores, 8 GB RAM

After:
16 CPU cores, 64 GB RAM
```

Advantages:

- Simple architecture.
- Minimal application changes.
- No additional application-to-application communication.

Disadvantages:

- Hardware limits.
- Potentially expensive upgrades.
- Scaling often requires infrastructure changes.
- A single instance may remain a point of failure.

### Horizontal scaling

Horizontal scaling means adding more machines or application instances.

```text
                Load Balancer
                     |
          +----------+----------+
          |          |          |
          v          v          v
        App 1      App 2      App 3
```

Advantages:

- More aggregate capacity.
- Improved resilience when instances fail.
- Independent scaling of stateless application instances.

Disadvantages:

- Distributed state and coordination become more complex.
- Network communication introduces failure modes.
- Shared databases and other dependencies can become bottlenecks.

### Stateless vs. stateful applications

A stateless request does not depend on essential session state stored only in the application instance.

For example, an application can validate a signed access token and retrieve user data from shared storage.

A stateful component retains session or other state that subsequent operations depend on.

Stateful components can also be scaled, but their storage, routing, replication, and failover require additional planning.

**Interview tip:** Stateless application instances, a shared database, suitable caching, and a load balancer form a common starting point for a horizontally scalable web application.

## 8. Load Balancing Algorithms

### Round Robin

Distributes requests across servers in rotation.

```text
Request 1 -> Server A
Request 2 -> Server B
Request 3 -> Server C
Request 4 -> Server A
```

Best when servers have similar capacity and requests have roughly similar processing costs.

### Weighted Round Robin

Assigns more traffic to higher-capacity servers.

For example, a server with weight 3 receives approximately three times as many selections as a server with weight 1 under a simple weighted distribution.

### Least Connections

Selects the server with the fewest active connections.

Useful when requests have different durations and connections remain open for different lengths of time.

### Least Response Time

Selects a server using a latency-related measure, often combined with connection counts or other health information.

The exact algorithm depends on the load balancer implementation.

### IP Hash

Uses a client IP address or related hash to select a server.

It can provide a degree of affinity but does not guarantee that every user's requests remain on one server in all configurations. NAT, changing addresses, and membership changes affect the mapping.

### Health checks

Health checks allow a load balancer or orchestrator to identify instances that should not receive traffic.

A good design distinguishes:

- **Liveness:** Is the process alive?
- **Readiness:** Is the instance ready to serve requests?
- **Dependency health:** Are critical dependencies available?

A temporary database problem should not necessarily cause every application instance to fail its liveness check and restart continuously.

## 9. Database Fundamentals

Database design should begin with the data model and expected access patterns.

### Relational databases

Examples: PostgreSQL and MySQL.

Use relational databases when you need:

- Structured schemas and relationships.
- SQL queries and joins.
- Transactions across related records.
- Constraints that enforce data integrity.
- Strong consistency for important business operations.

Example order schema:

```sql
CREATE TABLE orders (
    id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    status VARCHAR(30) NOT NULL,
    total_amount NUMERIC(12, 2) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_orders_customer_created
    ON orders (customer_id, created_at DESC);
```

The composite index can support queries filtering by `customer_id` and sorting by `created_at` in descending order. Actual performance should be verified with the database query planner.

### NoSQL databases

NoSQL databases include key-value, document, wide-column, and graph databases.

Examples:

- Redis: key-value and data structures.
- MongoDB: document-oriented data.
- Cassandra: wide-column distributed storage.
- DynamoDB: managed key-value and document storage.
- Neo4j: graph-oriented storage.

Choose a database according to the queries and consistency guarantees you need.

NoSQL does not automatically mean faster, more scalable, or eventually consistent. Different products support different consistency and transaction models.

### SQL vs. NoSQL

| Factor | Relational database | NoSQL database |
|---|---|---|
| Data model | Tables and relationships | Depends on database type |
| Querying | SQL, joins, aggregations | Product-specific query model |
| Schema | Typically explicitly defined | May be flexible or application-defined |
| Transactions | Mature ACID support | Varies by product |
| Scaling | Vertical and horizontal options | Horizontal scaling often emphasized |
| Best choice | Relational data and complex queries | Access patterns suited to the specific NoSQL model |

### ACID properties

ACID describes transaction guarantees.

- **Atomicity:** A transaction succeeds completely or rolls back.
- **Consistency:** A transaction preserves defined data integrity rules.
- **Isolation:** Concurrent transactions behave according to the selected isolation level.
- **Durability:** A committed transaction survives failures covered by the database's durability guarantees.

### BASE terminology

BASE is a broad model often associated with highly available distributed data systems:

- Basically Available.
- Soft State.
- Eventual Consistency.

It is not a universal specification for all NoSQL databases. Always check the actual product's guarantees.

## 10. Indexing

An index is an auxiliary data structure that helps a database find records without scanning every row.

### Without an index

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

If no useful index exists, the database may scan a large portion or all of the table.

### With an index

```sql
CREATE INDEX idx_orders_customer
ON orders (customer_id);
```

The database can use the index to locate matching records more efficiently.

### Common index types

**B-tree index**

Commonly used for equality and range queries.

```sql
CREATE INDEX idx_orders_created_at
ON orders (created_at);
```

Useful for conditions such as:

```sql
SELECT *
FROM orders
WHERE created_at >= TIMESTAMP '2026-01-01 00:00:00';
```

**Composite index**

An index covering multiple columns.

```sql
CREATE INDEX idx_orders_customer_status
ON orders (customer_id, status);
```

Column order matters. An index beginning with `customer_id` is generally more suitable for lookups using that leading column than for lookups using only `status`.

**Unique index**

Enforces uniqueness for the indexed key or key combination.

```sql
CREATE UNIQUE INDEX idx_users_email
ON users (email);
```

This is stronger than checking for duplicate email addresses only in Java because concurrent requests may pass an application-level check at the same time.

**Inverted index**

Maps terms to documents and is commonly used by search engines.

**Trie**

A prefix tree used in applications such as autocomplete. It is an in-memory data structure rather than a standard relational database index type.

### Index trade-offs

Indexes consume disk space and increase the cost of inserts, updates, and deletes.

Avoid creating indexes for every column. Use query plans, execution statistics, and representative workloads to decide which indexes are useful.

## 11. Replication

Replication maintains copies of data across multiple database nodes.

### Primary-replica replication

```text
           Application
                |
                v
          +-----------+
          |  Primary  |
          +-----+-----+
                |
          Replication
             stream
           /       \
          v         v
    +-----------+ +-----------+
    | Replica 1 | | Replica 2 |
    +-----------+ +-----------+
```

In a common primary-replica setup:

- Writes go to the primary.
- Changes are replicated to replicas.
- Reads may be served by replicas when the consistency requirements permit.

Advantages:

- More capacity for suitable read workloads.
- Additional copies of data.
- Potentially faster regional reads.

Challenges:

- Replication lag.
- Failover coordination.
- Stale reads from replicas.
- Additional storage and operational complexity.

Replication does not automatically guarantee zero data loss. The result depends on acknowledgement rules, replication mode, failover behavior, and recovery configuration.

### Synchronous vs. asynchronous replication

**Synchronous replication:** A write waits for acknowledgement from the configured replication participants. This can provide stronger durability or consistency guarantees, depending on the database's implementation, but may increase latency and reduce write availability.

**Asynchronous replication:** The primary acknowledges the write before every replica has received it. This can reduce latency but creates a window in which replicas are behind and acknowledged writes may be lost in certain failover scenarios.

## 12. Sharding and Partitioning

Partitioning divides data into smaller logical or physical parts.

Sharding commonly refers to distributing those partitions across multiple database nodes.

### Example: User-based sharding

Suppose a user table contains hundreds of millions of records.

```text
User ID
   |
   v
Shard Selection
   |
   +---- User IDs in range A ---> Shard 1
   |
   +---- User IDs in range B ---> Shard 2
   |
   +---- User IDs in range C ---> Shard 3
```

A hash-based strategy might use:

```java
public final class ShardRouter {

    private ShardRouter() {
    }

    public static int getShard(long userId, int shardCount) {
        if (shardCount <= 0) {
            throw new IllegalArgumentException(
                "shardCount must be positive"
            );
        }

        return Math.floorMod(Long.hashCode(userId), shardCount);
    }
}
```

Usage:

```java
int shard = ShardRouter.getShard(12345L, 4);
System.out.println("Shard: " + shard);
```

This is an educational routing example, not a complete production sharding implementation. Modulo-based routing can remap many keys when the shard count changes. Production designs may use consistent hashing, virtual partitions, or database-managed partitioning.

### Choosing a shard key

A good shard key:

- Distributes data reasonably evenly.
- Appears frequently in important queries.
- Minimizes cross-shard operations.
- Avoids concentrating writes on one shard.

### Common sharding problems

- Hot partitions.
- Cross-shard joins.
- Distributed transactions.
- Uneven data distribution.
- Rebalancing when nodes are added or removed.
- More complex backups and operations.

**Important:** Sharding is not the first solution for every slow database. Query optimization, indexing, caching, and vertical scaling may be sufficient.

## 13. CAP Theorem

The CAP theorem describes a fundamental trade-off in a distributed data system when a network partition prevents nodes from communicating reliably.

CAP stands for:

- **Consistency (C):** In the CAP sense, reads reflect the latest successful write or return an error, according to the model's guarantees.
- **Availability (A):** Every request received by a non-failing node gets a non-error response, though that response may not reflect the latest write.
- **Partition tolerance (P):** The system continues operating despite messages being lost or delayed between parts of the system.

When a network partition occurs, a distributed system cannot guarantee both CAP consistency and CAP availability for every request.

### CP system

A CP-oriented system prioritizes consistency during a partition and may reject or delay some requests.

Suitable for operations where conflicting or stale decisions would be unacceptable, provided the actual database's guarantees meet the requirement.

### AP system

An AP-oriented system continues responding during a partition, accepting that some responses may contain stale data or that conflicts may need resolution.

Suitable for certain distributed catalogs, feeds, or other workflows where temporary inconsistency is acceptable.

### Important distinction

CAP does not mean every database must permanently choose only two properties. When there is no partition, a system may provide both consistency and availability. The trade-off becomes unavoidable during a partition.

Also, do not assume that every banking application is CP or every social feed is AP. A single application may use different consistency requirements for different operations.

For example:

- Account balance updates may require strict coordination.
- Product recommendations can tolerate stale data.
- A shopping cart may use application-specific conflict resolution.
- Analytics can often accept delayed updates.

## 14. Consistency Models

Consistency models describe what users can observe when data is read and written.

### Strong consistency

After a successful write, subsequent reads under the defined consistency model observe that write or a later write.

Useful for operations such as inventory allocation and financial state transitions where stale data could lead to incorrect decisions.

### Eventual consistency

If no new updates occur and replication continues successfully, replicas eventually converge.

A user may briefly see stale data.

Example:

```text
Time 1: User updates profile name.
Time 2: Primary stores the new name.
Time 3: Replica still contains the old name.
Time 4: Replication catches up.
Time 5: Replica returns the new name.
```

### Read-your-writes consistency

A user who updates a value can subsequently read their own update, even if other users or replicas have not yet observed it.

### Choosing the right model

| Use case | Typical requirement |
|---|---|
| Account balance | Strongly coordinated updates |
| Product description | Briefly stale reads may be acceptable |
| Social feed | Eventual consistency may be acceptable |
| Analytics dashboard | Delayed aggregation is often acceptable |
| Inventory reservation | Atomic reservation or another safe concurrency strategy |

Consistency must be defined for a specific operation and data store, not chosen only at the application level.

## 15. Caching Strategies

Caching stores frequently accessed data in a faster storage layer.

It reduces database load and response latency, but introduces cache invalidation, staleness, memory limits, and failure-handling concerns.

### Types of caches

**In-process cache**

Stored within the application instance.

Example: Caffeine.

Advantages:

- Very low latency.
- No network call to retrieve an entry.

Disadvantages:

- Each instance has its own copy.
- Entries may differ between instances.
- Memory is limited to the application process.

**Distributed cache**

Shared across multiple application instances.

Example: Redis.

Advantages:

- Shared cache entries.
- Supports distributed deployments.
- Offers useful structures, expiration, and atomic operations.

Disadvantages:

- Adds network latency.
- Requires cache availability and capacity planning.
- Can become a bottleneck if poorly configured.

**CDN cache**

Stores suitable content near users at edge locations.

Useful for public images, scripts, and other cacheable content.

### Cache-aside

The application manages cache lookups and database access.

```java
public Product getProduct(long id) {
    Product cached = cache.get(id);

    if (cached != null) {
        return cached;
    }

    Product product = repository.findById(id)
        .orElseThrow(() ->
            new ProductNotFoundException(id)
        );

    cache.put(id, product);
    return product;
}
```

This example assumes a cache abstraction with `get` and `put` methods. In production, configure expiration, serialization, failure behavior, and cache-miss handling.

Flow:

1. Read from cache.
2. On a miss, read from the database.
3. Populate the cache.
4. Return the result.

### Write-through cache

The write path updates the cache and the underlying data store synchronously through the cache layer or configured integration.

The exact ordering and atomicity guarantees depend on the implementation. A write-through design can keep reads warm, but it does not automatically make the cache and database one atomic transaction.

### Write-behind cache

The application writes to the cache, which persists changes to the underlying store asynchronously.

This can reduce write latency but creates durability and data-loss risks if pending writes are lost. It is appropriate only when the persistence guarantees are understood and acceptable.

### Cache invalidation

When the source data changes, the cache must be refreshed, updated, or invalidated.

A common approach is:

```text
1. Update database.
2. Invalidate the cache entry.
3. Let the next read populate fresh data.
```

If the database update succeeds but cache invalidation fails, stale data can remain until expiration or another repair mechanism runs.

### TTL

TTL means Time To Live.

For example, a product description may be cached for five minutes, while a rapidly changing stock value may need a shorter TTL or a different consistency strategy.

The appropriate TTL depends on the business requirement. A shorter TTL reduces the duration of some stale entries but increases cache misses and database traffic.

### Cache stampede

A cache stampede occurs when many requests miss the same key simultaneously and all query the database.

Common mitigations:

- Request coalescing or single-flight loading.
- Randomized TTLs.
- Short-lived stale responses where safe.
- Background refresh.
- Per-key synchronization.

### Cache penetration

Repeated requests for nonexistent records can bypass the cache and overload the database.

Possible mitigations include negative caching with a short TTL, input validation, and rate limiting.

Negative caching must distinguish a genuine not-found result from a temporary database or network failure.

## 16. Cache Eviction Policies

When a cache reaches its capacity, it may need to evict entries.

| Policy | Behavior | Useful when |
|---|---|---|
| LRU | Removes the least recently used entry | Recently accessed items are likely to be accessed again |
| LFU | Removes the least frequently used entry | Frequently accessed items are more valuable |
| FIFO | Removes the oldest inserted entry | Simplicity is preferred |
| TTL expiration | Expires entries after a configured duration | Data has a known freshness window |
| Random | Removes a randomly selected entry | Simple eviction is acceptable in a particular workload |

TTL expiration and capacity-based eviction solve different problems. An entry can expire by TTL or be evicted earlier because the cache needs space.

Redis supports configurable eviction policies, including LRU-like and LFU-like approaches. These policies generally use approximations rather than tracking every possible access with perfect accuracy.

## 17. Latency and Throughput

### Latency

Latency is the time taken to complete an operation.

Examples:

- Database query: 15 ms.
- Application processing: 10 ms.
- Network and gateway overhead: 20 ms.

If these stages execute sequentially, the total is approximately 45 ms.

In a real distributed system, the total latency depends on which operations execute sequentially or in parallel, queueing delays, retries, and tail latency.

### Throughput

Throughput is the number of operations completed per unit of time.

Examples:

- 500 requests per second.
- 10,000 messages per second.
- 2,000 database writes per second.

### Percentile latency

Average latency alone can hide slow requests.

- **p50:** Half of requests complete at or below this latency.
- **p95:** 95% complete at or below this latency.
- **p99:** 99% complete at or below this latency.

Example:

```text
p50 = 30 ms
p95 = 120 ms
p99 = 450 ms
```

Most requests are fast, but the slowest 1% take much longer.

### Little's Law

For a stable system:

\[
L = \lambda W
\]

Where:

- \(L\) is the average number of operations in the system.
- \(\lambda\) is the average arrival or throughput rate.
- \(W\) is the average time spent in the system.

If a service processes 200 requests per second and the average time in the system is 0.1 seconds:

\[
L = 200 \times 0.1 = 20
\]

The average number of in-flight requests is approximately 20 under these assumptions.

### Performance optimization

1. Measure latency and throughput.
2. Identify the slowest component.
3. Optimize the relevant query, code path, or network interaction.
4. Add caching or batching when appropriate.
5. Load-test the change.
6. Verify correctness, resource usage, and tail latency.

Avoid optimizing a component merely because it looks expensive. Measure the actual bottleneck.

## 18. Reliability and Fault Tolerance

A reliable system handles expected failures without losing correctness or becoming entirely unavailable.

### Redundancy

Run multiple instances of critical components so one failure does not stop the entire service.

Redundancy is only effective if replicas do not share the same unprotected failure domain.

### Timeouts

A timeout bounds how long an operation waits for a response.

Always configure sensible timeouts for outbound network calls, database operations, and remote service requests.

Without timeouts, requests may occupy threads and connections while waiting indefinitely.

### Retries with exponential backoff

A retry can help when a failure is temporary.

Exponential backoff increases the delay between attempts.

```java
public final class RetryDelay {

    private RetryDelay() {
    }

    public static long delayMillis(int attempt) {
        if (attempt < 1) {
            throw new IllegalArgumentException(
                "attempt must be at least 1"
            );
        }

        int exponent = Math.min(attempt - 1, 6);
        long baseDelay = 100L;
        long maximumDelay = 5_000L;

        return Math.min(
            baseDelay * (1L << exponent),
            maximumDelay
        );
    }
}
```

Example delays:

```text
Attempt 1: 100 ms
Attempt 2: 200 ms
Attempt 3: 400 ms
Attempt 4: 800 ms
Attempt 5: 1600 ms
```

This educational example does not include jitter or execute the retry itself. Production retry policies should normally include jitter to prevent synchronized retries.

Retry only when appropriate:

- The failure may be transient.
- The operation is safe to retry or protected by idempotency.
- The retry budget and deadline permit another attempt.

Do not blindly retry every HTTP error or repeat a payment operation without deduplication.

### Circuit breaker

A circuit breaker prevents repeated calls to a failing dependency.

States:

- **CLOSED:** Calls are allowed.
- **OPEN:** Calls are rejected or short-circuited.
- **HALF_OPEN:** A limited number of trial calls determine whether the dependency has recovered.

Example using Resilience4j with Spring Boot:

```java
@Service
public class InventoryClient {

    private final RestClient restClient;

    public InventoryClient(RestClient restClient) {
        this.restClient = restClient;
    }

    @CircuitBreaker(
        name = "inventoryService",
        fallbackMethod = "inventoryFallback"
    )
    public Inventory getInventory(long productId) {
        return restClient.get()
            .uri("/api/inventory/{id}", productId)
            .retrieve()
            .body(Inventory.class);
    }

    public Inventory inventoryFallback(
            long productId,
            Throwable exception) {

        throw new InventoryUnavailableException(
            "Inventory service is temporarily unavailable",
            exception
        );
    }
}
```

This illustrates the integration pattern; the RestClient bean, endpoint configuration, dependency, and circuit breaker settings must also be configured.

The fallback deliberately reports unavailability rather than inventing an inventory quantity. A business may instead return cached information or a clearly marked degraded response when that is safe.

### Bulkhead pattern

A bulkhead isolates resources so that one failing dependency does not consume all application capacity.

For example, separate connection pools or bounded thread pools can isolate payment calls from recommendation calls.

### Graceful degradation

If a non-critical service fails, the application should preserve its core functionality where possible.

Examples:

- Show a cached product description if recommendations are unavailable.
- Display a generic homepage if personalized recommendations fail.
- Queue an email notification instead of failing a completed order.

Never silently degrade a critical business guarantee, such as confirming an order without establishing that its required inventory reservation succeeded.

## 19. Rate Limiting

Rate limiting controls how many requests a client or resource can receive during a period.

It helps protect systems from abusive traffic, accidental request floods, and resource exhaustion.

### Common algorithms

**Fixed window**

Counts requests within a fixed time interval.

Simple, but requests can cluster around a window boundary.

**Sliding window**

Calculates usage over a moving interval. It provides smoother enforcement than a fixed window, depending on the implementation.

**Token bucket**

Tokens accumulate up to a configured capacity. Each request consumes a token.

It allows short bursts while limiting the long-term average rate.

**Leaky bucket**

Processes requests at a controlled rate or queues them within bounded capacity, depending on the implementation.

### Java token bucket example

```java
import java.time.Duration;

public final class TokenBucket {

    private final long capacity;
    private final double refillTokensPerNano;
    private double tokens;
    private long lastRefillNanos;

    public TokenBucket(
            long capacity,
            double tokensPerSecond) {

        if (capacity <= 0 || tokensPerSecond <= 0) {
            throw new IllegalArgumentException(
                "Capacity and refill rate must be positive"
            );
        }

        this.capacity = capacity;
        this.refillTokensPerNano =
            tokensPerSecond / 1_000_000_000.0;

        this.tokens = capacity;
        this.lastRefillNanos = System.nanoTime();
    }

    public synchronized boolean allowRequest() {
        long now = System.nanoTime();

        long elapsed = Math.max(
            0L,
            now - lastRefillNanos
        );

        tokens = Math.min(
            capacity,
            tokens + elapsed * refillTokensPerNano
        );

        lastRefillNanos = now;

        if (tokens >= 1.0) {
            tokens -= 1.0;
            return true;
        }

        return false;
    }
}
```

Usage:

```java
TokenBucket bucket = new TokenBucket(10, 5);

if (bucket.allowRequest()) {
    System.out.println("Request allowed");
} else {
    System.out.println("Rate limit exceeded");
}
```

This bucket allows a burst of up to 10 requests and refills at 5 tokens per second.

**Production limitation:** This example is thread-safe within one JVM, but every application instance maintains its own bucket. A distributed rate limit across multiple instances needs shared coordination, such as an atomic Redis operation or a suitable API gateway feature.

For HTTP APIs, a rate-limited request commonly receives status `429 Too Many Requests`, optionally with a `Retry-After` header.

## 20. Message Queues and Event-Driven Architecture

A message broker allows producers and consumers to communicate asynchronously.

### Synchronous communication

```text
Order Service
     |
     v
Email Service
     |
     v
Email Provider
     |
     v
Order Service returns
```

The order service may wait for downstream work to complete, increasing latency and coupling.

### Asynchronous communication

```text
Order Service
     |
     v
Message Broker
     |
     v
Email Consumer
     |
     v
Email Provider
```

The order service publishes an event, and the consumer processes it separately.

### Apache Kafka vs. RabbitMQ

| Feature | Kafka | RabbitMQ |
|---|---|---|
| Core model | Distributed event log | Message broker with exchanges and queues |
| Common use | Event streaming and replayable event pipelines | Task queues and flexible message routing |
| Retention | Events retained according to topic policy | Queue behavior depends on configuration and acknowledgement |
| Replay | Consumers can reread retained events | Different mechanisms; not the same general-purpose event-log model |
| Ordering | Within a partition | Depends on queue and consumer configuration |
| Delivery | Depends on producer, broker, consumer, and processing configuration | Depends on acknowledgement, durability, and consumer configuration |

Neither technology automatically guarantees exactly-once business processing.

### Java asynchronous processing example

Spring's `@Async` can run a method on a configured executor.

```java
@Configuration
@EnableAsync
public class AsyncConfiguration {

    @Bean("notificationExecutor")
    public Executor notificationExecutor() {
        ThreadPoolTaskExecutor executor =
            new ThreadPoolTaskExecutor();

        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(8);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("notification-");

        executor.initialize();
        return executor;
    }
}
```

```java
@Service
public class NotificationService {

    @Async("notificationExecutor")
    public CompletableFuture<Void> sendEmail(
            String email,
            String message) {

        // Call the email provider here.
        System.out.println(
            "Sending email to " + email
        );

        return CompletableFuture.completedFuture(null);
    }
}
```

The configuration demonstrates bounded concurrency and queue capacity. In production, handle exceptions, monitor rejected tasks, configure shutdown behavior, and avoid logging sensitive information.

`@Async` is not a durable message queue. Pending tasks may be lost during process termination. Use a durable broker when the business requires persistent delivery and recovery.

### Idempotent consumers

A message may be delivered more than once. Consumers should safely handle duplicate processing.

For example, store a unique event ID in a database table with a unique constraint:

```sql
CREATE TABLE processed_events (
    event_id VARCHAR(100) PRIMARY KEY,
    processed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

The consumer can atomically claim an event ID and apply its business update within the same transaction. A duplicate key indicates that the event has already been processed.

The exact transaction design must ensure that recording the event and applying the business effect cannot become inconsistent.

### Transactional outbox

A transactional outbox addresses the dual-write problem.

Without an outbox:

1. Save an order in the database.
2. Publish `OrderCreated` to Kafka.
3. The database commit succeeds but event publishing fails.

Now the order exists, but downstream services may never receive the event.

With an outbox, the application writes both the order and an outbox record in the same database transaction. A separate publisher delivers outbox events to the broker and marks them as published.

The publisher may send duplicates, so consumers still need idempotency.

## 21. Synchronous vs. Asynchronous Processing

| Synchronous | Asynchronous |
|---|---|
| Caller waits for the result | Caller can continue before processing finishes |
| Easier immediate error reporting | Failures may be discovered later |
| Suitable for immediate queries and decisions | Suitable for notifications, analytics, and background jobs |
| Adds downstream latency to the request | Requires queues, status tracking, or callbacks |
| Can tightly couple availability | Can improve isolation and absorb traffic spikes |

For example, an order API may synchronously validate and persist an order, then asynchronously send notifications and update analytics.

Do not make an operation asynchronous merely to improve response time if the user needs a confirmed business result before proceeding.

## 22. API Design

APIs define how clients and services communicate.

### REST example

Create an order:

```http
POST /api/v1/orders
Content-Type: application/json
Authorization: Bearer <access-token>
Idempotency-Key: 0db3c2a4-4ef8-4f0d-b0d9-000000000001
```

Request:

```json
{
  "productId": 101,
  "quantity": 2
}
```

Response:

```json
{
  "orderId": 9001,
  "status": "PENDING",
  "quantity": 2
}
```

A response should distinguish between an order being accepted, confirmed, paid, and shipped. These are different business states.

### HTTP methods

| Method | Typical purpose |
|---|---|
| GET | Retrieve a resource |
| POST | Create a resource or initiate an operation |
| PUT | Replace a resource representation |
| PATCH | Partially update a resource |
| DELETE | Delete a resource |

### Common HTTP status codes

| Status | Meaning |
|---|---|
| 200 | Request succeeded |
| 201 | Resource created |
| 202 | Request accepted for asynchronous processing |
| 204 | Request succeeded with no response body |
| 400 | Invalid request |
| 401 | Authentication required or invalid |
| 403 | Access forbidden |
| 404 | Resource not found |
| 409 | Conflict with current resource state |
| 429 | Too many requests |
| 500 | Unexpected server error |
| 503 | Service temporarily unavailable |

### Pagination

Offset pagination:

```http
GET /api/v1/orders?page=0&size=20
```

This is easy to implement but can become inefficient for very large offsets.

Cursor pagination:

```http
GET /api/v1/orders?limit=20&cursor=eyJpZCI6OTAwMX0
```

A cursor can encode the last-seen sort key. Cursor pagination is often more efficient for large, continuously changing datasets when supported by an appropriate index.

### Idempotency

An operation is idempotent if repeating it produces the same intended effect as performing it once.

For payment or order creation, an idempotency key helps prevent duplicate business operations when a client retries after a timeout.

The server must persist and validate the key, associate it with the correct request and result, and handle concurrent requests using the same key safely.

## 23. Security Fundamentals

Security must be designed into the architecture.

### Authentication vs. authorization

- **Authentication:** Establishes who the caller is.
- **Authorization:** Determines what the caller is allowed to do.

A valid JWT does not automatically mean the user can access every resource.

For example, a user should not access another customer's order merely by changing an order ID in the URL.

### Common controls

- Use HTTPS for communication.
- Validate input on the server.
- Apply authorization checks to individual resources.
- Store passwords using a suitable password hashing algorithm.
- Store secrets in a secret manager rather than source code.
- Apply least-privilege permissions.
- Avoid logging tokens, passwords, payment credentials, and unnecessary personal data.
- Protect against injection, cross-site scripting, and cross-site request forgery where applicable.
- Rotate credentials and restrict secret access.

### Secrets management

Do not hardcode database passwords or cloud credentials in Java source code or commit them to Git.

Use environment-specific configuration and a managed secret store, such as AWS Secrets Manager or Google Cloud Secret Manager.

## 24. Observability and Monitoring

Observability helps engineers understand system behavior from its outputs.

### Metrics

Numerical measurements collected over time.

Examples:

- Request rate.
- Error rate.
- p95 and p99 latency.
- CPU and memory utilization.
- Database connection pool usage.
- Cache hit ratio.
- Queue depth and consumer lag.

### Logs

Logs capture individual events.

Example:

```text
timestamp=2026-10-19T10:00:00Z
level=ERROR
service=order-service
traceId=abc123
message="Payment provider timeout"
```

Avoid logging sensitive data. Structured logs make searching and aggregation easier.

### Distributed tracing

Tracing follows a request across service boundaries.

```text
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
Database
```

A trace contains spans representing individual operations. Trace IDs allow engineers to correlate work across services.

### Common tools

- Prometheus for metrics collection.
- Grafana for dashboards.
- OpenTelemetry for instrumentation.
- Jaeger or compatible tracing backends for distributed traces.
- ELK/OpenSearch-based systems for log aggregation.

### Golden signals

Four widely used signals are:

1. **Latency:** How long requests take.
2. **Traffic:** How much demand the system receives.
3. **Errors:** How many requests fail.
4. **Saturation:** How close resources are to capacity.

Monitoring should include business metrics too, such as payment success rate and order completion rate.

## 25. Deployment and Infrastructure

Application architecture also includes how software is packaged, deployed, and recovered.

### Containers

Docker packages an application and its runtime dependencies into an image.

Containers improve portability but do not automatically provide high availability or orchestration.

### Kubernetes

Kubernetes manages containerized workloads and can provide:

- Scheduling.
- Service discovery.
- Rolling deployments.
- Restarting failed containers.
- Horizontal pod autoscaling when configured.
- Health probes and resource controls.

Kubernetes cannot fix incorrect application logic, an overloaded database, or a dependency that fails for every replica.

### Horizontal Pod Autoscaler

An HPA can scale the number of pods based on configured metrics. CPU utilization is one possible metric, but queue depth, request rate, and custom metrics may be more relevant for some workloads.

### Deployment strategies

**Rolling deployment:** Replaces instances gradually.

**Blue-green deployment:** Maintains two deployment environments and switches traffic between them.

**Canary deployment:** Sends a controlled portion of traffic to a new version before expanding the rollout.

A production deployment strategy should account for database schema compatibility, rollback safety, health checks, and application startup behavior.

## 26. Data Pipelines and Stream Processing

Data processing can be divided into batch and stream processing.

### Batch processing

Processes accumulated data periodically.

Examples:

- Daily financial reports.
- Historical analytics.
- Bulk data migration.
- Scheduled data exports.

Tools include Apache Spark and Hadoop.

### Stream processing

Processes events continuously as they arrive.

Examples:

- Fraud detection.
- Live dashboards.
- User activity aggregation.
- Real-time inventory event processing.

Tools include Kafka Streams and Apache Flink.

### Event ordering

Events may be ordered within a particular partition or processing context, but global ordering across a distributed pipeline is not automatic.

Choose partition keys and consumer concurrency based on the ordering requirements of the business.

### Event time vs. processing time

- **Event time:** When the event actually occurred.
- **Processing time:** When the system processed it.

These can differ because of network delays, retries, or offline clients.

## 27. Case Study: Design a Search Suggestion System

Suppose an e-commerce application needs to suggest products and search terms while users type.

### Step 1: Functional requirements

- Accept a search prefix.
- Return relevant suggestions.
- Rank suggestions.
- Support continuously updated product catalogs.
- Handle invalid or empty input.

### Step 2: Non-functional requirements

Assume:

- One million daily active users.
- Peak traffic significantly higher than average.
- p95 latency target below 100 ms.
- High availability.
- Frequent reads and less frequent catalog updates.

These are design assumptions, not universal requirements.

### Step 3: API design

```http
GET /api/v1/search/suggestions?q=lap&limit=10
```

Example response:

```json
{
  "query": "lap",
  "suggestions": [
    "laptop",
    "laptop stand",
    "laptop bag"
  ]
}
```

### Step 4: Architecture

```text
               Web / Mobile Client
                       |
                       v
                 CDN / Gateway
                       |
                       v
                 Load Balancer
                       |
                       v
                Suggestion Service
                       |
                       v
                  Redis Cache
                   /       \
               Hit          Miss
                |             |
                v             v
             Response    Search Index
                              |
                              v
                         Rank Results
                              |
                              v
                         Update Cache
                              |
                              v
                           Response
```

### Step 5: Storage and indexing

A search index may store product titles, descriptions, and popularity signals.

Possible data structures include:

- Trie for prefix matching.
- Inverted index for text retrieval.
- Search-engine indexes for ranking and filtering.
- Redis for frequently requested prefixes.

A trie is not always the best choice. A search engine may be more suitable when fuzzy matching, relevance ranking, filters, and distributed indexing are required.

### Step 6: Cache strategy

Cache popular prefixes, such as:

```text
lap  -> laptop, laptop stand, laptop bag
iph  -> iphone, iphone case, iphone charger
```

Use an expiration policy and refresh or invalidate entries after catalog updates.

Avoid allowing a single popular prefix to overload one cache shard. Partitioning, replication, and request coalescing may be needed.

### Step 7: Failure handling

- If Redis fails, use a controlled fallback to the search index.
- Apply timeouts to search requests.
- Limit request frequency.
- Return a graceful error or safe fallback if the search backend is unavailable.
- Monitor latency, cache hit ratio, search errors, and traffic distribution.

A cache failure should not cause uncontrolled traffic to overwhelm the database or search index. Fallback concurrency and rate limits must be considered.

## 28. Trade-offs in System Design

Every architecture has trade-offs.

| Decision | Benefit | Cost or limitation |
|---|---|---|
| More replicas | Better redundancy and possible read capacity | Higher cost and replication overhead |
| More caching | Lower latency and database load | Stale data and invalidation complexity |
| Stronger consistency | More predictable reads and writes | Coordination latency and possible reduced availability |
| Microservices | Independent deployment and scaling | Operational and communication complexity |
| Asynchronous processing | Lower request coupling | Eventual completion and more complicated failure handling |
| More indexes | Faster suitable queries | Slower writes and additional storage |
| Sharding | More distributed storage capacity | Routing, rebalancing, and cross-shard complexity |
| Larger machines | Simpler scaling path | Hardware limits and potentially high costs |

The right choice depends on workload, business correctness, service-level objectives, engineering maturity, and budget.

## 29. A Standard System Design Interview Framework

Use the following process for most interview problems.

### Step 1: Clarify requirements

Ask:

- Who uses the system?
- What are its core features?
- What is out of scope?
- What latency and availability targets matter?
- Is strong consistency required for any operation?

### Step 2: Estimate scale

Estimate:

- Daily active users.
- Requests per second.
- Read-to-write ratio.
- Data growth.
- Payload size.
- Peak traffic.
- Geographic distribution.

State your assumptions before calculating.

### Step 3: Define APIs

Identify the main operations and request/response contracts.

For example:

```http
POST /api/v1/orders
GET /api/v1/orders/{id}
GET /api/v1/users/{id}/orders
```

Consider authorization, pagination, idempotency, and error responses.

### Step 4: Design the data model

Identify:

- Main entities.
- Relationships.
- Primary and unique keys.
- Query patterns.
- Indexes.
- Transaction boundaries.

### Step 5: Draw the high-level architecture

Start simply:

```text
Client
  |
  v
Load Balancer
  |
  v
Application Service
  |
  +------> Cache
  |
  +------> Database
  |
  +------> Message Broker
```

Add components only when requirements justify them.

### Step 6: Identify bottlenecks

Ask:

- Can one application instance handle the traffic?
- Is the database CPU-, memory-, or I/O-bound?
- Are popular keys creating hotspots?
- Are network calls dominating latency?
- Can the queue keep up with producers?
- What happens if one dependency fails?

### Step 7: Improve reliability

Consider:

- Multiple application instances.
- Database replication and recovery.
- Timeouts and bounded retries.
- Circuit breakers.
- Rate limiting.
- Idempotent operations.
- Backups and disaster recovery.

### Step 8: Explain trade-offs

For each important decision, explain why it is suitable and what it costs.

For example:

"I would start with PostgreSQL because orders have relational constraints and transactional requirements. If measured data volume or workload exceeds a single database's capacity, I would first optimize queries and indexes, then consider read replicas, partitioning, or sharding based on the bottleneck."

This is stronger than simply stating that NoSQL is better for scale.

## 30. Common System Design Interview Questions

### Fundamentals

1. What is System Design?
2. What is the difference between functional and non-functional requirements?
3. What is the difference between latency and throughput?
4. What is vertical versus horizontal scaling?
5. What is a single point of failure?
6. What is the difference between availability and reliability?
7. What is the difference between an API gateway and a load balancer?
8. What are SLI, SLO, and SLA?

### Databases

9. SQL vs. NoSQL: how do you choose?
10. Explain ACID properties.
11. What is database indexing?
12. What is replication?
13. What is sharding?
14. What is replication lag?
15. How do you choose a shard key?
16. What is the difference between strong and eventual consistency?
17. Explain the CAP theorem.
18. How would you handle duplicate database records?

### Caching

19. What is cache-aside?
20. Explain write-through and write-behind caching.
21. What is cache invalidation?
22. What are TTL, LRU, and LFU?
23. What is a cache stampede?
24. How do you handle Redis failure?
25. When would you use Caffeine instead of Redis?

### Distributed systems

26. What is a message broker?
27. Kafka vs. RabbitMQ: how do you choose?
28. What is idempotency?
29. What is the transactional outbox pattern?
30. What is a circuit breaker?
31. Why are timeouts important?
32. Why can retries make an outage worse?
33. How do you prevent duplicate payment processing?
34. How do you maintain consistency across microservices?

### Production engineering

35. How do you monitor a production service?
36. What is distributed tracing?
37. How do you diagnose high p99 latency?
38. What happens when a database becomes unavailable?
39. How would you deploy a service without downtime?
40. How would you design for sudden traffic spikes?

## 31. Common Mistakes to Avoid

- Starting with microservices before understanding the requirements.
- Selecting a database based only on popularity.
- Adding Redis without defining invalidation and failure behavior.
- Assuming a load balancer removes every single point of failure.
- Assuming retries always improve reliability.
- Treating Kafka delivery as equivalent to exactly-once business processing.
- Assuming database replicas always contain the latest data.
- Sharding before measuring the actual database bottleneck.
- Ignoring security and authorization.
- Reporting only average latency.
- Forgetting queue capacity, backpressure, and overload behavior.
- Drawing a complex architecture without explaining its trade-offs.

## 32. Quick Revision Cheat Sheet

| Concept | Remember |
|---|---|
| Scalability | Handle increasing demand |
| Availability | Provide usable responses when requested |
| Reliability | Continue delivering correct service over time |
| Latency | Time taken per operation |
| Throughput | Operations completed per unit of time |
| Load balancer | Distributes requests |
| API gateway | Applies API-level routing and policies |
| CDN | Serves suitable content near users |
| Cache | Reduces repeated expensive data access |
| Index | Accelerates supported lookup patterns |
| Replication | Maintains copies of data |
| Sharding | Distributes data across partitions or nodes |
| CAP | During a partition, consistency and availability cannot both be guaranteed for every request |
| Strong consistency | Reads obey the specified latest-write guarantees |
| Eventual consistency | Replicas converge if updates stop and propagation succeeds |
| Circuit breaker | Temporarily blocks calls to a failing dependency |
| Rate limiter | Controls request frequency |
| Message broker | Enables decoupled communication |
| Idempotency | Repeating an operation does not repeat its intended effect |
| Observability | Understands system behavior through metrics, logs, and traces |
| Backpressure | Prevents producers from overwhelming downstream capacity |

## Conclusion

System Design is about making informed decisions under real-world constraints.

For Java and Spring Boot developers, the most useful starting point is to understand HTTP APIs, SQL transactions, indexing, caching, concurrency, distributed messaging, resilience patterns, and observability. These fundamentals form the foundation for designing larger systems.

During interviews, begin with requirements and estimates, draw a simple architecture, explain the data flow, identify bottlenecks, and introduce additional components only when needed. Support every significant decision with a clear explanation of its benefits, limitations, and alternatives.

**The goal is not to use the most technologies. The goal is to build the simplest architecture that satisfies the requirements and can evolve safely as demand grows.**
