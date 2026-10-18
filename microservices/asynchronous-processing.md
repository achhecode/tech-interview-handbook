---
title: Asynchronous Processing
parent: Microservices
nav_order: 5
---

# Asynchronous Processing
{: .no_toc }

Async execution inside a service.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---


## How do you call methods asynchronously in Spring Boot? How do you call another microservice asynchronously?

### 1. What is Asynchronous Execution?

Asynchronous execution allows a program to start a task without blocking the current thread until that task finishes. The caller can continue performing other work while the task executes.

In Spring Boot, there are two common scenarios:

1. **Asynchronous method execution:** Use `@Async` with a configured task executor.
2. **Asynchronous microservice communication:** Use a message broker such as Kafka or RabbitMQ when the caller does not need an immediate response.

**Important interview distinction:** `@Async` runs a method asynchronously within an application. It does not, by itself, make an HTTP call to another microservice asynchronous or provide durable messaging.

---

### 2. How do you execute a method asynchronously using `@Async`?

Spring provides the `@Async` annotation to execute a method using a Spring-managed task executor.

**Step 1: Enable asynchronous execution**

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean("taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor =
                new ThreadPoolTaskExecutor();

        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();

        return executor;
    }
}
```

This creates a thread pool with a core size of 5, a maximum size of 10, and a queue capacity of 100.

**Step 2: Create an asynchronous service**

```java
@Service
public class NotificationService {

    @Async("taskExecutor")
    public CompletableFuture<String> sendNotification() {
        // Simulate a time-consuming operation
        try {
            Thread.sleep(2000);
        } catch (InterruptedException ex) {
            Thread.currentThread().interrupt();
            throw new IllegalStateException(
                    "Notification interrupted", ex);
        }

        return CompletableFuture.completedFuture(
                "Notification sent successfully");
    }
}
```

**Step 3: Call the asynchronous method**

```java
@Service
public class OrderService {

    private final NotificationService notificationService;

    public OrderService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void placeOrder() {
        System.out.println("Order processing started");

        notificationService.sendNotification();

        System.out.println("Order processing continues");
    }
}
```

The caller does not wait for the notification method to finish before printing the second message. The notification task runs on a thread managed by the configured executor.

### 3. How do you get the result of an asynchronous method?

Use `CompletableFuture` when the caller needs the result later.

```java
CompletableFuture<String> future =
        notificationService.sendNotification();

future.thenAccept(result ->
        System.out.println("Result: " + result));
```

`thenAccept()` registers a callback that runs when the result becomes available. It does not force the calling thread to wait.

If you call `future.get()` or `future.join()`, the caller may block until the result is available, so use those methods only when waiting is appropriate.

### 4. Important points about `@Async`

- Add `@EnableAsync` to enable Spring's asynchronous method execution.
- Call the method through a Spring-managed bean, typically by injecting the service into another service.
- In the default proxy-based configuration, a method calling another `@Async` method on the same object bypasses the proxy, so the call will not run asynchronously.
- Configure the thread pool appropriately to avoid exhausting application resources.
- For `void` methods, handle asynchronous exceptions through an `AsyncUncaughtExceptionHandler` or suitable logging and error-handling mechanisms.
- `@Async` is not a durable job queue: tasks can be lost if the application stops unexpectedly.

---

### 5. How do you call another microservice asynchronously?

There are two main approaches, depending on whether you need asynchronous HTTP execution or fully decoupled messaging.

#### Approach A: Asynchronous HTTP call

You can use Spring's `WebClient` to make a non-blocking HTTP request to another microservice.

```java
@Service
public class InventoryClient {

    private final WebClient webClient;

    public InventoryClient(WebClient.Builder builder) {
        this.webClient = builder
                .baseUrl("http://inventory-service")
                .build();
    }

    public Mono<String> checkInventory(Long productId) {
        return webClient.get()
                .uri("/api/inventory/{id}", productId)
                .retrieve()
                .bodyToMono(String.class);
    }
}
```

Usage:

```java
inventoryClient.checkInventory(101L)
        .subscribe(
            result -> System.out.println(result),
            error -> System.err.println(
                    "Inventory call failed: " + error.getMessage())
        );

