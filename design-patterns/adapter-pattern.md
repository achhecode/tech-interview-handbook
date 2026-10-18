---
title: Adapter
parent: Design Patterns and LLD
nav_order: 5
---

# Adapter
{: .no_toc }

Allows incompatible interfaces to work together by converting one interface into another that the client expects.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

1. TOC
{:toc}
</details>

---

## 1. What is the Adapter Design Pattern?

{: .label .label-green }

**Answer**

The Adapter is a structural design pattern that allows two incompatible interfaces to work together.

It acts as a bridge between a client and an existing class whose interface does not match what the client expects. The adapter converts the client's request into a format that the existing class understands.

**Real-world example:** A travel power adapter allows a plug from one country to fit a socket used in another country.

### Key components

- **Client:** The code that needs a particular interface.
- **Target interface:** The interface expected by the client.
- **Adaptee:** The existing class with an incompatible interface.
- **Adapter:** Implements the target interface and delegates requests to the adaptee.

### Example: Payment gateway adapter

Suppose our application expects every payment gateway to provide a `pay()` method, but an existing third-party gateway provides a `makePayment()` method.

The adapter makes them compatible without modifying the third-party class.

```java
// Target interface expected by our application
interface PaymentProcessor {
    void pay(double amount);
}

// Existing third-party class (Adaptee)
class LegacyPaymentGateway {

    public void makePayment(double amount) {
        System.out.println(
            "Payment processed: ₹" + amount
        );
    }
}

// Adapter
class PaymentAdapter implements PaymentProcessor {

    private final LegacyPaymentGateway gateway;

    public PaymentAdapter(LegacyPaymentGateway gateway) {
        this.gateway = gateway;
    }

    @Override
    public void pay(double amount) {
        gateway.makePayment(amount);
    }
}

// Client
public class Main {

    public static void main(String[] args) {
        LegacyPaymentGateway gateway =
                new LegacyPaymentGateway();

        PaymentProcessor processor =
                new PaymentAdapter(gateway);

        processor.pay(1000);
    }
}
```

**Output**

```text
Payment processed: ₹1000.0
```

**How it works**

1. The client calls `pay(1000)`.
2. `PaymentAdapter` receives the request.
3. The adapter translates the call into `makePayment(1000)`.
4. The existing gateway processes the payment.

The client does not need to know the gateway's original method name.

---

## 2. Why do we need the Adapter Pattern?

{: .label .label-green }

**Answer**

We use the Adapter Pattern when an existing class or third-party library provides functionality we need, but its interface is incompatible with our application's expected interface.

### Without an adapter

The client depends directly on the third-party implementation.

```java
LegacyPaymentGateway gateway =
        new LegacyPaymentGateway();

gateway.makePayment(1000);
```

If we replace the gateway with another provider that exposes a different method, the client code must change.

### With an adapter

The client depends on a common interface.

```java
PaymentProcessor processor =
        new PaymentAdapter(
            new LegacyPaymentGateway()
        );

processor.pay(1000);
```

Now the client uses `PaymentProcessor`, regardless of the gateway's original interface.

**Benefits**

- Integrates legacy systems and third-party libraries.
- Avoids modifying existing classes.
- Reduces coupling between client code and external implementations.
- Makes replacing implementations easier.
- Supports cleaner integration boundaries.

---

## 3. What are the types of Adapter Pattern?

{: .label .label-blue }

**Answer**

The two commonly discussed types are object adapters and class adapters.

### 3.1 Object Adapter

An object adapter uses composition: it holds a reference to the adaptee and delegates calls to it.

```java
class PaymentAdapter implements PaymentProcessor {

    private final LegacyPaymentGateway gateway;

    public PaymentAdapter(LegacyPaymentGateway gateway) {
        this.gateway = gateway;
    }

    @Override
    public void pay(double amount) {
        gateway.makePayment(amount);
    }
}
```

**Characteristics**

- Uses composition.
- Can adapt an existing object.
- Does not require extending the adaptee.
- Is generally more flexible in Java.

### 3.2 Class Adapter

A class adapter uses inheritance to adapt an existing class to the target interface.

```java
class PaymentClassAdapter
        extends LegacyPaymentGateway
        implements PaymentProcessor {

    @Override
    public void pay(double amount) {
        makePayment(amount);
    }
}
```

**Usage**

```java
PaymentProcessor processor =
        new PaymentClassAdapter();

processor.pay(1000);
```

**Characteristics**

- Uses inheritance.
- Extends the adaptee and implements the target interface.
- Is constrained by Java's single class inheritance.
- Works only when the adaptee can be extended appropriately.

**Interview tip:** Object adapters use composition; class adapters use inheritance. Object adapters are generally preferred in Java.

---

## 4. How do you implement the Adapter Pattern in Java?

