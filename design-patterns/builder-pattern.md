---
title: Builder
parent: Design Patterns and LLD
nav_order: 3
---

# Builder
{: .no_toc }

Constructs complex objects step by step, especially when they have many optional parameters.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

1. TOC
{:toc}
</details>

---

## 1. What is the Builder Design Pattern?

{: .label .label-green }

**Answer**

The Builder is a creational design pattern that constructs complex objects step by step. It separates the construction process from the final object's representation.

It is especially useful when a class has many fields, optional parameters, or configuration options.

### Why do we need it?

Suppose an `Employee` class has five fields:

```java
public class Employee {
    private String name;
    private int age;
    private String department;
    private String email;
    private double salary;
}
```

Without the Builder Pattern, we might need a constructor with many parameters:

```java
Employee employee = new Employee(
    "Danish", 28, "Engineering",
    "danish@example.com", 120000
);
```

This approach becomes difficult to read when the number of parameters increases. It can also be confusing when several parameters have the same type.

The Builder Pattern lets us construct the object using named method calls.

```java
Employee employee = new Employee.Builder()
        .name("Danish")
        .age(28)
        .department("Engineering")
        .email("danish@example.com")
        .salary(120000)
        .build();
```

The method names make the purpose of each value clear.

---

## 2. How do you implement the Builder Pattern in Java?

{: .label .label-green }

**Answer**

A typical implementation uses a nested `Builder` class that collects the values and creates the final object through a `build()` method.

### Complete Java example

```java
public class Employee {

    private final String name;
    private final int age;
    private final String department;
    private final String email;
    private final double salary;

    private Employee(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.department = builder.department;
        this.email = builder.email;
        this.salary = builder.salary;
    }

    public static class Builder {

        private String name;
        private int age;
        private String department;
        private String email;
        private double salary;

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder department(String department) {
            this.department = department;
            return this;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder salary(double salary) {
            this.salary = salary;
            return this;
        }

        public Employee build() {
            return new Employee(this);
        }
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public String getDepartment() {
        return department;
    }

    public String getEmail() {
        return email;
    }

    public double getSalary() {
        return salary;
    }
}
```

### Usage

```java
public class Main {

    public static void main(String[] args) {

        Employee employee = new Employee.Builder()
                .name("Danish")
                .age(28)
                .department("Engineering")
                .email("danish@example.com")
                .salary(120000)
                .build();

        System.out.println(employee.getName());
        System.out.println(employee.getDepartment());
    }
}
```

**Output**

```text
Danish
Engineering
```

### How it works

1. `new Employee.Builder()` creates a builder object.
2. Each method stores a value and returns the same builder.
3. Method chaining allows multiple configuration calls in one expression.
4. `build()` creates the final `Employee` object.
5. The private constructor copies the values into the employee.
6. The `final` fields prevent reassignment after construction.

**Important:** In this example, all fields are optional because no validation is performed. A production builder should validate required fields and enforce business rules inside `build()` or the object's constructor.

---

## 3. What is method chaining in the Builder Pattern?

{: .label .label-blue }

**Answer**

Method chaining means calling multiple methods on the same object in a single expression.

In a typical Builder implementation, each configuration method returns `this`, which refers to the current builder object.

```java
public Builder name(String name) {
    this.name = name;
    return this;
}
```

Without chaining:

```java
Employee.Builder builder = new Employee.Builder();

builder.name("Danish");
builder.age(28);
builder.department("Engineering");

Employee employee = builder.build();
```

With chaining:

```java
Employee employee = new Employee.Builder()
        .name("Danish")
        .age(28)
        .department("Engineering")
        .build();
```

Both examples use the same builder mechanism. Chaining simply makes the code more concise and readable.

---

## 4. How do you make a Builder object immutable and validate fields?

{: .label .label-yellow }

**Answer**

A common approach is to make the final object immutable, use a private constructor, and validate required fields during construction.

