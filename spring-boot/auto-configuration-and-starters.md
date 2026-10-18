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

## What method would you call before starting or loading a Spring Boot application?

{: .no_toc }

### Interview Answer

In Spring Boot, the method or callback I choose depends on **when the code needs to execute during the application lifecycle**.

The main options are:

- `main()` — the application's entry point, called to launch Spring Boot.
- `@PostConstruct` — executes after a bean is constructed and its dependencies are injected.
- `ApplicationContextInitializer` — executes while the application context is being prepared, before it is refreshed.
- `ApplicationRunner` and `CommandLineRunner` — execute after the application context has been refreshed and the application has started initializing its runner callbacks.
- `@EventListener` — executes in response to a specific Spring application lifecycle event.

If the requirement is to execute code **before the Spring application context is fully loaded**, I would consider an appropriate application lifecycle event or an `ApplicationContextInitializer`. If the requirement is to initialize data after the context is ready, I would use `ApplicationRunner` or `CommandLineRunner`.

### 1. `main()` Method

The `main()` method is the entry point of a standard Spring Boot application.

```java
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

`SpringApplication.run()` bootstraps the application, prepares the environment, creates and refreshes the application context, and completes startup processing.

**Use case:** Starting the application.

### 2. `@PostConstruct`

A method annotated with `@PostConstruct` executes after the bean's dependency injection and initialization callbacks.

```java
@Service
public class StartupService {

    @PostConstruct
    public void initialize() {
        System.out.println("Bean initialized");
    }
}
```

**Use case:** Initializing bean-specific state after dependencies have been injected.

**Important:** This method is not a callback that runs before the entire Spring Boot application starts. It runs during bean initialization, potentially before other beans have been fully initialized.

### 3. `ApplicationRunner`

`ApplicationRunner` executes after the application context has been refreshed and receives parsed application arguments through `ApplicationArguments`.

```java
@Component
public class StartupRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {
        System.out.println("ApplicationRunner executed");
    }
}
```

**Use case:** Loading reference data, validating startup configuration, or performing startup tasks after the context is initialized.

### 4. `CommandLineRunner`

`CommandLineRunner` serves a similar purpose but receives the original command-line arguments as a `String...` array.

```java
@Component
public class StartupRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {
        System.out.println("CommandLineRunner executed");
    }
}
```

**Difference:**

| Feature | ApplicationRunner | CommandLineRunner |
|---|---|---|
| Arguments | `ApplicationArguments` | `String...` |
| Argument handling | Parsed options and non-option arguments | Raw arguments |
| Execution timing | After context refresh | After context refresh |
| Common use | Startup initialization | Startup initialization |

Both interfaces run during startup after the context is refreshed. Use `@Order` or `Ordered` when the execution order of multiple runners matters.

### 5. `@EventListener` and Application Events

Spring Boot publishes lifecycle events at different stages of startup.

Common events include:

| Event | When it occurs |
|---|---|
| `ApplicationStartingEvent` | Early in startup, before the application context is created |
| `ApplicationEnvironmentPreparedEvent` | After the environment is prepared, before the context is created |
| `ApplicationContextInitializedEvent` | After context initialization, before bean definitions are loaded |
| `ApplicationPreparedEvent` | After bean definitions are loaded, before context refresh |
| `ApplicationStartedEvent` | After context refresh, before application runners execute |
| `ApplicationReadyEvent` | After application runners complete, when the application is considered ready |

For example, to run code when the application is ready:

```java
@Component
public class ReadyListener {

