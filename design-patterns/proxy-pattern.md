---
title: Proxy
parent: Design Patterns and LLD
nav_order: 6
---

# Proxy Pattern
{: .no_toc }

Provides a substitute object that controls access to the original object.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the Proxy Design Pattern?

The **Proxy Pattern** is a structural design pattern that provides a substitute or representative object for another object to control access to it.

The proxy implements the same interface as the real object and delegates requests to it when appropriate.

**Real-world example:** A bank ATM acts as a proxy for your bank account. You interact with the ATM instead of directly accessing the bank's internal systems. The ATM controls access through authentication and transaction validation.

### Key components

- **Subject:** Interface shared by the proxy and the real object.
- **Real Subject:** The actual object that performs the business operation.
- **Proxy:** Controls access to the real subject and may add authentication, caching, logging, or other behavior.
- **Client:** Interacts with the subject interface without needing to know whether it is using the proxy or the real object.

---

## How do you implement the Proxy Pattern in Java?

Consider a document system where loading a document from remote storage is expensive. We want to load it only when the client actually requests it.

### Step 1: Create the Subject interface

```java
public interface Document {
    void display();
}
```

Both the real object and the proxy implement this interface.

### Step 2: Create the Real Subject

```java
public class RealDocument implements Document {

    private final String fileName;

    public RealDocument(String fileName) {
        this.fileName = fileName;
        loadFromDisk();
    }

    private void loadFromDisk() {
        System.out.println("Loading document: " + fileName);
    }

    @Override
    public void display() {
        System.out.println("Displaying document: " + fileName);
    }
}
```

The constructor simulates an expensive operation, such as retrieving a document from remote storage.

### Step 3: Create the Proxy

```java
public class DocumentProxy implements Document {

    private final String fileName;
    private RealDocument realDocument;

    public DocumentProxy(String fileName) {
        this.fileName = fileName;
    }

    @Override
    public void display() {
        if (realDocument == null) {
            realDocument = new RealDocument(fileName);
        }

        realDocument.display();
    }
}
```

The proxy creates the real document only when `display()` is called for the first time.

### Step 4: Use the Proxy

```java
public class Main {
    public static void main(String[] args) {
        Document document = new DocumentProxy("report.pdf");

        System.out.println("Proxy created.");
        document.display();

        System.out.println("Displaying again.");
        document.display();
    }
}
```

**Output**

```text
Proxy created.
Loading document: report.pdf
Displaying document: report.pdf
Displaying again.
Displaying document: report.pdf
```

**Important:** The document is loaded only once. Subsequent calls reuse the existing object.

This is called a **Virtual Proxy**, because it delays creating an expensive object until it is needed.

---

## What are the types of Proxy?

### 1. Virtual Proxy

Delays the creation or loading of an expensive object until it is required.

**Example:** Loading large images or documents only when the user opens them.

```java
if (realDocument == null) {
    realDocument = new RealDocument(fileName);
}
```

### 2. Protection Proxy

Controls access based on permissions or authentication.

**Example:** Allowing administrators to delete records while read-only users can only view them.

```java
public class AdminProxy implements Document {

    private final Document document;
    private final boolean isAdmin;

    public AdminProxy(Document document, boolean isAdmin) {
        this.document = document;
        this.isAdmin = isAdmin;
    }

    @Override
    public void display() {
        if (!isAdmin) {
            throw new SecurityException("Access denied");
        }

        document.display();
    }
}
```

This example protects the `display()` operation for illustration. In a real application, authorization should be applied to the actual protected operation and enforced on the server side.

### 3. Remote Proxy

Represents an object that exists in another process or on another machine.

**Example:** A client-side object that communicates with a remote service over HTTP or RPC.

The client invokes a method on the proxy, which handles communication with the remote system.

### 4. Caching Proxy

Caches results to avoid repeatedly executing expensive operations.

**Example:** Caching the response of a remote API or database query.

