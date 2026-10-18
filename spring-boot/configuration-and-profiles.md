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

# What Is a Spring Boot Profile?

## 1. Definition

A **Spring Boot profile** allows us to configure an application differently for different environments, such as development, testing, staging, and production, without changing the application code.

Profiles help us manage environment-specific properties, database connections, logging levels, and even which beans should be created.

For example:
- **dev:** Uses a local development database.
- **test:** Uses a test database or an in-memory H2 database.
- **prod:** Uses a production database and production-specific settings.

## 2. How to configure profiles

Spring Boot supports profile-specific configuration files.

### `application.properties`

Common configuration shared across profiles:

```properties
spring.application.name=employee-service
server.port=8080
```

### `application-dev.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/employee_dev
spring.datasource.username=dev_user
logging.level.root=DEBUG
```

### `application-prod.properties`

```properties
spring.datasource.url=jdbc:mysql://prod-db:3306/employee
spring.datasource.username=prod_user
logging.level.root=INFO
```

When the `dev` profile is active, Spring Boot loads `application-dev.properties`. When `prod` is active, it loads `application-prod.properties`.

Keep passwords and other sensitive credentials in environment variables or a secrets manager rather than committing them to configuration files.

## 3. How to activate a profile

There are several ways to activate a Spring Boot profile.

### Method 1: In `application.properties`

```properties
spring.profiles.active=dev
```

This activates the `dev` profile by default. Avoid hardcoding a development profile as the default in a deployment configuration where the environment should control activation.

### Method 2: Using a command-line argument

```bash
java -jar employee-service.jar --spring.profiles.active=prod
```

Command-line properties generally take precedence over properties loaded from configuration files.

### Method 3: Using an environment variable

```bash
SPRING_PROFILES_ACTIVE=prod java -jar employee-service.jar
```

This is useful in Docker containers, Kubernetes deployments, and CI/CD pipelines.

### Method 4: Using Maven or an IDE

You can pass the active profile as a Spring Boot application argument in your IDE's run configuration. For example:

```text
--spring.profiles.active=dev
```

Note that Maven build profiles and Spring Boot runtime profiles are different concepts.

## 4. Using `@Profile` to create environment-specific beans

The `@Profile` annotation allows us to register a bean only when a particular profile is active.

For example, imagine that we have different implementations of a notification service for development and production.

```java
public interface NotificationService {
    void sendNotification(String message);
}
```

### Development implementation

```java
import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Service;

@Service
@Profile("dev")
public class DevNotificationService
        implements NotificationService {

    @Override
    public void sendNotification(String message) {
        System.out.println("DEV notification: " + message);
    }
}
```

### Production implementation

```java
import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Service;

@Service
@Profile("prod")
public class ProdNotificationService
        implements NotificationService {

    @Override
    public void sendNotification(String message) {
        // Integrate with the production notification provider.
        System.out.println("Sending production notification");
    }
}
```

When the `dev` profile is active, Spring registers `DevNotificationService`. When `prod` is active, Spring registers `ProdNotificationService`.

If neither profile is active, neither of these beans is registered. If the application requires a `NotificationService`, startup may fail unless another matching bean is available.

## 5. How to activate multiple profiles

You can activate multiple profiles at once.

```properties
spring.profiles.active=dev,debug
```

Or from the command line:

```bash
java -jar employee-service.jar --spring.profiles.active=dev,debug
```

This is useful when you want a main environment profile alongside an additional configuration profile.

## 6. `application.properties` vs profile-specific properties

| Feature | `application.properties` | `application-dev.properties` |
|---|---|---|
| Purpose | Shared/default configuration | Development-specific configuration |
| Loaded | When applicable, regardless of active profile | When the `dev` profile is active |
| Example | Application name | Development database URL |
| Overrides | Provides defaults | Profile-specific values can override shared values |

Profile-specific properties override the corresponding shared properties when both define the same setting and the profile-specific configuration has higher precedence.

## 7. `@Profile` vs `spring.profiles.active`

| `@Profile` | `spring.profiles.active` |
|---|---|
| Controls which beans or configuration classes are eligible for registration. | Selects which profiles are active. |
| Used on classes or `@Bean` methods. | Configured through properties, environment variables, or command-line arguments. |
| Example: `@Profile("prod")` | Example: `spring.profiles.active=prod` |

These features work together: activating a profile selects the matching configuration files and profile-conditional beans.

## 8. Important interview points

- If no profile is explicitly active, Spring Boot uses the `default` profile for profile-based configuration behavior.
- `spring.profiles.active` and `spring.profiles.default` must be declared in non-profile-specific configuration documents, not in a profile-specific document such as `application-dev.properties`.
- Profiles are useful for configuration management, but they are not a security boundary.
- Never store production secrets in source-controlled profile files.
- In production, prefer deployment-specific environment variables or a secrets manager to choose the active profile and supply credentials.

## 9. Interview-ready answer

**Question: What is a Spring Boot profile, and why do we use it?**

A Spring Boot profile is a mechanism for maintaining environment-specific configuration and beans. It allows the same application to behave differently in development, testing, and production without modifying the code.

We can define files such as `application-dev.properties` and `application-prod.properties`, and activate a profile using `spring.profiles.active` through a configuration property, environment variable, or command-line argument.

We can also use the `@Profile` annotation to register specific beans only when the corresponding profile is active. This helps us manage environment-specific configurations cleanly and reduces the need for environment-dependent code.

## Quick revision

- **Configuration files:** `application-dev.properties`, `application-prod.properties`
- **Activate a profile:** `spring.profiles.active=prod`
- **Environment variable:** `SPRING_PROFILES_ACTIVE=prod`
- **Conditional bean:** `@Profile("prod")`
- **Main benefit:** Different environment configurations without changing application code.


---


