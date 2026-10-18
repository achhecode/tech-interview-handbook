---
title: Observer
parent: Design Patterns and LLD
nav_order: 9
---

# Observer Pattern
{: .no_toc }

Defines a one-to-many dependency so observers are notified when a subject's state changes.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the Observer Design Pattern?

The **Observer Pattern** is a behavioral design pattern in which one object, called the **Subject** or publisher, maintains a collection of dependent objects called **Observers** or subscribers.

When the subject's state changes or an event occurs, it notifies its registered observers.

**Real-world example:** When a user subscribes to a YouTube channel, they may receive notifications when new videos are published. One publisher can notify many subscribers.

### Key components

- **Subject / Publisher:** Maintains subscribers and sends notifications.
- **Observer / Subscriber:** Defines the notification method.
- **Concrete Subject:** Produces events or changes state.
- **Concrete Observer:** Implements the action to perform when notified.
- **Client:** Registers or removes observers.

---

## How do you implement the Observer Pattern in Java?

Let's build a notification system that notifies subscribers whenever a channel publishes a new video.

### Step 1: Create the Observer interface

```java
public interface Observer {
    void update(String videoTitle);
}
```

Every subscriber must implement the `update()` method.

### Step 2: Create Concrete Observers

```java
public class EmailSubscriber implements Observer {

    private final String email;

    public EmailSubscriber(String email) {
        this.email = email;
    }

    @Override
    public void update(String videoTitle) {
        System.out.println(
            "Email sent to " + email
            + ": New video - " + videoTitle
        );
    }
}
```

```java
public class MobileSubscriber implements Observer {

    private final String mobileNumber;

    public MobileSubscriber(String mobileNumber) {
        this.mobileNumber = mobileNumber;
    }

    @Override
    public void update(String videoTitle) {
        System.out.println(
            "Push notification sent to " + mobileNumber
            + ": New video - " + videoTitle
        );
    }
}
```

Each observer reacts to the same event differently.

### Step 3: Create the Subject

```java
import java.util.ArrayList;
import java.util.List;

public class YouTubeChannel {

    private final List<Observer> observers = new ArrayList<>();
    private String latestVideo;

    public void subscribe(Observer observer) {
        observers.add(observer);
    }

    public void unsubscribe(Observer observer) {
        observers.remove(observer);
    }

    public void uploadVideo(String videoTitle) {
        this.latestVideo = videoTitle;
        notifyObservers();
    }

    private void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(latestVideo);
        }
    }
}
```

The channel maintains a list of subscribers and notifies them when a new video is uploaded.

### Step 4: Use the Observer Pattern

```java
public class Main {

    public static void main(String[] args) {

        YouTubeChannel channel = new YouTubeChannel();

        Observer subscriber1 =
            new EmailSubscriber("user@example.com");

        Observer subscriber2 =
            new MobileSubscriber("+911234567890");

        channel.subscribe(subscriber1);
        channel.subscribe(subscriber2);

        channel.uploadVideo("Observer Pattern in Java");

        channel.unsubscribe(subscriber1);

        channel.uploadVideo("Spring Boot Events");
    }
}
```

**Output**

```text
Email sent to user@example.com: New video - Observer Pattern in Java
Push notification sent to +911234567890: New video - Observer Pattern in Java
Push notification sent to +911234567890: New video - Spring Boot Events
```

The email subscriber receives the first notification but not the second because it unsubscribed.

---

## How does the Observer Pattern work?

1. Observers register with the subject.
2. The subject maintains a collection of observers.
3. An event occurs or the subject's state changes.
4. The subject notifies registered observers.
5. Each observer executes its own response.
6. Observers can unsubscribe when they no longer need notifications.

The subject does not need to know the concrete classes or internal behavior of its observers. It only depends on the observer interface.

---

## What problem does the Observer Pattern solve?

Suppose an order service must notify customers, update analytics, and send warehouse events when an order is placed.

A tightly coupled implementation might call each service directly:

```java
public void placeOrder(Order order) {
    saveOrder(order);
    sendEmail(order);
    updateAnalytics(order);
    notifyWarehouse(order);
}
```

As more reactions are added, the order service becomes responsible for coordinating many unrelated operations.

