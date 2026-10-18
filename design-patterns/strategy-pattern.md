---
title: Strategy
parent: Design Patterns and LLD
nav_order: 8
---

# Strategy Pattern
{: .no_toc }

Defines a family of algorithms, encapsulates each one, and makes them interchangeable at runtime.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the Strategy Design Pattern?

The **Strategy Pattern** is a behavioral design pattern that allows you to define multiple algorithms, encapsulate each algorithm in a separate class, and select the required algorithm at runtime.

Instead of implementing multiple behaviors using complex `if-else` or `switch` statements, you create interchangeable strategy implementations behind a common interface.

**Real-world example:** An e-commerce application supports credit card, UPI, and PayPal payments. Each payment method has a different implementation, but the checkout service can use any of them through the same interface.

### Key components

- **Strategy:** Interface that defines a common algorithm.
- **Concrete Strategy:** Class that implements a specific algorithm.
- **Context:** Uses a strategy without needing to know its implementation details.
- **Client:** Selects or configures the strategy.

---

## How do you implement the Strategy Pattern in Java?

Let's implement a payment system supporting multiple payment methods.

### Step 1: Create the Strategy interface

```java
public interface PaymentStrategy {
    void pay(double amount);
}
```

This interface defines the common contract for all payment methods.

### Step 2: Create Concrete Strategies

**Credit Card Payment**

```java
public class CreditCardPayment implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println(
            "Paid ₹" + amount + " using Credit Card"
        );
    }
}
```

**UPI Payment**

```java
public class UpiPayment implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println(
            "Paid ₹" + amount + " using UPI"
        );
    }
}
```

**PayPal Payment**

```java
public class PayPalPayment implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println(
            "Paid ₹" + amount + " using PayPal"
        );
    }
}
```

Each class implements the same interface but provides a different algorithm.

### Step 3: Create the Context

```java
public class CheckoutService {

    private final PaymentStrategy paymentStrategy;

    public CheckoutService(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void checkout(double amount) {
        paymentStrategy.pay(amount);
    }
}
```

The context delegates the payment operation to the selected strategy.

### Step 4: Use the Strategy

```java
public class Main {

    public static void main(String[] args) {

        PaymentStrategy upi = new UpiPayment();
        CheckoutService checkout = new CheckoutService(upi);
        checkout.checkout(1500);

        PaymentStrategy card = new CreditCardPayment();
        CheckoutService cardCheckout =
            new CheckoutService(card);

        cardCheckout.checkout(2500);
    }
}
```

**Output**

```text
Paid ₹1500.0 using UPI
Paid ₹2500.0 using Credit Card
```

### How does it work?

1. The client chooses a payment strategy.
2. The context receives the strategy through its constructor.
3. The context calls `pay()`.
4. The concrete strategy executes its own payment algorithm.

The checkout service does not need to know how UPI or credit card payments work.

---

## How do you change a Strategy at runtime?

If the application needs to change the algorithm dynamically, provide a setter or another method for replacing the strategy.

```java
public class CheckoutService {

    private PaymentStrategy paymentStrategy;

    public CheckoutService(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void setPaymentStrategy(
            PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void checkout(double amount) {
        paymentStrategy.pay(amount);
    }
}
```

Usage:

```java
CheckoutService checkout =
    new CheckoutService(new UpiPayment());

checkout.checkout(1000);

checkout.setPaymentStrategy(new CreditCardPayment());
checkout.checkout(2000);
```

This demonstrates selecting a different algorithm without changing the context's implementation.

In a real payment system, payment strategies must also handle validation, failures, retries, and secure payment-provider integration.

---

## What problem does the Strategy Pattern solve?

Consider a checkout service implemented with conditional logic:

```java
public void pay(String method, double amount) {

    if ("UPI".equalsIgnoreCase(method)) {
        // UPI payment logic
    } else if ("CARD".equalsIgnoreCase(method)) {
        // Card payment logic
    } else if ("PAYPAL".equalsIgnoreCase(method)) {
        // PayPal payment logic
    } else {
        throw new IllegalArgumentException(
            "Unsupported payment method"
        );
    }
}
```

As payment methods increase, the method becomes more complicated. Adding a new payment type requires modifying existing conditional logic.

With Strategy, each payment method is isolated in its own class. A new implementation can be added without changing the checkout service.

**Important:** Conditional statements are not always bad. Strategy is most useful when algorithms represent distinct behaviors that are likely to grow or change independently.

---

## Strategy Pattern with Spring Boot

Spring Boot can manage strategy implementations as beans and inject them into a service.

### Step 1: Create the interface

```java
public interface NotificationStrategy {
    void send(String message);
}
```

### Step 2: Create implementations

```java
import org.springframework.stereotype.Component;

@Component("email")
public class EmailNotificationStrategy
        implements NotificationStrategy {

    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}
```

```java
import org.springframework.stereotype.Component;

@Component("sms")
public class SmsNotificationStrategy
        implements NotificationStrategy {

    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}
```

