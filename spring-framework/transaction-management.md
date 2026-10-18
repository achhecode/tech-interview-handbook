---
title: Transaction Management
parent: Spring Framework
nav_order: 6
---

# Transaction Management
{: .no_toc }

How @Transactional works, propagation and common pitfalls.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

# Spring Transaction Management

## 1. What is transaction management in Spring?

**Spring transaction management** provides a consistent way to manage database transactions across technologies such as JDBC, JPA, Hibernate, and JTA.

A transaction groups multiple database operations into one logical unit of work. Either the operations are committed successfully, or the transaction is rolled back when a qualifying failure occurs.

For example, when transferring money between two bank accounts:

1. Deduct money from Account A.
2. Add money to Account B.
3. Commit both operations together.

If the second operation fails, the first operation should also be rolled back so that money is not lost.

### ACID properties

| Property | Meaning |
|---|---|
| Atomicity | All operations succeed together or are rolled back together. |
| Consistency | A transaction preserves defined data integrity rules. |
| Isolation | Concurrent transactions are isolated according to the selected isolation level. |
| Durability | Committed changes persist according to the database's durability guarantees. |

## 2. Main components of Spring transaction management

### `PlatformTransactionManager`

This is the central interface used by Spring's imperative transaction infrastructure to manage transactions.

Common implementations include:

- `DataSourceTransactionManager` — JDBC transactions.
- `JdbcTransactionManager` — JDBC transactions with additional JDBC exception translation support in applicable Spring versions.
- `JpaTransactionManager` — JPA transactions.
- `JtaTransactionManager` — transactions managed through JTA, including suitable distributed transaction setups.

`HibernateTransactionManager` is also available in applicable Spring versions for native Hibernate transaction management.

Spring Boot typically configures the appropriate transaction manager automatically when the relevant persistence technology and dependencies are configured.

### `@Transactional`

This annotation enables declarative transaction management. We mark a method or class as transactional, and Spring manages the transaction around intercepted method execution.

```java
@Service
public class EmployeeService {

    private final EmployeeRepository employeeRepository;

    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    @Transactional
    public void updateEmployee(Long id, String name) {
        Employee employee = employeeRepository.findById(id)
                .orElseThrow(() ->
                        new IllegalArgumentException("Employee not found"));

        employee.setName(name);
        employeeRepository.save(employee);
    }
}
```

When this method is invoked through the Spring proxy:

1. Spring starts a transaction or joins an existing one, depending on the propagation setting.
2. The method executes.
3. If it completes successfully, the transaction interceptor attempts to commit the transaction.
4. If a qualifying exception occurs, Spring applies the configured rollback rules.

**Important:** By default, Spring rolls back on `RuntimeException` and `Error`, but not on checked exceptions. You can customize this using `rollbackFor` and related attributes.

## 3. How to use transaction management in Spring Boot

For most Spring Boot applications, declarative transaction management using `@Transactional` is the recommended starting point.

### Step 1: Add the required dependency

For a Spring Boot application using Spring Data JPA, include the JPA starter and the database driver appropriate for your database.

For example, the Maven dependency for JPA is:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

Spring Boot typically configures the data source, JPA infrastructure, and transaction manager when the required dependencies and database configuration are present.

### Step 2: Create a service method

```java
@Service
public class EmployeeService {

    private final EmployeeRepository employeeRepository;

    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    @Transactional
    public void updateEmployee(Long id, String newName) {
        Employee employee = employeeRepository.findById(id)
                .orElseThrow(() ->
                        new IllegalArgumentException("Employee not found"));

        employee.setName(newName);
        employeeRepository.save(employee);
    }
}
```

Here, the read and write operations execute within the same transaction, provided the call enters through the Spring transaction proxy and the repository uses the same transaction manager and resource.

With JPA, saving a managed entity is often unnecessary because changes are detected and flushed automatically. The explicit `save()` can still be useful for clarity or for other repository workflows.