{: .label .label-green }

**Answer**

Follow these steps:

1. Define the target interface expected by the client.
2. Identify the existing class with an incompatible interface.
3. Create an adapter that implements the target interface.
4. Inject or pass the existing class into the adapter.
5. Translate target-interface calls into adaptee calls.
6. Let the client use the target interface.

### Complete example: Temperature converter

Suppose an application works with Fahrenheit temperatures, but an existing library returns Celsius temperatures.

```java
// Target interface
interface FahrenheitSensor {
    double getTemperatureFahrenheit();
}

// Adaptee
class CelsiusSensor {

    public double getTemperatureCelsius() {
        return 25.0;
    }
}

// Adapter
class TemperatureAdapter implements FahrenheitSensor {

    private final CelsiusSensor sensor;

    public TemperatureAdapter(CelsiusSensor sensor) {
        this.sensor = sensor;
    }

    @Override
    public double getTemperatureFahrenheit() {
        double celsius = sensor.getTemperatureCelsius();

        return (celsius * 9.0 / 5.0) + 32.0;
    }
}

// Client
public class Main {

    public static void main(String[] args) {
        FahrenheitSensor sensor =
                new TemperatureAdapter(
                    new CelsiusSensor()
                );

        System.out.println(
            sensor.getTemperatureFahrenheit()
        );
    }
}
```

**Output**

```text
77.0
```

The adapter converts Celsius to Fahrenheit and exposes the result through the interface the client expects.

---

## 5. What is the difference between Adapter and Decorator?

{: .label .label-yellow }

**Answer**

Both patterns use composition frequently, but their purposes are different.

| Adapter Pattern | Decorator Pattern |
|---|---|
| Converts one interface into another. | Adds responsibilities to an object. |
| Focuses on compatibility. | Focuses on extending behavior. |
| Makes incompatible classes work together. | Wraps an object while preserving its interface. |
| Example: Adapting a legacy payment gateway. | Example: Adding logging or caching around a service. |

**Remember:** Adapter changes the interface presented to the client; Decorator enhances behavior while generally preserving the interface.

---

## 6. What is the difference between Adapter and Facade?

{: .label .label-yellow }

**Answer**

| Adapter Pattern | Facade Pattern |
|---|---|
| Makes an existing interface compatible with the client's expected interface. | Provides a simpler interface to a complex subsystem. |
| Focuses on interface conversion. | Focuses on simplifying usage. |
| Usually wraps an adaptee. | Usually coordinates multiple subsystem components. |
| Example: Converting a legacy API to a common interface. | Example: Providing one method to complete an order workflow. |

**Remember:** Adapter makes interfaces compatible; Facade makes a complex system easier to use.

---

## 7. What is the difference between Adapter and Proxy?

{: .label .label-blue }

**Answer**

| Adapter Pattern | Proxy Pattern |
|---|---|
| Converts an interface. | Controls access to an object. |
| Focuses on compatibility. | Focuses on access management. |
| May translate parameters, results, or method calls. | May add access checks, lazy loading, or remote invocation. |
| Example: Wrapping a legacy payment API. | Example: A proxy that checks authorization before calling a service. |

**Remember:** Adapter changes the interface; Proxy controls access while generally presenting the same interface.

---

## 8. What are the advantages and disadvantages of the Adapter Pattern?

{: .label .label-blue }

**Answer**

### Advantages

- **Reusability:** Uses existing classes without modifying their source code.
- **Loose coupling:** The client depends on the target interface rather than a specific adaptee.
- **Integration:** Connects systems that expose different interfaces.
- **Maintainability:** Helps isolate external API differences.
- **Flexibility:** Supports switching implementations when adapters share the same target interface.

### Disadvantages

- **Additional classes:** Requires extra adapter classes and interfaces.
- **Complexity:** Too many adapters can make the code harder to navigate.
- **Performance overhead:** Translation or conversion may introduce small overhead.
- **Limited compatibility:** Some differences require substantial redesign rather than simple adaptation.

---

## 9. Where is the Adapter Pattern used in real-world applications?

{: .label .label-green }

**Answer**

The Adapter Pattern is useful whenever an application must integrate components that expose different interfaces.

### Common use cases

- **Payment integrations:** Expose a common interface for multiple payment providers.
- **Legacy systems:** Integrate older services without rewriting them.
- **Third-party APIs:** Convert external request and response formats into internal models.
- **Database integrations:** Wrap different data sources behind a common interface when their operations can be normalized.
- **Logging:** Adapt an external logging library to an application's logging interface.
- **Cloud providers:** Provide a common interface for storage services with different SDKs.

### Example: Multiple payment providers

