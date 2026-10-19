---
title: Messaging with Kafka
parent: Microservices
nav_order: 6
---

# Messaging with Kafka
{: .no_toc }

Apache Kafka, Spring Boot integration, event-driven architecture, and message delivery guarantees.
{: .fs-6 .fw-300 }

<details open markdown="block">
<summary>Questions on this page</summary>
{: .text-delta }

1. TOC
{:toc}
</details>

---

# Messaging with Kafka: A Complete Java Guide

Apache Kafka is a distributed event streaming platform used to publish, store, and process streams of events. It is commonly used for asynchronous communication between microservices, event-driven architecture, real-time analytics, and data pipelines.

Spring Boot integrates with Kafka through Spring for Apache Kafka, commonly called Spring Kafka. It provides `KafkaTemplate` for publishing messages, `@KafkaListener` for consuming messages, configuration support, and mechanisms for handling failures.

This guide uses Java 21, Spring Boot 3.5.x, Maven, and Apache Kafka running in Docker Compose. Spring Boot manages the compatible Spring Kafka dependency version, so the project does not need to specify a separate Spring Kafka version.

## 1. What is Apache Kafka?

Kafka is a distributed event streaming platform that allows applications to:

- Publish events to topics.
- Subscribe to topics and process events.
- Store events for a configurable retention period.
- Distribute processing across multiple consumers.
- Replicate data across brokers for fault tolerance.
- Replay retained events when required.

For example, consider an e-commerce application.

When an order is created, the Order Service publishes an `OrderCreated` event. Other services can independently consume that event to send notifications, update analytics, or initiate inventory workflows.

```text
                    Order Service
                          |
                          | Publish OrderCreated
                          v
                    +-------------+
                    |    Kafka    |
                    | order-events|
                    +------+------+
                           |
              +------------+------------+
              |            |            |
              v            v            v
        Notification   Analytics    Inventory
          Service       Service      Service
```

The Order Service does not need to call every downstream service synchronously. Kafka stores the event, and each consumer processes it independently.

**Important:** Kafka is not simply an in-memory message queue. It stores records in a durable, partitioned log. Records can remain available after consumption until they expire under the topic's retention policy or are removed according to the configured cleanup policy.

## 2. Core Kafka Concepts

### Topic

A topic is a named stream of records.

Examples:

- `order-events`
- `payment-events`
- `user-events`
- `notification-events`

A topic is divided into partitions to support parallel processing and distributed storage.

### Partition

A partition is an ordered log of records.

```text
Topic: order-events

Partition 0:
Offset 0 -> Order A
Offset 1 -> Order B
Offset 2 -> Order C

Partition 1:
Offset 0 -> Order D
Offset 1 -> Order E
Offset 2 -> Order F
```

Ordering is guaranteed within a partition, not across all partitions of a topic.

Records with the same key are normally routed to the same partition by the default keyed partitioning strategy, which helps preserve ordering for that key.

### Offset

An offset is a record's position within a partition.

For example:

```text
Topic: order-events
Partition: 0
Offset: 105
```

This identifies a record at offset `105` in partition `0`.

Offsets are partition-specific. Two different partitions can both contain a record at offset `105`.

Consumer offsets track progress through the partition. Committing an offset allows a consumer group to resume after a restart, subject to the consumer's configured offset-reset behavior.

### Producer

A producer publishes records to Kafka topics.

In Spring Boot, `KafkaTemplate` is the standard abstraction for sending records.

### Consumer

A consumer reads records from Kafka and processes them.

Spring Kafka provides the `@KafkaListener` annotation to implement message listeners.

### Broker

A broker is a Kafka server that stores partition data and serves producer and consumer requests.

A Kafka cluster consists of one or more brokers.

### Replication

Replication maintains copies of partitions across brokers.

A replicated partition has a leader that handles normal reads and writes, with follower replicas maintaining copies.

Replication improves fault tolerance, but the actual durability guarantee depends on acknowledgement settings, in-sync replicas, and the failure scenario.

### Consumer group

A consumer group is a set of consumers that cooperate to process a topic.

Within one consumer group, a partition is assigned to at most one consumer at a time.

For example, if a topic has three partitions and a group has three consumers, each consumer can process one partition concurrently.

If the group has five consumers, at least two will be idle for that topic while only three partitions exist.

Different consumer groups independently consume the same topic.

## 3. What is Spring Boot?

Spring Boot simplifies the development of Java applications using the Spring Framework.

It provides:

- Auto-configuration.
- Dependency management.
- Embedded web servers.
- Externalized configuration.
- Dependency injection.
- Production integration through Spring Boot Actuator.

For example:

```java
@SpringBootApplication
public class KafkaDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(
            KafkaDemoApplication.class,
            args
        );
    }
}
```

The `@SpringBootApplication` annotation combines configuration, component scanning, and auto-configuration.

## 4. Why Integrate Kafka with Spring Boot?

Spring Kafka provides convenient abstractions for working with Kafka from Java applications.

| Feature | Spring Kafka component |
|---|---|
| Publish messages | `KafkaTemplate` |
| Consume messages | `@KafkaListener` |
| Configure serialization | Producer and consumer configuration |
| Retry failed processing | Error handlers and retry mechanisms |
| Send failed records to a DLT | `DeadLetterPublishingRecoverer` |
| Create topics | `NewTopic` and `TopicBuilder` |
| Coordinate Kafka transactions | `KafkaTransactionManager` and transactional producers |

Kafka and Spring Boot are commonly used for:

- Asynchronous microservice communication.
- Order and payment event processing.
- Audit event streams.
- Real-time analytics.
- Log aggregation.
- Event-driven workflows.

## 5. Project Requirements

The following example implements a small order event application.

The application will:

1. Expose a REST endpoint to create an order event.
2. Publish a JSON event to Kafka.
3. Use the order ID as the Kafka message key.
4. Consume the event using `@KafkaListener`.
5. Retry transient consumer failures.
6. Publish repeatedly failing records to a dead-letter topic.
7. Demonstrate how Kafka offsets and delivery guarantees work.

### Prerequisites

- Java 21.
- Maven 3.6.3 or later.
- Docker and Docker Compose.
- An IDE such as IntelliJ IDEA or VS Code.

### Technology stack

| Technology | Purpose |
|---|---|
| Java 21 | Application code |
| Spring Boot 3.5.x | Application framework |
| Spring Web | REST API |
| Spring Kafka | Producer and consumer integration |
| Apache Kafka | Event streaming |
| Jackson | JSON serialization and deserialization |
| Docker Compose | Local Kafka environment |
| Maven | Build and dependency management |

## 6. Create the Spring Boot Project