```java
public class CachedService implements DataService {

    private final DataService realService;
    private String cachedData;

    public CachedService(DataService realService) {
        this.realService = realService;
    }

    @Override
    public String fetchData() {
        if (cachedData == null) {
            cachedData = realService.fetchData();
        }

        return cachedData;
    }
}
```

Assume `DataService` declares `String fetchData()`.

This simplified example caches one result in memory. Production caches need appropriate expiration, invalidation, concurrency, and error-handling policies.

### 5. Logging Proxy

Adds logging around calls to the real object.

**Example:** Recording when a service operation starts, completes, or fails without modifying its business logic.

### 6. Smart Reference Proxy

Performs additional actions whenever the real object is accessed.

**Example:** Reference counting, resource tracking, or checking whether a resource is still available.

---

## What is the difference between Proxy and Decorator?

Both patterns wrap an object and implement the same interface, but their primary intentions differ.

| Feature | Proxy | Decorator |
|---|---|---|
| Main purpose | Control access to an object | Add responsibilities or behavior |
| Focus | Access, lifecycle, communication, or caching | Extending functionality |
| Real object | Often controlled or created by the proxy | Usually supplied to the decorator |
| Typical use case | Lazy loading, authorization, remote access | Adding compression, notifications, or formatting |
| Client awareness | Ideally uses the same interface transparently | Can explicitly compose multiple decorators |

**Interview point:** Their structures can look almost identical. The primary difference is the intent, not simply the code structure.

---

## What is the difference between Proxy and Adapter?

| Feature | Proxy | Adapter |
|---|---|---|
| Purpose | Controls access to an object | Makes incompatible interfaces compatible |
| Interface | Usually the same as the real object | Converts one interface into another |
| Example | Lazy-loading a document | Adapting a third-party payment API |
| Main concern | Access control or object interaction | Interface compatibility |

---

## What is the difference between Proxy and Facade?

| Feature | Proxy | Facade |
|---|---|---|
| Purpose | Controls access to an underlying object | Simplifies access to a subsystem |
| Relationship | Commonly represents one subject | Commonly coordinates multiple subsystem components |
| Interface | Typically matches the real subject's interface | Provides a simplified interface |
| Example | Securing access to a document | Simplifying an order-processing workflow |

---

## How is Proxy used in Spring Boot?

Spring frequently uses proxies to provide framework behavior around application methods.

Common examples include:

- `@Transactional` — transaction management.
- `@Cacheable` — caching method results.
- `@Async` — asynchronous method execution.
- Spring AOP — logging, auditing, and cross-cutting concerns.
- Spring Security method security — authorization interceptors and proxies in supported configurations.

### Example: Transactional Proxy

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class EmployeeService {

    @Transactional
    public void saveEmployee() {
        // Perform database operations.
        // Spring manages the transaction around this method.
    }
}
```

When configured correctly, Spring can create a proxy around `EmployeeService`. A call through that proxy allows Spring's transaction interceptor to begin, commit, or roll back a transaction as appropriate.

Conceptually:

```text
Client
   |
   v
Spring Proxy
   |
   +--> Begin transaction
   |
   v
EmployeeService.saveEmployee()
   |
   v
Database
   |
   v
Commit / Rollback
```

The proxy provides transaction behavior without requiring the service method to manage transaction boundaries manually.

### JDK Dynamic Proxy vs. CGLIB Proxy

Spring can use different proxy mechanisms.

| Feature | JDK Dynamic Proxy | CGLIB Proxy |
|---|---|---|
| Based on | Java interfaces | Subclassing a class |
| Requirement | Target exposes an interface | Target class can be subclassed |
| Limitation | Proxies interface methods | Cannot override final methods; cannot subclass final classes |
| Typical use | Interface-based services | Class-based proxying |

The exact proxy mechanism depends on framework configuration and the type of proxy being created.

### Important Spring Proxy Limitation: Self-Invocation

Consider this example:

```java
@Service
public class PaymentService {

    public void processPayment() {
        savePayment();
    }