## 4. Important attributes of `@Transactional`

Spring allows us to configure transaction behavior using annotation attributes.

```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    timeout = 30,
    rollbackFor = Exception.class
)
public void updateEmployee() {
    // Database operations
}
```

| Attribute | Purpose |
|---|---|
| `propagation` | Determines how the method behaves when a transaction already exists. |
| `isolation` | Controls visibility of concurrent transaction changes. |
| `timeout` | Specifies the transaction timeout in seconds, subject to transaction manager support. |
| `readOnly` | Marks a transaction as read-only where supported; it is generally a hint, not a universal write prohibition. |
| `rollbackFor` | Specifies additional exception types that should trigger rollback. |
| `noRollbackFor` | Specifies exceptions that should not trigger rollback. |

### Transaction propagation

| Propagation | Behavior |
|---|---|
| `REQUIRED` | Joins an existing transaction or starts a new one. This is the default. |
| `REQUIRES_NEW` | Suspends an existing transaction and starts a separate one, if supported. |
| `SUPPORTS` | Joins an existing transaction if available; otherwise runs without one. |
| `MANDATORY` | Requires an existing transaction. |
| `NOT_SUPPORTED` | Suspends an existing transaction and runs without one, if supported. |
| `NEVER` | Fails if an existing transaction is present. |
| `NESTED` | Uses a nested transaction or savepoint when supported by the transaction manager. |

**Example:** If a service method uses `REQUIRED` and calls another transactional service method, the inner method normally joins the existing transaction. If the participating operation marks the shared transaction rollback-only, the outer transaction may ultimately roll back too.

### Transaction isolation

| Isolation level | General behavior |
|---|---|
| `READ_UNCOMMITTED` | May allow dirty reads; actual support varies by database. |
| `READ_COMMITTED` | Prevents dirty reads. |
| `REPEATABLE_READ` | Prevents dirty reads and non-repeatable reads under the database's implementation. |
| `SERIALIZABLE` | Provides the strongest standard isolation level, generally with greater contention or reduced concurrency. |

The exact behavior and supported isolation levels depend on the database and transaction manager.

## 5. How does rollback work?

Consider a bank transfer:

```java
@Transactional
public void transfer(Long fromId, Long toId, BigDecimal amount) {

    Account from = accountRepository.findById(fromId)
            .orElseThrow();

    Account to = accountRepository.findById(toId)
            .orElseThrow();

    from.setBalance(from.getBalance().subtract(amount));
    accountRepository.save(from);

    to.setBalance(to.getBalance().add(amount));
    accountRepository.save(to);
}
```

If a qualifying unchecked exception escapes this method before commit, Spring requests a rollback of the transaction, so both account changes are rolled back together when they participate in the same transaction.

For checked exceptions, specify the rollback rule when appropriate:

```java
@Transactional(rollbackFor = Exception.class)
public void transferMoney() throws Exception {
    // Database operations
}
```

Remember that catching and swallowing an exception can prevent the usual rollback trigger unless the transaction is explicitly marked rollback-only or another rollback condition applies.

## 6. What is programmatic transaction management?

Instead of relying on `@Transactional`, you can control transaction boundaries in code using `TransactionTemplate` or the lower-level `PlatformTransactionManager` API.

### Example using `TransactionTemplate`

```java
@Service
public class EmployeeService {

    private final TransactionTemplate transactionTemplate;
    private final EmployeeRepository employeeRepository;

    public EmployeeService(
            TransactionTemplate transactionTemplate,
            EmployeeRepository employeeRepository) {
        this.transactionTemplate = transactionTemplate;
        this.employeeRepository = employeeRepository;
    }

    public void updateEmployee(Long id, String newName) {
        transactionTemplate.executeWithoutResult(status -> {
            Employee employee = employeeRepository.findById(id)
                    .orElseThrow();

            employee.setName(newName);
            employeeRepository.save(employee);
        });
    }
}
```

