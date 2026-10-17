---
title: Aspect-Oriented Programming (AOP)
parent: Spring Framework
nav_order: 4
---

# Aspect-Oriented Programming (AOP)
{: .no_toc }

Cross-cutting concerns, advice types and proxies.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is AOP and what are its key terms?

Medium
{: .label .label-yellow }

**Answer**

AOP separates cross-cutting concerns (logging, security, transactions, metrics) from business logic.

| Term | Meaning |
|:--|:--|
| Aspect | Class holding cross-cutting logic |
| Join point | A point in execution (in Spring AOP: a method call) |
| Pointcut | Expression selecting join points |
| Advice | Action: `@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`, `@Around` |
| Weaving | Linking aspects to targets (Spring does it at runtime with proxies) |

```java
@Aspect
@Component
class TimingAspect {
    @Around("execution(* com.example.service..*(..))")
    Object time(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.nanoTime();
        try { return pjp.proceed(); }
        finally { System.out.println(pjp.getSignature() + " took " + (System.nanoTime() - start)); }
    }
}
```

---

## JDK dynamic proxy vs CGLIB, and what is the self-invocation problem?

Hard
{: .label .label-red }

**Answer**

JDK proxies work on interfaces; CGLIB creates a subclass (Spring Boot defaults to CGLIB). Because advice lives in the proxy, a call from one method to another **inside the same class** bypasses the proxy, so `@Transactional`, `@Async` or `@Cacheable` on the inner method is ignored. Fix by moving the method to another bean.

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
