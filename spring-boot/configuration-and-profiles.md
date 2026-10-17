---
title: Configuration and Profiles
parent: Spring Boot
nav_order: 5
---

# Configuration and Profiles
{: .no_toc }

Properties, profiles and type-safe configuration.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## @Value vs @ConfigurationProperties?

Medium
{: .label .label-yellow }

**Answer**

`@Value` injects a single property and supports SpEL. `@ConfigurationProperties` binds a group of properties to a typed class, with relaxed binding and validation, and is better for structured configuration.

```java
@ConfigurationProperties(prefix = "app.mail")
public record MailProps(String host, int port) {}
```

---

## How do profiles work, and what is the property precedence?

Medium
{: .label .label-yellow }

**Answer**

Activate with `spring.profiles.active=dev` and load `application-dev.yml` on top of `application.yml`. In general, command-line arguments override environment variables, which override profile-specific files, which override the default `application.properties` or `.yml`.

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
