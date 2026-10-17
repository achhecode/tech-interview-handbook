---
title: Coding Principles
parent: Design Patterns and LLD
nav_order: 1
---

# Coding Principles
{: .no_toc }

Coding principles for writing maintainable, flexible, and extensible software.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

  1. TOC
  {:toc}
</details>

---

## Explain the SOLID principles with examples.

Medium
{: .label .label-yellow }

**SOLID** is an acronym for five object-oriented design principles that help developers build software that is easier to understand, test, maintain, and extend.

- **S — Single Responsibility Principle (SRP):** A class should have one primary responsibility and one main reason to change. For example, a `UserService` should handle user-related business logic, while a separate `EmailService` handles sending emails.

- **O — Open/Closed Principle (OCP):** Software components should allow new functionality to be added without repeatedly modifying existing, working code. For example, a `PaymentProcessor` can work with a `PaymentMethod` interface, allowing new payment implementations to be introduced without changing the processor.

- **L — Liskov Substitution Principle (LSP):** An implementation of a base type must be usable wherever that type is expected without breaking the program's intended behavior. For example, every implementation of a `PaymentMethod` interface should honor its contract for processing payments rather than unexpectedly violating its assumptions.

- **I — Interface Segregation Principle (ISP):** Classes should not be forced to depend on methods they do not need. For example, instead of one large `PaymentService` interface containing payment, refund, and reporting operations, separate interfaces can define these capabilities for the classes that require them.

- **D — Dependency Inversion Principle (DIP):** High-level business logic should depend on abstractions rather than directly depending on low-level implementation details. For example, an `OrderService` can depend on a `PaymentGateway` interface instead of being tightly coupled to a specific payment provider. Spring's dependency injection makes this design easier to implement and test.

Quick revision

| Principle | Key idea |
|---|---|
| **S — SRP** | One responsibility per class |
| **O — OCP** | Extend behavior without unnecessary modification |
| **L — LSP** | Subtypes must honor the base type's contract |
| **I — ISP** | Prefer small, focused interfaces |
| **D — DIP** | Depend on abstractions, not concrete implementations |

**Interview tip:** SOLID principles are design guidelines, not rigid rules. Apply them where they reduce coupling, improve cohesion, and make the code easier to maintain.

---

## What are Cohesion and Coupling?

Medium
{: .label .label-yellow }

**Cohesion** and **Coupling** are two important concepts in software design that help us evaluate how well a system is organized and how much its components depend on one another.

- **Cohesion:** It describes how closely the responsibilities within a class or module are related. **High cohesion** means a class focuses on a clear, well-defined responsibility. For example, an `EmailService` that handles email-related operations has high cohesion. Low cohesion occurs when a class combines unrelated tasks, such as sending emails, processing payments, and generating reports.

- **Coupling:** It describes the degree of dependency between different classes or modules. **Low coupling** means components can be changed or replaced with minimal impact on other components. For example, an `OrderService` that depends on a `PaymentGateway` interface rather than a specific payment provider is loosely coupled. High coupling occurs when a class directly depends on the internal implementation of another class, making changes more difficult.

Quick comparison

| Cohesion | Coupling |
|---|---|
| Measures how closely related the responsibilities within a module are. | Measures how strongly one module depends on another. |
| High cohesion is generally desirable. | Low coupling is generally desirable. |
| Helps keep classes focused on specific responsibilities. | Helps minimize the impact of changes across modules. |

**Interview tip:** Good software design aims for **high cohesion and low coupling**. High cohesion keeps each component focused, while low coupling allows components to evolve independently, making the system easier to test, maintain, and extend.


---