System.out.println("Other work can continue");
```

`WebClient` supports non-blocking I/O. The returned `Mono` represents a result that may arrive later. Calling `subscribe()` starts the reactive flow, but production applications should manage error handling, lifecycle, and subscription appropriately. In WebFlux applications, composing and returning the `Mono` is generally preferable to manually subscribing.

**Use this approach when:** You need to make an HTTP request without blocking the calling thread, but still want a response from the other service.

#### Approach B: Asynchronous messaging with Kafka or RabbitMQ

When services should operate independently, use a message broker.

Example: The Order Service publishes an `OrderCreated` event, and the Notification Service consumes it.

<box border radius="lg" padding={3} align="center" gap={2}>
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **Order Service**
    <text color="secondary" size="sm">Publishes OrderCreated</text>
  </box>
  <icon name="arrow-down" size="lg" />
  <box border radius="md" padding={3} width="100%" align="center">
    **Kafka / RabbitMQ**
    <text color="secondary" size="sm">Buffers and delivers messages</text>
  </box>
  <icon name="arrow-down" size="lg" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **Notification Service**
    <text color="secondary" size="sm">Consumes event and sends notification</text>
  </box>
</box>

**Producer example using Spring Kafka**

```java
@Service
public class OrderEventPublisher {

    private final KafkaTemplate<String, String> kafkaTemplate;

    public OrderEventPublisher(
            KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void publishOrderCreated(String orderId) {
        kafkaTemplate.send(
                "order-created",
                orderId,
                "Order created: " + orderId
        );
    }
}
```

**Consumer example**

```java
@Service
public class NotificationConsumer {

    @KafkaListener(
        topics = "order-created",
        groupId = "notification-service"
    )
    public void consumeOrderCreated(String message) {
        System.out.println("Received event: " + message);

        // Send notification to the customer
    }
}
```

These examples assume Kafka and the appropriate Spring Kafka dependencies are configured, including the broker address and topic.

**Benefits of asynchronous messaging**

- **Loose coupling:** The producer does not need to call the consumer directly.
- **Resilience:** Durable broker configuration can retain messages while a consumer is temporarily unavailable.
- **Scalability:** Consumers can process messages independently and scale according to demand.
- **Buffering:** A backlog can absorb temporary traffic spikes.

For production, consider message acknowledgments, retry policies, dead-letter topics or queues, idempotent consumers, and the transactional outbox pattern where database updates and event publishing must remain consistent.

---

### 6. `@Async` vs `WebClient` vs Kafka/RabbitMQ

| Feature | `@Async` | `WebClient` | Kafka / RabbitMQ |
|---|---|---|---|
| Main purpose | Execute methods on another thread | Make asynchronous HTTP calls | Communicate through messages |
| Requires a broker | No | No | Yes |
| Direct response | Possible through `Future` | Yes, asynchronously | Not usually immediate |
| Loose coupling | Low to moderate | Moderate | High |
| Durable buffering | No, by itself | No, by itself | Possible with appropriate configuration |
| Best use case | Background tasks inside an application | Non-blocking service-to-service HTTP | Event-driven workflows and background processing |

### 7. Which approach should you choose?

- **Send an email in the background:** `@Async` may be sufficient for a simple, non-durable task.
- **Call Inventory Service and use its response:** Use REST with `WebClient` or another suitable HTTP client.
- **Publish an order event for several independent services:** Use Kafka or RabbitMQ.
- **Run a business-critical background task that must survive application restarts:** Use a durable job or messaging system rather than relying solely on `@Async`.

### Quick Interview Revision

**Question:** How do you call methods asynchronously in Spring Boot, and how do you call another microservice asynchronously?

**Answer:** For asynchronous method execution within a Spring Boot application, I use `@Async`, `@EnableAsync`, and a properly configured `TaskExecutor`. I can return a `CompletableFuture` when the result is required later. For non-blocking HTTP communication between microservices, I use `WebClient`. When I need loose coupling, durable message delivery, or event-driven processing, I use Kafka or RabbitMQ. The choice depends on whether I need a direct response, non-blocking I/O, or reliable asynchronous messaging.