### Step 3: Inject and select the strategy

```java
import java.util.Map;
import org.springframework.stereotype.Service;

@Service
public class NotificationService {

    private final Map<String, NotificationStrategy> strategies;

    public NotificationService(
            Map<String, NotificationStrategy> strategies) {
        this.strategies = strategies;
    }

    public void notifyUser(String type, String message) {
        NotificationStrategy strategy = strategies.get(type);

        if (strategy == null) {
            throw new IllegalArgumentException(
                "Unsupported notification type: " + type
            );
        }

        strategy.send(message);
    }
}
```

Spring can inject a map whose keys are bean names and whose values are the corresponding strategy beans.

Usage:

```java
notificationService.notifyUser("email", "Order confirmed");
notificationService.notifyUser("sms", "Your OTP is 123456");
```

For production OTP systems, never log or expose the actual OTP in plain text.

**Interview point:** A map of strategy beans is a common way to replace long `if-else` or `switch` blocks in Spring applications.

---

## Strategy vs. State Pattern

Both patterns can delegate behavior to separate classes, but their intent differs.

| Feature | Strategy | State |
|---|---|---|
| Purpose | Select an algorithm | Represent behavior that depends on an object's state |
| Selection | Often selected by the client or application logic | Often changes as the object's state changes |
| Context | Uses a selected algorithm | Maintains or delegates to its current state |
| Example | Choose UPI or card payment | Order moves from pending to shipped |
| Primary focus | Interchangeable behavior | State-dependent behavior |

---

## Strategy vs. Factory Pattern

| Feature | Strategy | Factory |
|---|---|---|
| Purpose | Encapsulates interchangeable algorithms | Encapsulates object creation |
| Main concern | How an operation is performed | Which object should be created |
| Output | Executes behavior through a strategy | Returns a created object |
| Example | Apply a discount algorithm | Create the correct payment strategy |

These patterns can be used together. A factory can select and create a strategy, while the strategy performs the actual operation.

---

## Advantages of the Strategy Pattern

1. **Open/Closed Principle:** New strategies can be added without modifying the context.
2. **Single Responsibility Principle:** Each strategy contains one algorithm or behavior.
3. **Reduced conditional logic:** Avoids large conditional blocks for interchangeable algorithms.
4. **Runtime flexibility:** A strategy can be replaced when required.
5. **Testability:** Strategies can be tested independently or replaced with test doubles.

## Disadvantages of the Strategy Pattern

1. Introduces additional interfaces and classes.
2. The client or configuration must select an appropriate strategy.
3. Can be excessive when there are only a few simple, stable alternatives.
4. Different strategies may require different configuration or input data.

---

## When should you use the Strategy Pattern?

Use it when:

- Multiple algorithms perform the same general task.
- The algorithm needs to be selected at runtime.
- Conditional logic is becoming difficult to maintain.
- New behaviors should be added independently.
- You need to test each algorithm in isolation.

Common examples include payment methods, discount calculations, shipping-cost calculations, authentication mechanisms, compression algorithms, and routing policies.

---

## How does Strategy relate to SOLID principles?

- **Single Responsibility Principle (SRP):** Each strategy owns a distinct algorithm.
- **Open/Closed Principle (OCP):** New strategies can be introduced without rewriting the context.
- **Dependency Inversion Principle (DIP):** The context depends on the `PaymentStrategy` abstraction rather than concrete payment classes.

---

## Strategy Pattern — Interview Revision

**Q1. What is the Strategy Pattern?**

A behavioral pattern that encapsulates interchangeable algorithms behind a common interface.

**Q2. What are its main components?**

Strategy, Concrete Strategy, Context, and Client.

**Q3. How does it replace `if-else` logic?**

It moves each alternative algorithm into its own implementation and delegates execution through a common interface.

**Q4. Can Strategy be changed at runtime?**

Yes. The context can receive a different strategy through a setter or another selection mechanism.

**Q5. How is Strategy implemented in Spring Boot?**

Register multiple implementations as Spring beans and inject them through a common interface, collection, or map.

**Q6. What is the difference between Strategy and State?**

Strategy selects an algorithm; State represents behavior that changes according to an object's internal state.

**Q7. Can Strategy and Factory be used together?**

Yes. A factory can select and instantiate the appropriate strategy.

**Q8. Which SOLID principles does Strategy support?**

Primarily SRP, OCP, and DIP.

---

## Final Summary

The **Strategy Pattern** encapsulates interchangeable algorithms and allows the application to select the required behavior without coupling the context to concrete implementations.

Remember the key example:

- `PaymentStrategy` — common interface.
- `UpiPayment` — one algorithm.
- `CreditCardPayment` — another algorithm.
- `CheckoutService` — context that delegates payment execution.

**Interview takeaway:** Use Strategy when you need interchangeable behavior; use Factory when you need to encapsulate object creation.
