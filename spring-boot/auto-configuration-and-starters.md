---
title: Auto-Configuration and Starters
parent: Spring Boot
nav_order: 4
---

# Auto-Configuration and Starters
{: .no_toc }

How Boot configures beans automatically.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How does Spring Boot auto-configuration work?

Hard
{: .label .label-red }

**Answer**

Boot reads auto-configuration classes listed in `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`. Each class is guarded by conditions such as `@ConditionalOnClass`, `@ConditionalOnMissingBean` and `@ConditionalOnProperty`, so beans are only created when the right libraries are on the classpath and you have not defined your own.

Starters are curated dependency bundles (for example `spring-boot-starter-web`) that pull in the libraries needed for a feature.

{: .tip }
Run with `--debug` to print the conditions evaluation report and see why a configuration did or did not apply.

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