    @EventListener(ApplicationReadyEvent.class)
    public void onApplicationReady() {
        System.out.println("Application is ready");
    }
}
```

**Important:** Very early events, such as `ApplicationStartingEvent`, occur before regular Spring beans are available for normal dependency injection. For such events, use an appropriately registered listener or an initializer rather than assuming an ordinary `@Component` listener will receive them.

### 6. `ApplicationContextInitializer`

An `ApplicationContextInitializer` allows you to customize an application context before it is refreshed.

```java
public class MyContextInitializer
        implements ApplicationContextInitializer<
                ConfigurableApplicationContext> {

    @Override
    public void initialize(
            ConfigurableApplicationContext context) {
        System.out.println("Context is being initialized");
    }
}
```

Register it with `SpringApplication`:

```java
public static void main(String[] args) {
    SpringApplication app =
            new SpringApplication(MyApplication.class);

    app.addInitializers(new MyContextInitializer());
    app.run(args);
}
```

**Use case:** Customizing the context before bean creation and refresh, such as registering context-level configuration.

### Quick Interview Revision

**Question:** What method would you call before starting or loading a Spring Boot application?

**Answer:** It depends on the lifecycle stage. The `main()` method starts the application through `SpringApplication.run()`. For initialization before context refresh, I can use `ApplicationContextInitializer` or an appropriately registered early application listener. For bean-level initialization after dependency injection, I use `@PostConstruct`. For tasks after the application context has been refreshed, I use `ApplicationRunner` or `CommandLineRunner`. If the task must run when the application is fully ready, I use `ApplicationReadyEvent`.


---

# How to Set Properties Across Different Environments in Spring Boot (DEV, QA, PROD)?

## 1. Overview

In Spring Boot, we can configure environment-specific properties using **profiles and externalized configuration**. This allows the same application artifact to run in Development, QA, and Production with different database URLs, logging levels, API endpoints, and other settings.

For example:

- **DEV:** Local database, debug logging, test services.
- **QA:** QA database, integration testing, staging services.
- **PROD:** Production database, optimized logging, production services.

The application code remains the same; only the configuration changes.

## 2. Approach 1: Use Spring Boot profiles

Create a separate configuration file for each environment.

### Project structure

```text
src/main/resources/
├── application.properties
├── application-dev.properties
├── application-qa.properties
└── application-prod.properties
```

### `application.properties` — common properties

```properties
spring.application.name=employee-service
server.port=8080
```

### `application-dev.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/employee_dev
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

logging.level.root=DEBUG
```

### `application-qa.properties`

```properties
spring.datasource.url=jdbc:mysql://qa-db:3306/employee_qa
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

logging.level.root=INFO
```

### `application-prod.properties`

```properties
spring.datasource.url=jdbc:mysql://prod-db:3306/employee_prod
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

logging.level.root=WARN
```

The database hostnames above are examples. In a real deployment, use the actual environment-specific endpoints.

Notice that database credentials are supplied through environment variables rather than hardcoded into files.

## 3. How to activate each environment

Spring Boot supports several ways to activate a profile.

### Option A: Command-line argument

```bash
# Development
java -jar employee-service.jar --spring.profiles.active=dev

# QA
java -jar employee-service.jar --spring.profiles.active=qa

# Production
java -jar employee-service.jar --spring.profiles.active=prod
```

### Option B: Environment variable

```bash
export SPRING_PROFILES_ACTIVE=dev
export DB_USERNAME=dev_user
export DB_PASSWORD='your-secret'

java -jar employee-service.jar
```

For QA and Production, the deployment platform should supply the corresponding environment variables and secrets.

**Best practice:** Choose the active profile at deployment time rather than hardcoding `spring.profiles.active` in the packaged application.

## 4. Approach 2: Use environment variables directly

Spring Boot can read configuration values from environment variables without requiring a separate profile file for every property.

For example, you could write:

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

Then supply different values in each environment.

| Environment | `DB_URL` example |
|---|---|
| DEV | `jdbc:mysql://localhost:3306/employee_dev` |
| QA | `jdbc:mysql://qa-db:3306/employee_qa` |
| PROD | `jdbc:mysql://prod-db:3306/employee_prod` |

This approach is especially useful with Docker, Kubernetes, and CI/CD pipelines.

The same configuration file can be used in every environment, while the deployment supplies the appropriate values.

## 5. Approach 3: Use external configuration files

You can keep environment-specific configuration outside the application JAR.

For example:

```text
/config/
├── application-dev.properties
├── application-qa.properties
└── application-prod.properties
```

You can specify an external configuration location when launching the application:

```bash
java -jar employee-service.jar \
  --spring.config.additional-location=file:/config/ \
  --spring.profiles.active=prod
```

Spring Boot can then load applicable configuration from that location in addition to its normal configuration locations.