Spring manages transaction completion around the callback. An unchecked exception escaping the callback normally triggers rollback; you can also explicitly mark the transaction rollback-only through `status.setRollbackOnly()`.

### Declarative vs programmatic transaction management

| Declarative | Programmatic |
|---|---|
| Uses `@Transactional`. | Uses `TransactionTemplate` or transaction manager APIs. |
| Less boilerplate. | More explicit control over transaction boundaries. |
| Best for most ordinary service methods. | Useful when transaction handling must be controlled dynamically in code. |

## 7. Common interview pitfalls

### Does `@Transactional` work on every method call?

No. In the default proxy-based configuration, a method call must pass through the Spring proxy. **Self-invocation**—one method calling another transactional method on the same object—normally bypasses the proxy, so the second method's transaction settings may not be applied.

### Does `@Transactional` make multiple microservice calls atomic?

No. A local database transaction generally controls resources managed by its transaction manager. It does not automatically roll back an HTTP call to another service or a message already delivered to Kafka.

Distributed workflows may require a saga, an outbox pattern, compensating actions, or a properly configured distributed transaction solution.

### Is `@Transactional` applied to private methods?

Private methods cannot be intercepted through standard Spring AOP proxies in the usual way. Put the transactional boundary on a suitable externally invoked method, typically a public service method.

### Does Spring Boot require manual transaction-manager configuration?

Usually not for a standard application with correctly configured Spring Data JPA or JDBC infrastructure. Spring Boot commonly auto-configures a suitable transaction manager. Manual configuration may be necessary for multiple data sources or specialized transaction requirements.

## 8. Interview-ready answer

**Question: What is Spring transaction management, and how do you use it in Spring Boot?**

Spring transaction management provides a consistent way to control database transactions across JDBC, JPA, Hibernate, and other supported technologies. Its central abstraction is `PlatformTransactionManager`, and the most common approach is declarative transaction management using `@Transactional`.

I typically put `@Transactional` on service-layer methods so multiple repository operations execute within a single transaction. Spring starts or joins a transaction according to the propagation rules and commits or rolls it back according to the outcome and configured rollback rules.

I can configure propagation, isolation, timeout, read-only behavior, and rollback rules through annotation attributes. For more explicit control, I use `TransactionTemplate` for programmatic transaction management.

In production, I pay attention to rollback behavior, proxy limitations such as self-invocation, transaction boundaries, and the fact that local database transactions do not automatically make distributed microservice operations atomic.

## Quick revision

- **Main interface:** `PlatformTransactionManager`
- **Declarative approach:** `@Transactional`
- **Programmatic approach:** `TransactionTemplate`
- **Default propagation:** `REQUIRED`
- **Default rollback:** `RuntimeException` and `Error`
- **Checked exceptions:** Configure `rollbackFor` when needed.
- **Typical placement:** Service layer
- **Common pitfall:** Self-invocation bypasses the default Spring transaction proxy.
- **Microservices:** Local transactions do not automatically provide distributed atomicity.


---

# Spring Boot Transactions: Isolation Levels and Microservice Transactions

## Part 1: How do you handle transactions in Spring Boot?

### 1. What is a transaction?

A transaction is a group of database operations treated as a single logical unit of work. It should satisfy the ACID properties:

- **Atomicity:** All operations succeed together or are rolled back together.
- **Consistency:** Data integrity rules remain satisfied.
- **Isolation:** Concurrent transactions interact according to the configured isolation level.
- **Durability:** Committed changes persist according to database guarantees.

Spring provides transaction management abstractions that work with JDBC, JPA, Hibernate, and other supported technologies. Spring transactions do not necessarily require JTA.

### 2. Declarative transaction management using `@Transactional`

This is the most common approach in Spring Boot.

