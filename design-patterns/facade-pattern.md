---
title: Facade
parent: Design Patterns and LLD
nav_order: 10
---

# Facade Pattern
{: .no_toc }

Provides a simplified interface to a complex subsystem.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the Facade Design Pattern?

The **Facade Pattern** is a structural design pattern that provides a simple, unified interface to a complex subsystem.

Instead of requiring clients to interact with multiple classes directly, the facade coordinates those classes and exposes a simpler operation.

**Real-world example:** When you place an online order, you do not manually coordinate inventory, payment, shipping, and notifications. An order-processing service can provide one simple operation that coordinates these subsystems.

### Key components

- **Facade:** Provides a simplified interface and coordinates subsystem operations.
- **Subsystems:** Classes that perform the actual work.
- **Client:** Uses the facade instead of coordinating subsystem classes directly.

---

## How do you implement the Facade Pattern in Java?

Consider an e-commerce application where placing an order involves inventory validation, payment processing, and shipping.

### Step 1: Create the subsystem classes

**Inventory Service**

```java
public class InventoryService {

    public boolean checkAvailability(String productId) {
        System.out.println(
            "Checking inventory for " + productId
        );

        return true;
    }
}
```

**Payment Service**

```java
public class PaymentService {

    public void processPayment(double amount) {
        System.out.println(
            "Processing payment of ₹" + amount
        );
    }
}
```

**Shipping Service**

```java
public class ShippingService {

    public void arrangeShipping(String productId) {
        System.out.println(
            "Shipping arranged for " + productId
        );
    }
}
```

Each subsystem is responsible for one specific task.

### Step 2: Create the Facade

```java
public class OrderFacade {

    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    private final ShippingService shippingService;

    public OrderFacade(
            InventoryService inventoryService,
            PaymentService paymentService,
            ShippingService shippingService) {

        this.inventoryService = inventoryService;
        this.paymentService = paymentService;
        this.shippingService = shippingService;
    }

    public void placeOrder(
            String productId, double amount) {

        if (!inventoryService.checkAvailability(productId)) {
            throw new IllegalStateException(
                "Product is unavailable"
            );
        }

        paymentService.processPayment(amount);
        shippingService.arrangeShipping(productId);

        System.out.println("Order placed successfully");
    }
}
```

The facade coordinates the subsystems behind one method: `placeOrder()`.

### Step 3: Use the Facade

```java
public class Main {

    public static void main(String[] args) {

        InventoryService inventory =
            new InventoryService();

        PaymentService payment =
            new PaymentService();

        ShippingService shipping =
            new ShippingService();

        OrderFacade orderFacade =
            new OrderFacade(inventory, payment, shipping);

        orderFacade.placeOrder("PROD-101", 2500);
    }
}
```

**Output**

```text
Checking inventory for PROD-101
Processing payment of ₹2500.0
Shipping arranged for PROD-101
Order placed successfully
```

### How does it work?

1. The client calls `OrderFacade.placeOrder()`.
2. The facade checks product availability.
3. It coordinates payment processing.
4. It arranges shipping.
5. It reports that the order was placed.

The client does not need to know the sequence of subsystem calls.

**Note:** This is a simplified example. A production order workflow must handle payment failures, refunds or compensation, shipping failures, persistence, and idempotency. The facade itself does not automatically provide transaction management across all these operations.

---

## What problem does the Facade Pattern solve?

Without a facade, a client might need to coordinate several subsystems directly.

```java
InventoryService inventory = new InventoryService();
PaymentService payment = new PaymentService();
ShippingService shipping = new ShippingService();

if (inventory.checkAvailability("PROD-101")) {
    payment.processPayment(2500);
    shipping.arrangeShipping("PROD-101");
}
```

If many clients repeat this sequence, changes to the workflow may require updates in multiple places.

With a facade:

```java
orderFacade.placeOrder("PROD-101", 2500);
```

The workflow is centralized behind a simple interface.

---

## What are the advantages of the Facade Pattern?

1. **Simplifies usage:** Clients interact with a small, convenient interface.
2. **Reduces coupling:** Clients depend less on individual subsystem classes.
3. **Centralizes coordination:** Common workflows can be implemented in one place.
4. **Improves maintainability:** Subsystem interactions can change without requiring every client to change.
5. **Supports layering:** A facade can provide an entry point to a lower-level subsystem.

## What are the disadvantages of the Facade Pattern?

1. A poorly designed facade can become a large class with too many responsibilities.
2. It can hide useful subsystem capabilities if its interface is too restrictive.
3. It does not automatically solve distributed transactions, retries, or failure recovery.
4. Clients may become overly dependent on a facade that tries to handle unrelated workflows.

---

## Facade Pattern with Spring Boot

In Spring Boot, a service can act as a facade by coordinating multiple services and exposing a simpler business operation.

### Example: Customer Dashboard

Suppose a dashboard requires customer details, order history, and loyalty points.

**Customer Service**

```java
import org.springframework.stereotype.Service;

@Service
public class CustomerService {

    public String getCustomer(Long customerId) {
        return "Customer-" + customerId;
    }
}
```

**Order Service**

```java
import java.util.List;
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    public List<String> getOrders(Long customerId) {
        return List.of("Order-101", "Order-102");
    }
}
```

**Loyalty Service**

```java
import org.springframework.stereotype.Service;

@Service
public class LoyaltyService {

    public int getPoints(Long customerId) {
        return 500;
    }
}
```

### Create the Facade Service