Generate a Maven project using [Spring Initializr](https://start.spring.io/).

Select:

- Project: Maven
- Language: Java
- Java: 21
- Spring Boot: a compatible 3.5.x release
- Packaging: Jar

Add these dependencies:

- Spring Web
- Validation
- Spring for Apache Kafka

The corresponding Maven dependencies are shown below.

### `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
           http://maven.apache.org/POM/4.0.0
           https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.6</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>kafka-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <name>kafka-demo</name>
    <description>Spring Boot Kafka demonstration</description>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

The Spring Boot parent manages compatible dependency versions. Do not independently override the Kafka client or Spring Kafka version unless there is a specific compatibility requirement.

## 7. Project Structure

Create the following files.

```text
kafka-demo/
├── compose.yaml
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── example/
        │           └── kafkademo/
        │               ├── KafkaDemoApplication.java
        │               ├── config/
        │               │   ├── KafkaTopicConfig.java
        │               │   └── KafkaConsumerConfig.java
        │               ├── controller/
        │               │   └── OrderController.java
        │               ├── dto/
        │               │   ├── CreateOrderRequest.java
        │               │   └── CreateOrderResponse.java
        │               ├── events/
        │               │   └── OrderCreatedEvent.java
        │               ├── producer/
        │               │   └── OrderEventProducer.java
        │               └── consumer/
        │                   └── OrderEventConsumer.java
        └── resources/
            └── application.yml
```

Each class has one clear responsibility:

- `KafkaTopicConfig`: creates the required topics.
- `KafkaConsumerConfig`: configures retries and dead-letter publishing.
- `OrderController`: exposes the REST endpoint.
- `OrderEventProducer`: publishes order events.
- `OrderEventConsumer`: processes order events.
- `OrderCreatedEvent`: defines the event contract.

## 8. Start Kafka Using Docker Compose

Create `compose.yaml` in the project root.

This configuration runs a single Kafka broker in KRaft mode, without ZooKeeper.

```yaml
services:
  kafka:
    image: apache/kafka:4.0.0
    container_name: kafka-demo-broker

    ports:
      - "9092:9092"

    environment:
      KAFKA_NODE_ID: 1

      KAFKA_PROCESS_ROLES: broker,controller

      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093

      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092

      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER

      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT

      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@localhost:9093

      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1

      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1

      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0

    healthcheck:
      test:
        - CMD-SHELL
        - /opt/kafka/bin/kafka-broker-api-versions.sh --bootstrap-server localhost:9092
      interval: 10s
      timeout: 10s
      retries: 10
      start_period: 15s
```

Start Kafka:

```bash
docker compose up -d
```

Check the container:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f kafka
```

Stop Kafka:

```bash
docker compose down
```

This is a development-only, single-broker configuration. It does not provide broker redundancy. The example also uses plaintext communication without authentication, so do not expose it to an untrusted network.

For production, use an appropriately secured multi-broker cluster with replication, monitoring, access controls, and a tested recovery plan.

## 9. Configure Spring Kafka

Create `src/main/resources/application.yml`.

```yaml
spring:
  application:
    name: kafka-demo

  kafka:
    bootstrap-servers: ${KAFKA_BOOTSTRAP_SERVERS:localhost:9092}

    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer

      acks: all
      retries: 2147483647

      properties:
        enable.idempotence: true
        max.in.flight.requests.per.connection: 5

    consumer:
      group-id: order-service
      auto-offset-reset: earliest
      enable-auto-commit: false

      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer

      properties:
        "[spring.json.value.default.type]": com.example.kafkademo.events.OrderCreatedEvent
        "[spring.json.trusted.packages]": com.example.kafkademo.events

    listener:
      ack-mode: record

server:
  port: 8080

logging:
  level:
    com.example.kafkademo: INFO
```

### Important configuration properties

**`bootstrap-servers`**

Specifies the Kafka broker address used to establish the initial connection.

**`JsonSerializer`**

Serializes Java event objects into JSON records.

**`JsonDeserializer`**

Deserializes JSON records into the specified Java event type.

**`spring.json.value.default.type`**

Specifies the target Java type for incoming event values in this example.

**`spring.json.trusted.packages`**

Restricts the Java packages trusted by the JSON deserializer. Trust only the packages needed by your application; do not use `*` indiscriminately.

**`acks: all`**

Requests acknowledgement from the leader after the required in-sync replicas have acknowledged the write.

**`enable.idempotence: true`**

Prevents duplicate Kafka log entries caused by retries from the same idempotent producer session, within Kafka's documented guarantees.

**`enable-auto-commit: false`**

Disables periodic background offset commits. Spring Kafka's listener container manages commits according to the configured acknowledgement mode.

**`ack-mode: record`**

Commits the offset after the listener successfully processes an individual record, under the normal record-listener flow. A failure must propagate to the container for its error-handling mechanism to take effect.

These settings improve reliability, but they do not guarantee exactly-once processing of database updates or external API calls.

## 10. Main Spring Boot Application

Create `KafkaDemoApplication.java`.

```java
package com.example.kafkademo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class KafkaDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(
            KafkaDemoApplication.class,
            args
        );
    }
}
```

Spring Boot automatically configures the Kafka infrastructure from `application.yml` and the `spring-kafka` dependency.

## 11. Create the Kafka Topics

Create `config/KafkaTopicConfig.java`.

```java
package com.example.kafkademo.config;

import org.apache.kafka.clients.admin.NewTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.TopicBuilder;

@Configuration(proxyBeanMethods = false)
public class KafkaTopicConfig {

    public static final String ORDER_EVENTS =
        "order-events";

    public static final String ORDER_EVENTS_DLT =
        "order-events.DLT";

    @Bean
    public NewTopic orderEventsTopic() {
        return TopicBuilder
            .name(ORDER_EVENTS)
            .partitions(3)
            .replicas(1)
            .build();
    }

    @Bean
    public NewTopic orderEventsDeadLetterTopic() {
        return TopicBuilder
            .name(ORDER_EVENTS_DLT)
            .partitions(3)
            .replicas(1)
            .build();
    }
}
```

### Explanation

`NewTopic` declares a topic that Spring Kafka can create through Kafka's administration API when the application starts, provided the application has the required permissions and the broker is available.

`partitions(3)` creates three partitions, allowing up to three consumers in one consumer group to process these partitions concurrently.

`replicas(1)` is appropriate only for the single-broker local setup. Production topics should use a replication factor compatible with the cluster size and durability requirements, commonly three when three suitable brokers are available.

The dead-letter topic has three partitions so that records can be routed to corresponding partitions when using the default dead-letter partitioning behavior.

## 12. Define the Event Model

Create `events/OrderCreatedEvent.java`.

```java
package com.example.kafkademo.events;

