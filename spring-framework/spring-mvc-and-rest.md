---
title: Spring MVC and REST
parent: Spring Framework
nav_order: 5
---

# Spring MVC and REST
{: .no_toc }

Request flow, filters and interceptors.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Explain the request flow in Spring MVC.

Medium
{: .label .label-yellow }

**Answer**

1. Request reaches `DispatcherServlet` (front controller).
2. `HandlerMapping` finds the controller method.
3. `HandlerAdapter` invokes it, with argument resolvers binding `@RequestBody`, `@PathVariable`, etc.
4. For REST, an `HttpMessageConverter` (Jackson) writes the response body; for MVC views, a `ViewResolver` picks the view.

---

## What is the difference between a Filter and an Interceptor?

Medium
{: .label .label-yellow }

**Answer**

A **Filter** is a Servlet API component that runs before the `DispatcherServlet` and wraps the whole request. A **HandlerInterceptor** is a Spring MVC component with `preHandle`, `postHandle` and `afterCompletion` that knows about the handler method. Use filters for security, CORS and logging of raw requests; interceptors for handler-aware logic.

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