```java
@Service
public class AccountService {

    private final AccountRepository accountRepository;

    public AccountService(AccountRepository accountRepository) {
        this.accountRepository = accountRepository;
    }

    @Transactional
    public void transferMoney(
            Long fromId, Long toId, BigDecimal amount) {

        Account from = accountRepository.findById(fromId)
                .orElseThrow();

        Account to = accountRepository.findById(toId)
                .orElseThrow();

        if (from.getBalance().compareTo(amount) < 0) {
            throw new IllegalStateException("Insufficient balance");
        }

        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));
    }
}
```

Assuming both accounts use the same database transaction, both changes are committed together. If an unchecked exception escapes the method, Spring normally rolls back the transaction.

For a production banking application, also consider concurrent balance updates, optimistic or pessimistic locking, database constraints, and duplicate request prevention.

### 3. Isolation levels in Spring

An isolation level determines how one transaction can observe changes made by other concurrent transactions.

| Isolation level | Dirty reads | Non-repeatable reads | Phantom reads |
|---|---|---|---|
| `READ_UNCOMMITTED` | Possible | Possible | Possible |
| `READ_COMMITTED` | Prevented | Possible | Possible |
| `REPEATABLE_READ` | Prevented | Prevented | Database-dependent phantom handling |
| `SERIALIZABLE` | Prevented | Prevented | Prevented according to serializable semantics |

The actual behavior depends on the database engine and its concurrency implementation. These are the standard isolation-level guarantees, not a promise that every database implements them identically.

#### A. Read uncommitted

A transaction may read data that another transaction has modified but not committed.

**Problem:** Dirty reads.

Example: Transaction A changes an account balance to ₹5,000 but has not committed. Transaction B reads ₹5,000. Transaction A then rolls back, so Transaction B read a value that was never committed.

#### B. Read committed

A transaction reads only committed data.

**Problem:** The same query may return different values if another transaction commits an update between reads.

This is a common default isolation level in databases such as PostgreSQL and Oracle.

#### C. Repeatable read

If a transaction reads a row twice, another transaction's committed update should not change the value observed by that transaction under repeatable-read semantics.

Phantom handling depends on the database implementation. Some engines provide stronger behavior through snapshot isolation or additional locking.

#### D. Serializable

This provides the strongest standard SQL isolation level. Concurrent transactions must behave as if they executed in some serial order.

It can reduce concurrency and may cause transactions to block or abort and require retries.

### 4. How to configure isolation in Spring Boot

```java
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Transactional;

@Transactional(isolation = Isolation.READ_COMMITTED)
public void updateAccount() {
    // Database operations
}
```

You can also specify a timeout and rollback rules:

```java
@Transactional(
    isolation = Isolation.READ_COMMITTED,
    timeout = 30,
    rollbackFor = Exception.class
)
public void processOrder() throws Exception {
    // Database operations
}
```

Important points:

- The isolation setting applies when Spring starts a new transaction. If the method joins an existing transaction, the existing transaction's isolation typically remains in effect.
- A transaction manager or database may not support every requested isolation level.
- The timeout is subject to transaction-manager support and is not a universal guarantee that every database operation will be interrupted at precisely 30 seconds.
- Spring normally rolls back on unchecked exceptions and `Error`; `rollbackFor` can extend the rollback rules to checked exceptions.

---

## Part 2: How do transactions work in Spring Boot microservices?

### 5. Why are microservice transactions different?

Consider an e-commerce application with three independent services:

- **Order Service:** Creates an order.
- **Payment Service:** Charges the customer.
- **Inventory Service:** Reserves stock.

A customer places an order. The order is created, but payment fails.

What should happen to the order and the inventory reservation?

Each service typically owns its own database. A local `@Transactional` method can manage changes within its own transactional resource, but it does not automatically roll back changes made by another service over HTTP or Kafka.

**A local Spring transaction is not a distributed business transaction.**

Microservices can be stateless at the application-instance level while maintaining durable state in their databases. That durable business state is what allows a workflow to resume, retry, or compensate after a failure.

### 6. Approach 1: Local transactions with `@Transactional`

Each service manages its own database operations.