import java.math.BigDecimal;
import java.time.Instant;
import java.util.UUID;

public record OrderCreatedEvent(
    UUID eventId,
    UUID orderId,
    String customerId,
    BigDecimal totalAmount,
    Instant occurredAt
) {
}
```

This Java record defines the JSON event contract.

Example event:

```json
{
  "eventId": "7a95cb89-26d0-47af-b1fd-849a9340a001",
  "orderId": "b08cb1e2-cd8f-4eaf-97d7-a56e8e900002",
  "customerId": "customer-101",
  "totalAmount": 2499.00,
  "occurredAt": "2026-10-19T08:30:00Z"
}
```

The fields have distinct meanings:

- `eventId`: identifies this particular event.
- `orderId`: identifies the business order and is used as the Kafka key.
- `customerId`: identifies the customer.
- `totalAmount`: represents the order amount.
- `occurredAt`: indicates when the event was created.

The event ID is useful for application-level deduplication. The order ID is useful for partition routing and preserving the relative order of events for the same order, provided the same partitioning strategy is used.

In a production system, event schemas should be versioned and evolved compatibly. A schema registry can help enforce compatibility when many services share event contracts.

## 13. Create Request and Response DTOs

Create `dto/CreateOrderRequest.java`.

```java
package com.example.kafkademo.dto;

import jakarta.validation.constraints.DecimalMin;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;

import java.math.BigDecimal;

public record CreateOrderRequest(

    @NotBlank
    String customerId,

    @NotNull
    @DecimalMin(value = "0.01")
    BigDecimal totalAmount

) {
}
```

This validates the REST request before the event is published.

Create `dto/CreateOrderResponse.java`.

```java
package com.example.kafkademo.dto;

import java.util.UUID;

public record CreateOrderResponse(
    UUID eventId,
    UUID orderId,
    String status
) {
}
```

The response identifies the published event and the order.

In this demonstration, the endpoint publishes an event rather than persisting a complete order in a database. A real Order Service would generally persist its authoritative order state and use an appropriate transaction/outbox design before publishing the event.

## 14. Implement the Kafka Producer

Create `producer/OrderEventProducer.java`.

```java
package com.example.kafkademo.producer;

import com.example.kafkademo.config.KafkaTopicConfig;
import com.example.kafkademo.events.OrderCreatedEvent;

import org.apache.kafka.clients.producer.RecordMetadata;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Service;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

@Service
public class OrderEventProducer {

    private final KafkaTemplate<Object, Object> kafkaTemplate;

    public OrderEventProducer(
            KafkaTemplate<Object, Object> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public RecordMetadata publish(OrderCreatedEvent event) {
        try {
            SendResult<Object, Object> result =
                kafkaTemplate.send(
                    KafkaTopicConfig.ORDER_EVENTS,
                    event.orderId().toString(),
                    event
                ).get(5, TimeUnit.SECONDS);

            RecordMetadata metadata =
                result.getRecordMetadata();

            return metadata;

        } catch (InterruptedException exception) {
            Thread.currentThread().interrupt();

            throw new KafkaPublishException(
                "Publishing the order event was interrupted",
                exception
            );

        } catch (ExecutionException | TimeoutException exception) {
            throw new KafkaPublishException(
                "Unable to confirm publishing the order event",
                exception
            );
        }
    }
}
```

Create `producer/KafkaPublishException.java`.

```java
package com.example.kafkademo.producer;

public class KafkaPublishException extends RuntimeException {

    public KafkaPublishException(
            String message,
            Throwable cause) {
        super(message, cause);
    }
}
```

### How the producer works

`KafkaTemplate` publishes the event to `order-events`.

The second argument to `send()` is the message key:

```java
event.orderId().toString()
```

Records with the same order ID are normally routed to the same partition. This helps preserve their order within that partition.

The `get(5, TimeUnit.SECONDS)` call waits for the broker's send result for at most five seconds. It is used here to make the example's success response straightforward.

A timeout does **not** prove that Kafka rejected the record. Kafka may have stored the record while the acknowledgement was delayed or lost. Retrying the entire HTTP request can therefore publish another business event unless the application has an idempotency mechanism.

For high-throughput production APIs, an asynchronous send callback or future is often preferable to blocking a request thread.

The returned `RecordMetadata` contains the topic, partition, and offset assigned to the published record.

## 15. Implement the Kafka Consumer

Create `consumer/OrderEventConsumer.java`.

```java
package com.example.kafkademo.consumer;

import com.example.kafkademo.config.KafkaTopicConfig;
import com.example.kafkademo.events.OrderCreatedEvent;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Component;

@Component
public class OrderEventConsumer {