```java
import java.util.List;

import org.springframework.stereotype.Service;

@Service
public class CustomerDashboardFacade {

    private final CustomerService customerService;
    private final OrderService orderService;
    private final LoyaltyService loyaltyService;

    public CustomerDashboardFacade(
            CustomerService customerService,
            OrderService orderService,
            LoyaltyService loyaltyService) {

        this.customerService = customerService;
        this.orderService = orderService;
        this.loyaltyService = loyaltyService;
    }

    public CustomerDashboard getDashboard(Long customerId) {

        String customer =
            customerService.getCustomer(customerId);

        List<String> orders =
            orderService.getOrders(customerId);

        int points =
            loyaltyService.getPoints(customerId);

        return new CustomerDashboard(customer, orders, points);
    }
}
```

Define the response DTO:

```java
import java.util.List;

public record CustomerDashboard(
    String customer,
    List<String> orders,
    int loyaltyPoints
) {
}
```

### Expose an API Endpoint

```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/customers")
public class CustomerController {

    private final CustomerDashboardFacade dashboardFacade;

    public CustomerController(
            CustomerDashboardFacade dashboardFacade) {
        this.dashboardFacade = dashboardFacade;
    }

    @GetMapping("/{customerId}/dashboard")
    public CustomerDashboard getDashboard(
            @PathVariable Long customerId) {

        return dashboardFacade.getDashboard(customerId);
    }
}
```

**Request**

```http
GET /customers/101/dashboard
```

**Example response**

```json
{
  "customer": "Customer-101",
  "orders": ["Order-101", "Order-102"],
  "loyaltyPoints": 500
}
```

The controller invokes one facade method rather than coordinating three services itself.

**Interview point:** A facade is not necessarily a dedicated framework class. It is a design role that any appropriately designed class can fulfill.

---

## Facade vs. Adapter

| Feature | Facade | Adapter |
|---|---|---|
| Purpose | Simplifies access to a subsystem | Converts an incompatible interface |
| Interface | Introduces a convenient interface | Adapts one interface to another |
| Subsystems | Often coordinates multiple classes | Often wraps one incompatible component |
| Example | Order-processing facade | Adapter for a third-party payment API |

**Remember:** Facade simplifies; Adapter translates.

---

## Facade vs. Proxy

| Feature | Facade | Proxy |
|---|---|---|
| Purpose | Simplifies subsystem interaction | Controls access to an object |
| Main focus | Convenience and coordination | Access, lifecycle, or communication |
| Interface | Usually a simplified interface | Usually matches the real subject's interface |
| Example | Customer dashboard service | Lazy-loading document proxy |

---

## Facade vs. Mediator

Both patterns coordinate interactions, but they have different intentions.

| Feature | Facade | Mediator |
|---|---|---|
| Purpose | Simplifies access to a subsystem | Centralizes communication between peer objects |
| Interaction | Client invokes facade methods | Colleagues communicate through the mediator |
| Main focus | A convenient entry point | Reducing direct dependencies among interacting objects |
| Example | Order-processing facade | Chat-room mediator coordinating participants |

---

## Facade vs. Service Layer

A **Service Layer** is an architectural layer that contains application or business operations. A facade is a design pattern that provides a simplified interface to a subsystem.

A Spring service may serve both purposes when it exposes a convenient operation and coordinates several other services.

They are related concepts, but they are not interchangeable terms.

---

## When should you use the Facade Pattern?

Use Facade when:

- A client needs to interact with several subsystem classes.
- The subsystem is complicated or has a lengthy workflow.
- Multiple clients repeat the same coordination logic.
- You want a clean entry point for a business operation.
- You need to hide low-level implementation details from consumers.

Common examples include order processing, customer dashboards, payment workflows, legacy-system integration, and service orchestration within an application.

Avoid creating a facade that merely forwards every method without simplifying or organizing the underlying interactions.

---

## How does Facade relate to SOLID principles?

- **Single Responsibility Principle (SRP):** A well-designed facade centralizes one related workflow, although an oversized facade can violate SRP.
- **Dependency Inversion Principle (DIP):** A facade can depend on subsystem abstractions instead of concrete implementations.
- **Open/Closed Principle (OCP):** Clients can continue using the facade while internal subsystem coordination evolves, provided its public contract remains stable.

---

## Facade Pattern — Interview Revision

**Q1. What is the Facade Pattern?**

A structural design pattern that provides a simplified interface to a complex subsystem.

**Q2. What are its main components?**

Facade, Subsystems, and Client.

**Q3. What problem does it solve?**

It reduces the complexity and coupling involved when clients must coordinate multiple subsystem classes.

**Q4. Is Facade the same as a wrapper?**

A facade is commonly implemented as a wrapper around subsystem functionality, but its key intention is to simplify access and coordinate operations.

**Q5. How is Facade used in Spring Boot?**

A service can act as a facade by coordinating multiple services and presenting one business operation to a controller or another client.

**Q6. Does Facade automatically handle transactions?**

No. Transaction management must be implemented separately where needed.

**Q7. What is the difference between Facade and Adapter?**

Facade simplifies a subsystem's interface; Adapter makes incompatible interfaces work together.

**Q8. What is the difference between Facade and Mediator?**

Facade simplifies access to a subsystem, while Mediator coordinates communication among peer objects.

**Q9. Can a facade call multiple microservices?**

Yes, an application-level facade can coordinate calls to multiple services. However, network failures, timeouts, retries, and partial success must be handled explicitly.

**Q10. Can a facade become an anti-pattern?**

Yes. It can become a God Object if it accumulates unrelated responsibilities and excessive business logic.

---

## Final Summary

The **Facade Pattern** provides one simple entry point to a complex subsystem and hides the details of coordinating its components.

Remember the example:

- `InventoryService` — checks product availability.
- `PaymentService` — processes payment.
- `ShippingService` — arranges shipping.
- `OrderFacade` — coordinates the order workflow.

**Interview takeaway:** Use Facade to simplify subsystem interaction, Adapter to translate interfaces, and Proxy to control access to an object.
