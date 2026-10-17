---
title: Fundamentals and Entry Point
parent: Spring Boot
nav_order: 1
---

# Fundamentals and Entry Point
{: .no_toc }

What Spring Boot adds and how an application starts.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What does @SpringBootApplication do?

Easy
{: .label .label-green }

**Answer**

It is a combination of three annotations:
- `@SpringBootConfiguration` (a `@Configuration` class)
- `@EnableAutoConfiguration`
- `@ComponentScan` (scans the package of the main class and its sub-packages)

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

---

## What happens when SpringApplication.run() is called?

Medium
{: .label .label-yellow }

**Answer**

1. Creates the `ApplicationContext` and determines the application type (servlet, reactive, none).
2. Loads the environment and properties.
3. Registers beans from component scanning and auto-configuration.
4. Starts the embedded server (Tomcat by default).
5. Runs `CommandLineRunner` and `ApplicationRunner` beans.

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