    private static final Logger log =
        LoggerFactory.getLogger(OrderEventConsumer.class);

    @KafkaListener(
        id = "order-event-consumer",
        topics = KafkaTopicConfig.ORDER_EVENTS,
        groupId = "order-service"
    )
    public void consume(
            ConsumerRecord<String, OrderCreatedEvent> record,
            @Header(KafkaHeaders.RECEIVED_KEY) String key,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset) {

        OrderCreatedEvent event = record.value();

        log.info(
            "Received order event: eventId={}, orderId={}, " +
            "customerId={}, key={}, partition={}, offset={}",
            event.eventId(),
            event.orderId(),
            event.customerId(),
            key,
            partition,
            offset
        );

        processOrderEvent(event);
    }

    private void processOrderEvent(OrderCreatedEvent event) {
        // Implement the actual business operation here.
        log.info(
            "Processing order {} with amount {}",
            event.orderId(),
            event.totalAmount()
        );
    }
}
```

### Explanation

`@KafkaListener` registers a method as a Kafka message listener.

The method receives:

- The Kafka record key.
- The deserialized `OrderCreatedEvent`.
- The partition.
- The offset.

In this example, `processOrderEvent()` logs the event. A real consumer would execute a business operation such as updating a database or initiating a workflow.

**Important:** Do not catch an exception and silently ignore it when the operation has failed. The exception should reach the listener container so that the configured retry and dead-letter behavior can run.

## 16. Configure Retry and Dead-Letter Handling

A consumer can fail because of a temporary database outage, a downstream service timeout, or invalid message data.

Retrying every failure indefinitely can block progress. A dead-letter topic provides a place to send records that repeatedly fail processing.

Create `config/KafkaConsumerConfig.java`.

```java
package com.example.kafkademo.config;

import org.apache.kafka.common.TopicPartition;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.listener.DefaultErrorHandler;
import org.springframework.kafka.listener.DeadLetterPublishingRecoverer;

import org.springframework.util.backoff.FixedBackOff;

@Configuration(proxyBeanMethods = false)
public class KafkaConsumerConfig {

    @Bean
    public DefaultErrorHandler kafkaErrorHandler(
            KafkaTemplate<Object, Object> kafkaTemplate) {

        DeadLetterPublishingRecoverer recoverer =
            new DeadLetterPublishingRecoverer(
                kafkaTemplate,
                (record, exception) ->
                    new TopicPartition(
                        record.topic() + ".DLT",
                        record.partition()
                    )
            );

        FixedBackOff backOff =
            new FixedBackOff(1_000L, 2L);

        return new DefaultErrorHandler(
            recoverer,
            backOff
        );
    }
}
```

Spring Kafka's listener container detects this `DefaultErrorHandler` bean and uses it for listener failures under the standard auto-configured listener setup.

### What happens when processing fails?

With `FixedBackOff(1_000L, 2L)`:

1. The listener receives the record.
2. If processing fails, the record is retried after a one-second delay.
3. If it fails again, another retry occurs after a one-second delay.
4. If processing still fails, the recoverer publishes the record to `order-events.DLT`.
5. After successful recovery, the failed record can be skipped on the original topic and processing can continue.

This is two retries after the initial delivery, not two attempts in total.

The error handler's recovery and offset behavior depends on the listener container configuration. If dead-letter publishing fails, the recovery should not be treated as successful.

For production, use exponential backoff and jitter where appropriate, distinguish transient from permanent errors, monitor DLT volume, and define an operational process for replaying recovered records.

### Why use a dead-letter topic?

A poison message may fail every time because its contents are invalid or because a business rule cannot be satisfied.

Without an appropriate recovery policy, the same record may repeatedly fail and delay processing of later records in that partition.

A dead-letter topic preserves the failed record for investigation, correction, or controlled replay.

A DLT is not a substitute for fixing the underlying problem. It must be monitored and managed.

## 17. Create the REST Controller

Create `controller/OrderController.java`.

```java
package com.example.kafkademo.controller;

import com.example.kafkademo.dto.CreateOrderRequest;
import com.example.kafkademo.dto.CreateOrderResponse;
import com.example.kafkademo.events.OrderCreatedEvent;
import com.example.kafkademo.producer.OrderEventProducer;

import jakarta.validation.Valid;

import org.apache.kafka.clients.producer.RecordMetadata;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.time.Instant;
import java.util.UUID;

@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    private final OrderEventProducer producer;