### Example: Required fields and validation

```java
public class User {

    private final String username;
    private final String email;
    private final int age;

    private User(Builder builder) {
        this.username = builder.username;
        this.email = builder.email;
        this.age = builder.age;
    }

    public static class Builder {

        private String username;
        private String email;
        private int age;

        public Builder username(String username) {
            this.username = username;
            return this;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public User build() {
            if (username == null || username.isBlank()) {
                throw new IllegalStateException(
                        "Username is required"
                );
            }

            if (email == null || email.isBlank()) {
                throw new IllegalStateException(
                        "Email is required"
                );
            }

            if (age < 18) {
                throw new IllegalStateException(
                        "User must be at least 18"
                );
            }

            return new User(this);
        }
    }
}
```

**Usage**

```java
User user = new User.Builder()
        .username("danish")
        .email("danish@example.com")
        .age(28)
        .build();
```

If a required field is missing or a validation rule fails, `build()` throws an exception instead of returning an invalid object.

**Note:** `final` fields prevent reassignment of references or primitive values, but they do not automatically make mutable objects stored in those fields immutable. Defensive copies may be necessary for collections or other mutable objects.

---

## 5. What is the difference between Builder and Constructor?

{: .label .label-yellow }

**Answer**

| Constructor | Builder |
|---|---|
| Creates an object directly. | Collects configuration and creates an object through `build()`. |
| Can become difficult to read with many parameters. | Uses descriptive method names for each value. |
| Often requires separate constructors for different combinations of parameters. | Supports optional parameters without requiring a constructor for every combination. |
| Can enforce validation during construction. | Can collect values and validate them during `build()`. |
| Suitable for simple objects with few required fields. | Suitable for objects with many fields or configuration options. |

### Constructor example

```java
Employee employee = new Employee(
        "Danish", 28, "Engineering",
        "danish@example.com", 120000
);
```

### Builder example

```java
Employee employee = new Employee.Builder()
        .name("Danish")
        .age(28)
        .department("Engineering")
        .email("danish@example.com")
        .salary(120000)
        .build();
```

**Remember:** A constructor is often simpler for a small number of parameters. A Builder is useful when construction has many optional values or requires readable configuration.

---

## 6. What is the difference between Builder and Factory?

{: .label .label-yellow }

**Answer**

Both are creational design patterns, but they solve different problems.

| Builder Pattern | Factory Pattern |
|---|---|
| Constructs an object step by step. | Encapsulates object creation and selects which object to create. |
| Focuses on configuring a complex object. | Focuses on choosing or creating an implementation. |
| Often uses chained methods and `build()`. | Often exposes a factory method such as `create()`. |
| Example: Building an HTTP request with headers, body, and timeout. | Example: Creating a notification sender based on type. |

### Builder example

```java
Employee employee = new Employee.Builder()
        .name("Danish")
        .department("Engineering")
        .build();
```

### Factory example

```java
interface Notification {
    void send(String message);
}

class EmailNotification implements Notification {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class SmsNotification implements Notification {
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

class NotificationFactory {

    public static Notification create(String type) {
        if ("email".equalsIgnoreCase(type)) {
            return new EmailNotification();
        }

        if ("sms".equalsIgnoreCase(type)) {
            return new SmsNotification();
        }

        throw new IllegalArgumentException(
                "Unsupported notification type"
        );
    }
}
```

**Remember:** Builder focuses on how an object is constructed; Factory focuses on which object or implementation is created.

---

## 7. What is the difference between Builder and Prototype?

{: .label .label-blue }

**Answer**

| Builder Pattern | Prototype Pattern |
|---|---|
| Creates an object by configuring its fields or construction steps. | Creates an object by copying an existing prototype. |
| Focuses on controlled construction. | Focuses on cloning or copying. |
| Useful when many parameters must be configured. | Useful when creating a similar object from an existing object is more convenient than building from scratch. |