This is useful when operations teams need to manage configuration independently of application builds.

Be careful with configuration precedence: command-line properties and other higher-priority property sources can override values in configuration files.

## 6. Approach 4: Use Spring Cloud Config Server

For microservices, configuration may be managed centrally using Spring Cloud Config Server.

<box gap={2} align="center">
  <box border radius="lg" padding={3} width="100%" align="center">
    **Configuration repository**

    <text color="secondary" size="sm">Git repository containing environment-specific properties</text>
  </box>
  <icon name="arrow-down" size="lg" color="secondary" />
  <box border radius="lg" padding={3} width="100%" align="center">
    **Spring Cloud Config Server**

    <text color="secondary" size="sm">Provides configuration to clients</text>
  </box>
  <icon name="arrow-down" size="lg" color="secondary" />
  <grid columns={3} gap={2}>
    <grid-item>
      <box background="surface-secondary" radius="md" padding={2} align="center">
        **DEV**

        <text size="sm">Properties</text>
      </box>
    </grid-item>
    <grid-item>
      <box background="surface-secondary" radius="md" padding={2} align="center">
        **QA**

        <text size="sm">Properties</text>
      </box>
    </grid-item>
    <grid-item>
      <box background="surface-secondary" radius="md" padding={2} align="center">
        **PROD**

        <text size="sm">Properties</text>
      </box>
    </grid-item>
  </grid>
</box>

Example configuration files in the Git repository:

```text
employee-service.properties
employee-service-dev.properties
employee-service-qa.properties
employee-service-prod.properties
```

The Config Server serves common and profile-specific configuration to the application based on its application name and active profile.

This approach is useful when many microservices need centralized configuration management, versioning, and consistent environment-specific settings. It requires additional infrastructure and suitable security controls.

## 7. What about `spring.profiles.include`?

`spring.profiles.include` can activate additional profiles alongside the active profile.

For example:

```properties
spring.profiles.include=common
```

However, it is not the same as selecting an environment. For example, you might use a `common` profile alongside `qa`.

For profile activation, use `spring.profiles.active`. For profile groups, Spring Boot also supports `spring.profiles.group` when one logical profile should activate several related profiles.

These profile activation settings should be defined in non-profile-specific configuration, rather than inside `application-dev.properties` or another profile-specific document.

## 8. Which approach should we use?

| Approach | Best suited for |
|---|---|
| Profile-specific files | Small to medium applications with distinct environment settings |
| Environment variables | Containers, CI/CD, and deployment-specific values |
| External configuration files | Applications where configuration is managed outside the JAR |
| Spring Cloud Config Server | Multiple microservices needing centralized configuration |

In practice, these approaches can be combined. For example, use profile-specific files for non-sensitive defaults, environment variables for deployment-specific values and secrets, and Spring Cloud Config Server if centralized configuration is needed across many services.

Never commit production passwords, API keys, or database credentials to source control. Use a secrets manager or your deployment platform's secure secret mechanism.

## 9. Interview-ready answer

**Question: How do you set properties across DEV, QA, and PROD environments in Spring Boot?**

I use Spring Boot profiles and externalized configuration. I create `application-dev.properties`, `application-qa.properties`, and `application-prod.properties` for environment-specific settings, while keeping shared properties in `application.properties`.

I activate the required profile through `SPRING_PROFILES_ACTIVE` or a command-line argument. For deployment-specific values, such as database URLs and credentials, I use environment variables or a secrets manager instead of hardcoding them.

For a microservices architecture with many services, I may use Spring Cloud Config Server to centralize and version configuration. This lets us promote the same application artifact across environments while keeping configuration separate from application code.

## Quick revision

- **Profiles:** `application-dev.properties`, `application-qa.properties`, `application-prod.properties`
- **Activate:** `--spring.profiles.active=qa`
- **Environment variable:** `SPRING_PROFILES_ACTIVE=prod`
- **External properties:** `--spring.config.additional-location=file:/config/`
- **Centralized configuration:** Spring Cloud Config Server
- **Security:** Keep secrets outside source-controlled configuration files.
- **Best practice:** Build once, deploy the same artifact, and supply environment-specific configuration at runtime.

---