    public OrderController(OrderEventProducer producer) {
        this.producer = producer;
    }

    @PostMapping
    public ResponseEntity<CreateOrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request) {

        OrderCreatedEvent event = new OrderCreatedEvent(
            UUID.randomUUID(),
            UUID.randomUUID(),
            request.customerId(),
            request.totalAmount(),
            Instant.now()
        );

        RecordMetadata metadata = producer.publish(event);

        CreateOrderResponse response =
            new CreateOrderResponse(
                event.eventId(),
                event.orderId(),
                "PUBLISHED"
            );

        return ResponseEntity
            .status(HttpStatus.ACCEPTED)
            .header(
                "X-Kafka-Partition",
                String.valueOf(metadata.partition())
            )
            .header(
                "X-Kafka-Offset",
                String.valueOf(metadata.offset())
            )
            .body(response);
    }
}
```

### Explanation

The endpoint accepts a JSON request, validates it, constructs an event, and publishes the event to Kafka.

If the send result is received successfully, it returns HTTP `202 Accepted`.

The `PUBLISHED` status means the producer received a successful Kafka acknowledgement. It does **not** mean that the consumer has completed the order workflow.

This distinction matters in asynchronous architectures.

In a real application, the order ID would normally be created as part of the authoritative order workflow, and the application would persist the order before acknowledging the request. The simplified example generates the order ID solely to demonstrate Kafka messaging.

## 18. Run the Complete Application

First, start Kafka:

```bash
docker compose up -d
```

Then build the project:

```bash
mvn clean package
```

Start Spring Boot:

```bash
mvn spring-boot:run
```

The application will start on port `8080`.

### Publish an event using curl

```bash
curl -X POST http://localhost:8080/api/v1/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "customer-101",
    "totalAmount": 2499.00
  }'
```

Example response:

```json
{
  "eventId": "7a95cb89-26d0-47af-b1fd-849a9340a001",
  "orderId": "b08cb1e2-cd8f-4eaf-97d7-a56e8e900002",
  "status": "PUBLISHED"
}
```

The UUID values will be different for each request.

Check the Spring Boot logs. The consumer should log the event, including its partition and offset.

## 19. Inspect Kafka Topics from the Command Line

List topics:

```bash
docker exec kafka-demo-broker \
  /opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --list
```

Describe the order topic:

```bash
docker exec kafka-demo-broker \
  /opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --topic order-events
```

Read records using a console consumer:

```bash
docker exec -it kafka-demo-broker \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-events \
  --from-beginning
```

The console consumer above reads raw record values. Because the producer sends JSON, the output should be readable JSON.

Stop the console consumer with `Ctrl+C`.

The console consumer uses its own consumer group when no group is specified. It does not consume records on behalf of the Spring Boot consumer group.

## 20. Consumer Groups and Parallel Processing

A consumer group distributes partitions among its members.

Suppose `order-events` has three partitions.

### Three consumers

```text
Topic: order-events

Partition 0 --------> Consumer 1
Partition 1 --------> Consumer 2
Partition 2 --------> Consumer 3

Consumer group: order-service
```

All three consumers can work in parallel.

### Five consumers

```text
Partition 0 --------> Consumer 1
Partition 1 --------> Consumer 2
Partition 2 --------> Consumer 3

Consumer 4 --------> Idle
Consumer 5 --------> Idle
```

There is no additional partition-level parallelism for this topic until the number of partitions increases or another topic is consumed.

### Two independent consumer groups

```text
                 order-events
                       |
              +--------+--------+
              |                 |
              v                 v
      order-service       analytics-service
              |                 |
        Processes orders   Processes analytics
```

Each group tracks its own offsets and receives its own logical stream of records.

A new consumer group does not automatically start from the beginning in every situation. The initial position depends on committed offsets and `auto-offset-reset`. In this example, `earliest` applies when no valid committed offset exists.

## 21. Kafka Delivery Guarantees

Kafka supports different delivery and processing models. It is important to distinguish producing a record from processing it in an application.

### At-most-once processing

A record is not processed more than once, but it may be lost from the application's processing perspective.

For example, committing an offset before completing the business operation can cause the record to be skipped after a failure.

### At-least-once processing

A record is retried until processing succeeds or a configured recovery policy takes over, but it may be processed more than once.

This is a common consumer model.

For example:

1. The consumer processes an event.
2. The database update succeeds.
3. The application crashes before the offset is committed.
4. Kafka delivers the record again after restart.
5. The business operation runs a second time unless it is idempotent.

### Exactly-once semantics

Kafka transactions can provide exactly-once processing for supported Kafka-to-Kafka workflows when the producer, consumer offsets, transaction configuration, and downstream reads are configured correctly.

However, Kafka transactions do not automatically make arbitrary database updates, emails, payment provider calls, or other external side effects exactly-once.

Those operations need their own transactional or idempotency strategy.

## 22. How to Avoid Message Loss and Duplicates

Reliability requires coordinated configuration across the producer, broker, and consumer.

### Producer settings

```yaml
spring:
  kafka:
    producer:
      acks: all
      retries: 2147483647
      properties:
        enable.idempotence: true