For example, in the Order Service:

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        Order order = new Order();
        order.setCustomerId(request.customerId());
        order.setStatus("PENDING");

        return orderRepository.save(order);
    }
}
```

This transaction manages the local order database operation. It does not include payment or inventory changes performed by remote services.

### 7. Approach 2: Saga pattern

The **Saga pattern** coordinates a business workflow across multiple services using a sequence of local transactions. If a later step fails, the workflow runs compensating actions to counteract previously completed steps.

Example workflow:

<box border radius="lg" padding={3} gap={2}>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **1. Order Service**

    <text size="sm">Create order: PENDING</text>
  </box>
  <icon name="arrow-down" size="lg" color="secondary" />
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **2. Inventory Service**

    <text size="sm">Reserve stock</text>
  </box>
  <icon name="arrow-down" size="lg" color="secondary" />
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **3. Payment Service**

    <text size="sm">Authorize or capture payment</text>
  </box>
  <icon name="arrow-down" size="lg" color="secondary" />
  <box border radius="md" padding={3} align="center">
    **4. Order Service**

    <text size="sm">Mark order CONFIRMED</text>
  </box>
  <divider />
  <text weight="medium">If payment fails</text>
  <box background="surface" border radius="md" padding={3} align="center">
    Release reserved stock and mark the order FAILED or CANCELLED, according to business rules.
  </box>
</box>

Each service commits its own local transaction. A failure triggers compensating business operations instead of attempting to roll back every database across all services.

For example:

- Reserve stock → compensation: release stock.
- Authorize payment → compensation: void the authorization.
- Capture payment → compensation: refund the payment, if supported and appropriate.

A compensation is a new business operation, not a literal rollback of another service's database transaction. It can fail and must itself be retried and monitored.

#### Two common Saga styles

| Choreography | Orchestration |
|---|---|
| Services react to events and publish subsequent events. | A coordinator directs the workflow. |
| Avoids a central workflow controller. | Makes workflow steps and failure handling explicit. |
| Can become difficult to trace as workflows grow. | Can be easier to understand for complex business processes. |

For a complex order-payment-inventory workflow, orchestration can make the sequence and compensation logic clearer. Kafka-based event choreography can also work well when services are loosely coupled.

### 8. Approach 3: Transactional outbox pattern

A common problem arises when a service updates its database and publishes an event to Kafka.

Suppose the Order Service:

1. Saves the order in its database.
2. Publishes `OrderCreated` to Kafka.

If the database commit succeeds but Kafka publication fails, other services may never learn that the order exists.

The transactional outbox pattern addresses this by saving both the business change and an event record in the **same local database transaction**.

```java
@Transactional
public void createOrder(CreateOrderRequest request) {

    Order order = orderRepository.save(
            Order.pending(request.customerId())
    );

    OutboxEvent event = OutboxEvent.orderCreated(
            order.getId()
    );

    outboxRepository.save(event);
}
```

A separate publisher reads unpublished outbox records and sends them to Kafka. After successful publication, the record is marked as published.

The publisher may send an event more than once, so consumers should be idempotent—for example, by recording processed event IDs or enforcing business-level uniqueness.

The outbox pattern makes the database update and event-record creation atomic. It does not, by itself, guarantee exactly-once delivery to every downstream consumer.

### 9. Approach 4: Distributed transactions using JTA/XA

In certain systems, distributed transactions can be implemented using JTA/XA-capable transaction managers and resources.

A two-phase commit protocol coordinates participating resources through preparation and commit phases.

However, distributed transactions introduce coordination overhead, resource requirements, and operational complexity. Many microservice architectures avoid XA transactions across independently owned services and instead use Saga and outbox patterns.

### 10. Why not use `ThreadLocal` to maintain transaction state?

`ThreadLocal` stores data associated with a particular thread. It does not provide a durable transaction record or transfer transaction context automatically across network boundaries.

Spring's transaction infrastructure already manages transaction resources and synchronization for local transactions. `TransactionSynchronizationManager` is an infrastructure API, not a general-purpose mechanism for carrying a business transaction between microservices.

Avoid using `ThreadLocal` for distributed transaction coordination because:

- Requests can be processed on different threads.
- Asynchronous execution may switch threads.
- HTTP and Kafka boundaries do not share the caller's thread-local state.
- Thread-local data is not durable across process restarts.
- It cannot atomically commit multiple independent databases.

Use durable workflow state, correlation IDs, event IDs, idempotent handlers, and Saga coordination when a business operation spans services.

`TransactionSynchronization` is useful for callbacks associated with a local transaction—for example, reacting after commit—but it does not turn local transactions into distributed transactions.

---

## Part 3: Production interview scenarios

### Scenario 1: Payment succeeds, but the Order Service does not receive the response

The network may fail after payment succeeds, leaving the caller unsure whether the payment happened.

A robust design should:

1. Use a stable payment idempotency key for retries of the same logical payment.
2. Persist payment status and a unique business reference.
3. Query or reconcile payment status when the result is uncertain.
4. Avoid issuing a second charge just because a response timed out.
5. Use an appropriate refund or compensation workflow if the order ultimately cannot be completed.

### Scenario 2: Kafka delivers the same event twice

Kafka consumers and downstream systems may see duplicate processing attempts, depending on the delivery and failure scenario.

A consumer should be idempotent. For example, store a unique event ID in the same local transaction as the business update, and skip processing if that event has already been applied.

### Scenario 3: Inventory reservation succeeds, but payment fails

The Saga should invoke a compensation to release the reservation. If the release operation temporarily fails, retry it and retain enough durable workflow state to resume the process.

### Scenario 4: Two customers attempt to buy the final item

Transaction isolation alone may not be sufficient for every inventory design. Use appropriate concurrency control, such as an atomic conditional update, optimistic locking, pessimistic locking, or a database constraint.

For example, an atomic stock decrement can prevent inventory from becoming negative:

```sql
UPDATE inventory
SET available_quantity = available_quantity - 1
WHERE product_id = :productId
  AND available_quantity > 0;