    @Transactional
    public void savePayment() {
        // Save payment to the database.
    }
}
```

If `processPayment()` calls `savePayment()` directly on the same object, the call does not pass through the Spring proxy. Therefore, the `@Transactional` interceptor may not run for `savePayment()` as expected.

**Recommended solution:** Move the transactional method into another Spring-managed bean and invoke it through that bean's injected reference. This allows the call to pass through the proxy.

Also remember that transaction behavior depends on configuration, transaction-manager setup, and the applicable rollback rules.

---

## What are the advantages of the Proxy Pattern?

1. **Access control:** Restricts access to sensitive operations.
2. **Lazy initialization:** Delays expensive object creation.
3. **Performance:** Can cache frequently requested results.
4. **Remote communication:** Hides communication details from clients.
5. **Separation of concerns:** Keeps cross-cutting logic outside the core business object.
6. **Transparent interaction:** Allows clients to use a common interface.

## What are the disadvantages of the Proxy Pattern?

1. Adds extra classes or indirection.
2. May increase latency, especially for remote proxies.
3. Caching can introduce stale-data or invalidation problems.
4. Debugging becomes more complicated because method calls pass through additional layers.
5. Proxy behavior may be bypassed by direct references, self-invocation, or unsupported method types.

---

## When should you use the Proxy Pattern?

Use Proxy when you need to:

- Delay creating an expensive object.
- Enforce access restrictions before executing an operation.
- Cache results from expensive service calls.
- Represent a remote object behind a local interface.
- Add logging, auditing, or other cross-cutting behavior.
- Integrate with framework features such as Spring transactions or caching.

Avoid adding a proxy when the underlying operation is simple and no access-control, lifecycle, performance, or cross-cutting requirement justifies the extra indirection.

---

## How does Proxy relate to SOLID principles?

- **Single Responsibility Principle (SRP):** The proxy can handle access control or caching separately from the real object's business logic.
- **Open/Closed Principle (OCP):** New behavior can be added through a proxy without modifying the real subject.
- **Dependency Inversion Principle (DIP):** The client can depend on the shared subject interface instead of a concrete implementation.

---

## Proxy Pattern — Interview Revision

**Q1. What is the Proxy Pattern?**

It is a structural design pattern that provides a substitute object to control access to a real object.

**Q2. What are the main components?**

Subject, Real Subject, Proxy, and Client.

**Q3. What is a Virtual Proxy?**

A proxy that delays the creation or loading of an expensive object until it is needed.

**Q4. What is a Protection Proxy?**

A proxy that checks whether a caller is authorized to access an operation.

**Q5. How does Spring use the Proxy Pattern?**

Spring uses proxies for features such as declarative transactions, caching, asynchronous execution, and AOP-based interception.

**Q6. Why might `@Transactional` not work during self-invocation?**

Because the internal call bypasses the Spring proxy, so the transaction interceptor may not execute.

**Q7. What is the difference between Proxy and Decorator?**

Proxy primarily controls access to an object, while Decorator primarily adds responsibilities to it. Their implementations can have similar structures.

**Q8. What are the two common proxy mechanisms in Spring?**

JDK dynamic proxies and CGLIB-based proxies.

**Q9. Can a proxy improve performance?**

Yes. A virtual proxy can delay expensive initialization, and a caching proxy can avoid repeated expensive operations. However, proxies can also introduce overhead.

**Q10. Is Spring's `@Transactional` an example of the Proxy Pattern?**

Yes. In common proxy-based Spring configurations, a proxy intercepts eligible method calls and applies transaction management.

---

## Final Summary

The **Proxy Pattern** controls access to an object through a substitute that shares the subject's interface.

Remember these examples:

- **Virtual Proxy:** Lazy loading.
- **Protection Proxy:** Authorization.
- **Remote Proxy:** Remote service communication.
- **Caching Proxy:** Reusing stored results.
- **Spring Proxy:** Transactions, caching, and AOP.

The key interview distinction is: **Proxy controls access; Decorator adds responsibilities; Adapter converts interfaces; Facade simplifies a subsystem.**