```

`acks=all` requests acknowledgement from the current in-sync replica set. The durability guarantee still depends on cluster configuration and the failure scenario.

Idempotent producer support prevents duplicate log entries caused by retries within the producer's Kafka protocol guarantees. It does not deduplicate separate application-level requests that publish the same logical event.

### Broker settings

For a suitably sized production cluster, a common configuration is:

```text
replication.factor=3
min.insync.replicas=2
```

These settings mean that a topic partition has three replicas, while writes using `acks=all` require at least two in-sync replicas to be available for a successful write.

The topic's replication factor and the broker's minimum in-sync replica setting must be configured consistently.

A single-broker local environment cannot provide the same redundancy as a three-broker production cluster.

### Consumer settings

```yaml
spring:
  kafka:
    consumer:
      enable-auto-commit: false
    listener:
      ack-mode: record
```

The listener container commits offsets after successful record processing under the normal record-listener flow.

For at-least-once processing, commit progress only after the business operation has succeeded. If a record fails, let the configured error handler determine retry and recovery behavior.

### Idempotent business processing

Use a stable event ID and a durable deduplication mechanism.

For example:

```sql
CREATE TABLE processed_events (
    event_id UUID PRIMARY KEY,
    processed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

A production consumer can insert the event ID and apply the business update in the same database transaction.

If the event ID already exists, the consumer knows that the event has been processed. The unique constraint protects against concurrent duplicate deliveries.

The deduplication record and business update must be committed atomically; otherwise, the application can still end up with inconsistent state.

## 23. Dead-Letter Topics vs. Retry Topics

These patterns serve different purposes.

| Pattern | Purpose |
|---|---|
| Blocking retry | Retries the record in the listener's processing flow |
| Dead-letter topic | Stores a record that exhausted its configured retries |
| Retry topic | Routes a record to another topic for delayed reprocessing |
| Manual replay | Allows an operator or controlled tool to reprocess failed records |

The example uses blocking retries followed by a dead-letter topic.

This is straightforward, but a long retry delay can hold up processing for a partition, depending on the listener container and error-handling configuration.

For workflows requiring longer delays without holding up normal processing, non-blocking retry topics may be appropriate. They introduce additional topic management and ordering considerations.

## 24. Kafka Transactions

Kafka transactions allow a producer to write records to multiple Kafka partitions atomically. They can also be used to coordinate consumed Kafka records and produced Kafka records in a transactional consume-process-produce workflow.

Spring Kafka supports transactions through a transaction-capable producer factory and `KafkaTransactionManager`.

With Spring Boot, transactional producer support can be enabled by configuring a unique transaction ID prefix:

```yaml
spring:
  kafka:
    producer:
      transaction-id-prefix: ${KAFKA_TRANSACTION_ID_PREFIX:kafka-demo-${random.uuid}-}
```

Each concurrently running application instance must use a transaction ID prefix that is unique to that instance.

When using transactional listeners, consumers that must not see aborted transactional records should use `read_committed` isolation. Transactional listener configuration and offset management must also be designed together.

**Important:** Do not add transactional configuration merely to claim exactly-once processing. Choose it when you need the guarantees it provides and understand how it interacts with retries, consumer offsets, and downstream side effects.

## 25. Kafka vs. RabbitMQ

| Feature | Apache Kafka | RabbitMQ |
|---|---|---|
| Core abstraction | Partitioned event log | Exchanges and queues |
| Typical use | Event streaming, replay, analytics | Task queues and message routing |
| Retention | Records retained by topic policy | Queue messages generally removed according to acknowledgement and queue policy |
| Replay | Read retained records again using offsets | Different queue and replay mechanisms |
| Ordering | Per partition | Depends on queue and consumer configuration |
| Scaling | Partitions and consumer groups | Queues, consumers, and broker topology |
| Best fit | High-volume retained event streams | Flexible routing and work distribution |

Neither technology is universally better. Choose based on throughput, routing needs, retention, replay, operational requirements, and delivery guarantees.

## 26. Kafka Best Practices

1. Choose partition keys based on the ordering and distribution requirements.
2. Use a stable event ID for business-level deduplication.
3. Configure producer acknowledgements and idempotence deliberately.
4. Set replication and minimum in-sync replica policies appropriate for production.
5. Disable automatic offset commits when using controlled listener processing.
6. Do not swallow exceptions that should trigger retries.
7. Use a dead-letter topic and monitor it.
8. Keep event schemas backward compatible.
9. Monitor consumer lag, processing failures, and broker health.
10. Avoid long-running blocking work inside consumers when it interferes with polling and partition progress.
11. Make database updates and external side effects idempotent.
12. Use the transactional outbox pattern when a database update and event publication must not become inconsistent.
13. Secure Kafka with authentication, authorization, and encryption in production.
14. Use multiple brokers and test failover when production availability requires it.
15. Treat event replay as an operational workflow that needs safeguards.

## 27. Interview Questions and Answers

### Q1. What is Apache Kafka?

Kafka is a distributed event streaming platform that stores records in partitioned logs and allows producers and consumers to process event streams at scale.

### Q2. What is the difference between a topic and a partition?

A topic is a named stream of records. A partition is an ordered log within that topic, used for parallelism and distributed storage.

### Q3. What is an offset?

An offset identifies a record's position within a partition. Consumer groups use committed offsets to track their progress.

### Q4. Can Kafka guarantee message ordering?

Kafka guarantees ordering within a partition. It does not guarantee global ordering across all partitions of a topic.

### Q5. What happens when there are more consumers than partitions?

Within one consumer group, some consumers will remain idle for that topic because each partition is assigned to at most one consumer at a time.

### Q6. What is a consumer group?

A consumer group is a set of consumers that cooperate to process topic partitions. Different groups track their progress independently.

### Q7. How do you avoid message loss?

Use suitable producer acknowledgements, idempotence, appropriate replication and in-sync replica settings, and consumer offset handling that reflects successful business processing.

### Q8. Does Kafka guarantee exactly-once processing?

Kafka supports exactly-once semantics for properly configured Kafka transactional workflows. It does not automatically guarantee exactly-once external side effects.

### Q9. Why is idempotency important?

A consumer may receive the same record more than once. Idempotency ensures repeated processing does not repeat the intended business effect.

### Q10. What is a dead-letter topic?

A dead-letter topic stores records that could not be processed successfully under the configured error-handling policy. It supports investigation and controlled recovery.

### Q11. What does `acks=all` mean?

The producer waits for acknowledgement from the current in-sync replica set before treating the write as successful. The effective durability depends on the cluster's replication configuration.

### Q12. What is the difference between Kafka and a traditional message queue?

Kafka is based on retained, partitioned event logs with consumer offsets. Traditional queue-based messaging often removes messages after acknowledgement, although actual behavior depends on the product and configuration.

### Q13. Why use Spring Kafka?

Spring Kafka simplifies Java integration through `KafkaTemplate`, `@KafkaListener`, listener containers, error handling, and transaction support.

### Q14. What happens when a consumer crashes after processing but before committing the offset?

The record can be delivered again after the consumer restarts or its partition is reassigned. An idempotent business operation prevents duplicate effects.

### Q15. What is the transactional outbox pattern?

The application writes its business data and an outbox event record in one database transaction. A separate publisher sends the event to Kafka, preventing the database update and event creation from being independent writes.

## Conclusion

Kafka enables scalable, asynchronous communication between microservices. Spring Kafka makes it straightforward to publish events with `KafkaTemplate` and consume them with `@KafkaListener`.

A reliable implementation needs more than a producer and consumer. It must account for partitioning, offsets, replication, retries, dead-letter handling, idempotency, schema evolution, and the difference between Kafka delivery guarantees and business-level processing guarantees.

Start with the complete local project in this guide, verify that an order event reaches the consumer, and then extend it with database persistence, an idempotent consumer, and the transactional outbox pattern.

For official references, see the [Spring for Apache Kafka documentation](https://docs.spring.io/spring-kafka/reference/), [Apache Kafka producer configuration](https://kafka.apache.org/40/configuration/producer-configs/), and [Spring Boot system requirements](https://docs.spring.io/spring-boot/3.5/system-requirements.html).