```java
interface PaymentProcessor {
    void pay(double amount);
}

class StripeGateway {
    public void charge(double amount) {
        System.out.println("Stripe: " + amount);
    }
}

class StripeAdapter implements PaymentProcessor {

    private final StripeGateway stripe;

    public StripeAdapter(StripeGateway stripe) {
        this.stripe = stripe;
    }

    @Override
    public void pay(double amount) {
        stripe.charge(amount);
    }
}

class PayPalGateway {
    public void sendPayment(double amount) {
        System.out.println("PayPal: " + amount);
    }
}

class PayPalAdapter implements PaymentProcessor {

    private final PayPalGateway paypal;

    public PayPalAdapter(PayPalGateway paypal) {
        this.paypal = paypal;
    }

    @Override
    public void pay(double amount) {
        paypal.sendPayment(amount);
    }
}
```

**Client code**

```java
public class Main {

    public static void main(String[] args) {
        PaymentProcessor stripe =
                new StripeAdapter(new StripeGateway());

        PaymentProcessor paypal =
                new PayPalAdapter(new PayPalGateway());

        stripe.pay(1000);
        paypal.pay(2000);
    }
}
```

**Output**

```text
Stripe: 1000.0
PayPal: 2000.0
```

The client calls `pay()` for both providers. Each adapter translates the request into the method supported by its gateway.

In production, provider-specific authentication, error handling, currency conversion, and transaction-status mapping would also need to be handled.

---

## 10. How is the Adapter Pattern used in Spring Boot?

{: .label .label-blue }

**Answer**

In Spring Boot, adapters are useful for isolating external APIs, legacy services, and vendor-specific implementations from business logic.

For example, define an application-owned interface:

```java
public interface PaymentProcessor {
    void pay(double amount);
}
```

Implement it using a third-party gateway:

```java
import org.springframework.stereotype.Component;

@Component
public class LegacyPaymentAdapter implements PaymentProcessor {

    private final LegacyPaymentGateway gateway;

    public LegacyPaymentAdapter(LegacyPaymentGateway gateway) {
        this.gateway = gateway;
    }

    @Override
    public void pay(double amount) {
        gateway.makePayment(amount);
    }
}
```

Inject the interface into a service:

```java
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    private final PaymentProcessor paymentProcessor;

    public OrderService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }

    public void checkout(double amount) {
        paymentProcessor.pay(amount);
    }
}
```

**How it works**

1. Spring creates the adapter bean.
2. Spring injects the required gateway dependency.
3. `OrderService` depends only on `PaymentProcessor`.
4. The adapter translates the application's call into the external gateway's API.

If multiple `PaymentProcessor` implementations are registered, configure `@Qualifier` or `@Primary` as appropriate.

**Important:** Spring dependency injection does not itself make a class an Adapter. The pattern is present because the adapter translates one interface into another.

---

## 11. What are the SOLID principles related to the Adapter Pattern?

{: .label .label-blue }

**Answer**

The Adapter Pattern supports several SOLID principles when designed appropriately.

- **Single Responsibility Principle (SRP):** The adapter handles interface translation instead of mixing it into business logic.
- **Open/Closed Principle (OCP):** New providers can often be integrated by adding new adapters without modifying client code.
- **Dependency Inversion Principle (DIP):** The client depends on an application-owned abstraction instead of a concrete external implementation.

These principles are supported by good adapter design, but using the pattern does not automatically guarantee SOLID compliance.

---

## 12. Quick interview revision

{: .label .label-green }

| Question | Short answer |
|---|---|
| What is the Adapter Pattern? | A structural pattern that makes incompatible interfaces work together. |
| Why is it used? | To integrate existing classes or external systems without changing their source code. |
| What are its main components? | Client, target interface, adaptee, and adapter. |
| What are the two common types? | Object adapter and class adapter. |
| Which type uses composition? | Object adapter. |
| Which type uses inheritance? | Class adapter. |
| Which type is generally preferred in Java? | Object adapter. |
| Does Adapter modify the adaptee? | Usually not; it wraps or extends it. |
| Adapter vs Decorator? | Adapter changes the interface; Decorator adds responsibilities. |
| Adapter vs Facade? | Adapter provides compatibility; Facade simplifies a subsystem. |
| Adapter vs Proxy? | Adapter changes the interface; Proxy controls access. |
| Is Adapter a creational pattern? | No, it is a structural design pattern. |
| Is dependency injection the same as Adapter? | No. Dependency injection supplies dependencies; an adapter translates interfaces. |

---

## 13. Final recommendation

Remember this simple rule:

**Use the Adapter Pattern when an existing class provides the functionality you need, but its interface does not match what your application expects.**

For Java and Spring Boot projects, prefer an object adapter using composition. Define an application-owned interface, implement an adapter for each external provider, and keep provider-specific details out of your business logic.