With Observer, the order service publishes an event, and interested observers handle their own responses.

```text
                Order Service
                     |
                     v
               Order Placed
                     |
          +----------+----------+
          |          |          |
          v          v          v
     Email Alert  Analytics  Warehouse
```

This reduces direct coupling between the publisher and its subscribers.

---

## Observer Pattern with Spring Boot

Spring provides application events that support the Observer Pattern.

An event publisher publishes an event, and listener methods react to it.

### Step 1: Create the event

```java
public record OrderPlacedEvent(
    Long orderId,
    String customerEmail
) {
}
```

This event contains information about the newly placed order.

### Step 2: Publish the event

```java
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    private final ApplicationEventPublisher eventPublisher;

    public OrderService(
            ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    public void placeOrder(Long orderId, String email) {

        // Save the order to the database.

        eventPublisher.publishEvent(
            new OrderPlacedEvent(orderId, email)
        );
    }
}
```

`ApplicationEventPublisher` publishes the event to interested listeners in the Spring application context.

### Step 3: Create an event listener

```java
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class EmailNotificationListener {

    @EventListener
    public void handleOrderPlaced(OrderPlacedEvent event) {
        System.out.println(
            "Sending order confirmation to "
            + event.customerEmail()
        );
    }
}
```

### Step 4: Add another observer

```java
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class AnalyticsListener {

    @EventListener
    public void handleOrderPlaced(OrderPlacedEvent event) {
        System.out.println(
            "Recording order " + event.orderId()
            + " in analytics"
        );
    }
}
```

Now, when `OrderService` publishes `OrderPlacedEvent`, both listener methods can react to it.

**Important:** Spring's standard `@EventListener` mechanism is synchronous by default. The publisher generally waits for the listeners to finish. Adding `@Async` can make supported listeners execute asynchronously when asynchronous processing is enabled and configured.

### Does Spring application-event publishing guarantee database consistency?

No. Publishing an in-memory event does not automatically guarantee that the event is delivered reliably after a database transaction commits or survives an application crash.

For transactional use cases, consider `@TransactionalEventListener` with an appropriate transaction phase. For reliable delivery across database and message-broker boundaries, consider the **Transactional Outbox Pattern**.

---

## Observer vs. Pub/Sub

Observer and Publish-Subscribe are related, but they are not exactly the same.

| Feature | Observer | Publish-Subscribe |
|---|---|---|
| Communication | Usually direct notification between objects | Usually mediated by a message broker or event channel |
| Coupling | Subject knows registered observer references or interfaces | Publisher and subscribers can be independent |
| Scope | Often within one application or process | Can span processes and services |
| Example | Java listener collection | Kafka topic with multiple consumer groups |
| Delivery | Depends on implementation | Depends on broker, configuration, and acknowledgment model |

A simple Java observer list is usually in-memory and does not provide durable event delivery. A message broker can support persistence, retries, and independent consumers when configured appropriately.

---

## Observer vs. Strategy

| Feature | Observer | Strategy |
|---|---|---|
| Purpose | Notifies multiple interested objects | Selects an algorithm |
| Relationship | One subject can notify many observers | A context typically uses one selected strategy at a time |
| Trigger | Event or state change | An operation requiring a chosen algorithm |
| Example | Notify subscribers when an order is placed | Choose a shipping-cost calculation method |

---

## Observer vs. Chain of Responsibility

| Feature | Observer | Chain of Responsibility |
|---|---|---|
| Purpose | Broadcasts an event to interested observers | Passes a request through a sequence of handlers |
| Execution | Multiple observers may be notified | A handler may process or pass the request onward |
| Coupling | Publisher is decoupled from observer implementations | Each handler is commonly connected to the next handler |
| Example | Notify analytics and email services | Process an approval request through multiple approval levels |

---

## Advantages of the Observer Pattern

1. **Loose coupling:** The subject does not depend on concrete observer implementations.
2. **Extensibility:** New observers can be added without changing the subject.
3. **Separation of concerns:** Each observer handles its own responsibility.
4. **One-to-many notifications:** A single event can trigger multiple actions.
5. **Dynamic subscriptions:** Observers can register and unregister at runtime.

