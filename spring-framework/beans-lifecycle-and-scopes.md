---
title: Beans, Lifecycle and Scopes
parent: Spring Framework
nav_order: 2
---

# Beans, Lifecycle and Scopes
{: .no_toc }

Bean creation, lifecycle callbacks, cyclic dependencies and scopes.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Explain the Spring bean lifecycle.

Hard
{: .label .label-red }

**Answer**

1. Instantiate the bean.
2. Populate properties (dependency injection).
3. Aware callbacks (`BeanNameAware`, `ApplicationContextAware`).
4. `BeanPostProcessor.postProcessBeforeInitialization`.
5. `@PostConstruct`, then `InitializingBean.afterPropertiesSet()`, then custom `init-method`.
6. `BeanPostProcessor.postProcessAfterInitialization` (AOP proxies are created here).
7. Bean is ready for use.
8. On shutdown: `@PreDestroy`, `DisposableBean.destroy()`, custom `destroy-method`.

---

## What is a cyclic dependency and how do you fix it?

Hard
{: .label .label-red }

**Answer**

Bean A needs B and B needs A.

- **Constructor injection:** fails with `BeanCurrentlyInCreationException`.
- **Setter or field injection (singletons):** can work through early singleton references.
- Spring Boot 2.6+ prohibits circular references by default (`spring.main.allow-circular-references=false`).

**Fixes:** redesign by extracting the shared logic into a third bean, use events, or as a last resort `@Lazy` on one injection point.

---

## What are the bean scopes in Spring?

Medium
{: .label .label-yellow }

**Answer**

`singleton` (default), `prototype`, and web scopes `request`, `session`, `application`, `websocket`.

{: .trap }
Injecting a prototype bean into a singleton gives you a single instance for the singleton's lifetime. Use `ObjectProvider` or a scoped proxy to get a fresh instance.

---

<!--
ADD NEW QUESTIONS ABOVE THIS COMMENT. Copy the block below, then replace the text.
Levels: Easy (label-green), Medium (label-yellow), Hard (label-red).

## Your question here?

Medium
{: .label .label-yellow }

**Answer**

Write the answer here. Use tables, lists and ```java code blocks.

**Key points**
- Point one

**Follow-up questions**
- Follow-up one

Separate questions with a horizontal rule (three dashes on their own line).
-->
