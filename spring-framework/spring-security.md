---
title: Spring Security
parent: Spring Framework
nav_order: 7
---

# Spring Security
{: .no_toc }

Authentication, authorization and JWT.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How does JWT authentication work with Spring Security?

Hard
{: .label .label-red }

**Answer**

1. The client sends credentials to a login endpoint.
2. The server authenticates (`AuthenticationManager`) and returns a signed JWT.
3. The client sends `Authorization: Bearer <token>` on later calls.
4. A custom filter in the `SecurityFilterChain` validates signature and expiry, then sets the `Authentication` in the `SecurityContext`.
5. Authorization rules (`hasRole`, `@PreAuthorize`) are checked.

**Key points**
- Authentication = who you are; authorization = what you may do.
- Keep access tokens short-lived and use refresh tokens.
- Never store secrets in the JWT payload; it is only encoded, not encrypted.

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