**Example:** Builder constructs a new employee from specified values; Prototype creates a copy of an existing employee configuration.

---

## 8. What are the advantages and disadvantages of the Builder Pattern?

{: .label .label-blue }

**Answer**

### Advantages

- **Readability:** Named methods make construction code easier to understand.
- **Optional parameters:** Supports different combinations of values without numerous constructors.
- **Validation:** Centralizes validation before creating the final object.
- **Immutability:** Works well with immutable objects.
- **Maintainability:** Adding optional fields can be easier than maintaining many overloaded constructors.

### Disadvantages

- **More code:** A manual builder adds another class and configuration methods.
- **Boilerplate:** Each new field may require another builder method.
- **Overengineering:** A builder may be unnecessary for simple objects with only one or two parameters.
- **Validation responsibility:** The builder must enforce required fields and other construction rules correctly.

---

## 9. Where is the Builder Pattern used in real-world applications?

{: .label .label-green }

**Answer**

The Builder Pattern is commonly used to construct objects with many optional settings or nested configuration values.

### Common examples

- **HTTP requests:** Configure URI, headers, body, and timeout.
- **Database queries:** Build complex query specifications.
- **Immutable DTOs:** Construct objects with many fields.
- **Cloud SDKs:** Configure client options, credentials, and endpoints.
- **Test data:** Create test objects with different field combinations.
- **Application configuration:** Assemble objects from multiple configuration options.

### Example: Java HTTP request

Java's built-in `HttpRequest` API provides a real Builder implementation.

```java
import java.net.URI;
import java.net.http.HttpRequest;
import java.time.Duration;

public class HttpRequestExample {

    public static void main(String[] args) {

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://example.com/api/users"))
                .timeout(Duration.ofSeconds(10))
                .header("Accept", "application/json")
                .GET()
                .build();

        System.out.println(request.uri());
        System.out.println(request.timeout());
    }
}
```

**How it works**

1. `newBuilder()` creates a builder.
2. `uri()` configures the request destination.
3. `timeout()` sets the request timeout.
4. `header()` adds a request header.
5. `GET()` specifies the HTTP method.
6. `build()` creates the final `HttpRequest`.

The code above constructs the request but does not send it.

---

## 10. How is the Builder Pattern used in Spring Boot?

{: .label .label-blue }

**Answer**

In Spring Boot applications, builders are useful for constructing DTOs, API responses, domain objects, and configuration objects.

### Example: Building an API response

```java
public class ApiResponse {

    private final boolean success;
    private final String message;
    private final Object data;

    private ApiResponse(Builder builder) {
        this.success = builder.success;
        this.message = builder.message;
        this.data = builder.data;
    }

    public boolean isSuccess() {
        return success;
    }

    public String getMessage() {
        return message;
    }

    public Object getData() {
        return data;
    }

    public static class Builder {

        private boolean success;
        private String message;
        private Object data;

        public Builder success(boolean success) {
            this.success = success;
            return this;
        }

        public Builder message(String message) {
            this.message = message;
            return this;
        }

        public Builder data(Object data) {
            this.data = data;
            return this;
        }

        public ApiResponse build() {
            return new ApiResponse(this);
        }
    }
}
```

### Usage in a REST controller

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EmployeeController {

    @GetMapping("/api/employee")
    public ApiResponse getEmployee() {

        return new ApiResponse.Builder()
                .success(true)
                .message("Employee retrieved successfully")
                .data("Danish")
                .build();
    }
}
```

Spring Boot serializes the returned object into JSON through its configured HTTP message converters.

**Note:** In production, use a typed DTO instead of `Object` for the `data` field when possible. This improves compile-time safety and API documentation.

### Using Lombok `@Builder`

Lombok can generate builder code automatically.

```java
import lombok.Builder;
import lombok.Getter;