```

Check the affected row count. If it is zero, the item was unavailable or could not be reserved. Execute this within the appropriate local transaction.

---

## 11. Interview-ready answer

**Question: How do you handle transactions and isolation levels in Spring Boot microservices?**

For a single service, I use Spring's declarative transaction management with `@Transactional`. I choose an isolation level such as `READ_COMMITTED` based on the application's consistency requirements and database behavior. I also consider propagation, rollback rules, transaction timeouts, and concurrency control.

In microservices, each service generally manages its own local database transaction. A normal `@Transactional` annotation cannot make independent services atomic across HTTP or Kafka calls.

For a business workflow spanning multiple services, I would typically use the Saga pattern with local transactions and compensating actions. To reliably publish events, I would use the transactional outbox pattern and make consumers idempotent to handle duplicate delivery. JTA/XA distributed transactions are an option in systems that require them and can support the associated infrastructure, but they are not the default solution for most microservice workflows.

I would not use `ThreadLocal` to share transaction state between services. Instead, I would persist workflow status and use correlation IDs, idempotency keys, durable events, and retries to maintain consistency.

## Quick revision

- **Local transaction:** `@Transactional`
- **Transaction manager:** `PlatformTransactionManager`
- **Isolation:** Controls how concurrent transactions observe data.
- **Dirty read:** Reading uncommitted data.
- **Non-repeatable read:** The same row produces a different value on a later read.
- **Phantom read:** A repeated query returns a different set of matching rows.
- **Saga:** Coordinates distributed business steps using local transactions and compensations.
- **Transactional outbox:** Atomically records a database change and the event to publish.
- **Idempotent consumer:** Safely handles duplicate processing attempts.
- **JTA/XA:** An option for coordinated distributed transactions when supported and appropriate.
- **ThreadLocal:** Not a distributed transaction mechanism.


---