## Disadvantages of the Observer Pattern

1. Notifications may trigger many operations, making the execution flow harder to follow.
2. A slow synchronous observer can delay the publisher.
3. Observer ordering and exception handling need careful consideration.
4. In-memory observers may be lost when the application restarts.
5. Incorrect subscription management can cause memory leaks or duplicate notifications.
6. Asynchronous observers introduce concurrency and delivery-order considerations.

---

## Common implementation concerns

### 1. Avoid duplicate subscriptions

The basic `ArrayList` implementation allows the same observer to be registered multiple times. If duplicate registrations are not intended, use a `Set` or check before adding.

### 2. Avoid concurrent modification

If observers can register or unregister while notifications are being sent, a normal `ArrayList` may be unsafe. Choose an appropriate concurrency strategy, such as taking a snapshot of observers before iterating or using a suitable concurrent collection.

### 3. Handle observer exceptions

If one observer throws an exception, a simple notification loop may stop before notifying the remaining observers. Decide whether failures should propagate, be logged individually, or be retried.

### 4. Consider asynchronous processing

For slow operations such as email delivery or analytics, asynchronous event processing can reduce request latency. However, asynchronous processing requires deliberate handling of failures, ordering, and observability.

---

## When should you use the Observer Pattern?

Use it when:

- One event needs to trigger multiple independent actions.
- Multiple components need to react to state changes.
- You want to reduce direct dependencies between components.
- Subscribers should be added or removed dynamically.
- You need event-driven behavior within an application.

Common examples include UI event listeners, order notifications, stock-price updates, audit events, domain events, and Spring application events.

For reliable communication across distributed services, use an appropriate messaging system such as Kafka or RabbitMQ rather than relying solely on an in-memory observer list.

---

## How does Observer relate to SOLID principles?

- **Single Responsibility Principle (SRP):** Each observer owns its own reaction to an event.
- **Open/Closed Principle (OCP):** New observers can be introduced without modifying the subject's notification logic.
- **Dependency Inversion Principle (DIP):** The subject works with an observer abstraction rather than concrete observer classes.

---

## Observer Pattern — Interview Revision

**Q1. What is the Observer Pattern?**

A behavioral design pattern in which a subject notifies registered observers when an event occurs or its state changes.

**Q2. What are its main components?**

Subject, Concrete Subject, Observer, Concrete Observer, and Client.

**Q3. What is the difference between a subject and an observer?**

The subject publishes notifications; the observer receives notifications and reacts to them.

**Q4. How does Spring Boot support the Observer Pattern?**

Through application events using `ApplicationEventPublisher` and listeners using `@EventListener`.

**Q5. Are Spring application events asynchronous by default?**

No. Standard event listeners execute synchronously by default. Asynchronous processing requires additional configuration, such as `@Async` with enabled async processing.

**Q6. What is the difference between Observer and Pub/Sub?**

Observer commonly uses direct in-process object references, while Pub/Sub commonly uses a message broker or intermediary channel.

**Q7. What happens if an observer throws an exception?**

In a simple synchronous notification loop, the exception can stop subsequent notifications unless failures are handled explicitly.

**Q8. Is the Observer Pattern suitable for microservices?**

The underlying event-driven idea is useful, but cross-service communication generally requires a message broker or another communication mechanism. A local observer list cannot provide distributed delivery guarantees.

**Q9. How can you prevent duplicate observer registration?**

Use a `Set`, or check whether the observer is already registered before adding it.

**Q10. Which SOLID principles does Observer support?**

Primarily SRP, OCP, and DIP.

---

## Final Summary

The **Observer Pattern** establishes a one-to-many relationship in which one publisher notifies multiple interested observers about events or state changes.

Remember the key example:

- `YouTubeChannel` — subject.
- `Observer` — subscriber interface.
- `EmailSubscriber` — concrete observer.
- `MobileSubscriber` — concrete observer.
- `uploadVideo()` — event that triggers notifications.

In Spring Boot, remember `ApplicationEventPublisher` and `@EventListener`.

**Interview takeaway:** Use Observer to notify multiple components about an event without tightly coupling the publisher to each component.