@Getter
@Builder
public class EmployeeDto {
    private final String name;
    private final String department;
    private final String email;
}
```

Usage:

```java
EmployeeDto employee = EmployeeDto.builder()
        .name("Danish")
        .department("Engineering")
        .email("danish@example.com")
        .build();
```

**Important:** Lombok generates the builder during compilation. It reduces boilerplate but does not automatically enforce every business validation rule.

---

## 11. What is a fluent API in the Builder Pattern?

{: .label .label-blue }

**Answer**

A fluent API is an API designed to make method calls read like a sequence of instructions, commonly by returning the current object from each configuration method.

```java
Employee employee = new Employee.Builder()
        .name("Danish")
        .age(28)
        .department("Engineering")
        .build();
```

The chained calls form a readable sequence.

**Remember:** Builder is a design pattern; a fluent API is a style of API design. A Builder commonly uses a fluent API, but not every fluent API implements the Builder Pattern.

---

## 12. What is the difference between a mutable Builder and an immutable object?

{: .label .label-yellow }

**Answer**

A typical Builder is mutable while the final object is immutable.

| Builder | Final object |
|---|---|
| Stores values as configuration methods are called. | Stores the completed object's values. |
| Its fields can change during construction. | Its fields are commonly `final`. |
| Usually exists only during object creation. | Exists after `build()` returns. |
| May be reused, although a fresh builder is often clearer. | Should not expose operations that modify its state when designed to be immutable. |

Example:

```java
Employee employee = new Employee.Builder()
        .name("Danish")
        .age(28)
        .build();
```

Here, the builder collects values first. The `Employee` constructor copies those values into the final object's fields.

**Important:** Reusing a builder may carry over previously configured values. Unless reuse is deliberately supported and documented, create a new builder for each object.

---

## 13. What are the SOLID principles related to the Builder Pattern?

{: .label .label-blue }

**Answer**

The Builder Pattern can support good object-oriented design when used appropriately.

- **Single Responsibility Principle (SRP):** The builder handles object construction separately from the final object's business behavior.
- **Open/Closed Principle (OCP):** A builder can support additional optional configuration without introducing many overloaded constructors.
- **Dependency Inversion Principle (DIP):** This is not an automatic property of Builder; it depends on how the resulting objects and their dependencies are designed.

**Remember:** Builder improves construction and readability, but it does not automatically guarantee SOLID compliance.

---

## 14. Quick interview revision

{: .label .label-green }

| Question | Short answer |
|---|---|
| What is Builder? | A creational pattern that constructs objects step by step. |
| Why use Builder? | To simplify construction when objects have many optional parameters. |
| What does `build()` do? | Creates and returns the final object. |
| Why does a builder method return `this`? | To support method chaining. |
| Can Builder support immutable objects? | Yes; copy the values into final fields during construction. |
| Builder vs Constructor? | Builder improves readability for complex construction; constructors are simpler for straightforward objects. |
| Builder vs Factory? | Builder configures one object; Factory selects or creates an implementation. |
| Builder vs Prototype? | Builder constructs from configuration; Prototype copies an existing object. |
| Is Builder a creational pattern? | Yes. |
| Does Lombok support Builder? | Yes, through `@Builder`. |
| Is a Builder automatically thread-safe? | No. A typical mutable builder is not thread-safe. |
| Does Builder automatically validate fields? | No. Validation must be implemented explicitly. |
| What is a fluent API? | An API that supports readable chained method calls. |

---

## 15. Final recommendation

{: .label .label-green }

Remember this simple rule:

**Use the Builder Pattern when constructing an object requires many optional parameters, multiple configuration steps, or clear validation before the object is created.**

For Java and Spring Boot applications:

1. Prefer constructors for simple objects with a few required fields.
2. Use a Builder for complex object construction and readable configuration.
3. Keep the final object's state immutable where practical.
4. Validate required fields and business rules before returning the object.
5. Consider Lombok's `@Builder` when its generated behavior meets your requirements.