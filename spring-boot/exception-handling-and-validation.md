---
title: Exception Handling and Validation
parent: Spring Boot
nav_order: 6
---

# Exception Handling and Validation
{: .no_toc }

Global error handling and Bean Validation.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How to Handle Exceptions in Spring Framework?

Spring provides several mechanisms for exception handling. In a Spring Boot REST API, the most common approach is to use **`@ExceptionHandler` with `@ControllerAdvice` or `@RestControllerAdvice`** to handle exceptions centrally and return consistent HTTP responses.

### 1. Using a `try-catch` Block

Handle exceptions locally inside the method where they occur.

```java
public User getUser(Long id) {
    try {
        return userRepository.findById(id)
                .orElseThrow(() ->
                        new RuntimeException("User not found"));
    } catch (RuntimeException ex) {
        System.err.println("Error: " + ex.getMessage());
        throw ex;
    }
}
```

**Advantages**
- Useful for handling specific exceptions locally.
- Allows recovery or alternative logic within the method.

**Disadvantages**
- Repeating exception-handling logic across multiple methods makes code harder to maintain.
- Avoid catching exceptions just to log and rethrow them without adding value.

### 2. Using `@ExceptionHandler`

The `@ExceptionHandler` annotation handles specific exceptions thrown by controller methods. It can be defined inside a controller or within a global advice class.

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        if (id <= 0) {
            throw new IllegalArgumentException(
                    "Invalid user ID");
        }

        return new User(id, "Danish");
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<String> handleInvalidArgument(
            IllegalArgumentException ex) {

        return ResponseEntity
                .badRequest()
                .body(ex.getMessage());
    }
}
```

**Key points**
- Handles exceptions of the specified type thrown during request processing.
- When defined in a controller, it is scoped to that controller.
- Can return a response body, `ResponseEntity`, or another supported response type.

### 3. Using `@ControllerAdvice` and `@RestControllerAdvice`

These annotations allow exception handling to be shared across multiple controllers.

- `@ControllerAdvice`: provides global controller advice; response-body behavior depends on the handler and annotations used.
- `@RestControllerAdvice`: combines `@ControllerAdvice` and `@ResponseBody`, making it especially suitable for REST APIs.

**Example: Global exception handling**

First, create a custom exception:

```java
public class UserNotFoundException extends RuntimeException {

    public UserNotFoundException(String message) {
        super(message);
    }
}
```

Use it in the service layer:

```java
@Service
public class UserService {

    public User getUser(Long id) {
        if (id == 1) {
            return new User(id, "Danish");
        }

        throw new UserNotFoundException(
                "User not found with ID: " + id);
    }
}
```

Create a global exception handler:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<Map<String, String>> handleUserNotFound(
            UserNotFoundException ex) {

        Map<String, String> body = new HashMap<>();
        body.put("error", "USER_NOT_FOUND");
        body.put("message", ex.getMessage());

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(body);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<Map<String, String>> handleGenericException(
            Exception ex) {

        Map<String, String> body = new HashMap<>();
        body.put("error", "INTERNAL_SERVER_ERROR");
        body.put("message", "An unexpected error occurred");

        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(body);
    }
}
```

**Example response for a missing user**

HTTP status: `404 Not Found`

```json
{
  "error": "USER_NOT_FOUND",
  "message": "User not found with ID: 2"
}
```

**Best practices**
- Use custom exceptions to represent meaningful business errors.
- Return appropriate HTTP status codes.
- Keep response structures consistent across endpoints.
- Do not expose stack traces, database details, secrets, or internal exception messages in production responses.
- Log unexpected errors on the server, with appropriate care not to log sensitive data.
- Avoid a broad `Exception.class` handler that accidentally hides known errors; define specific handlers first and test the resulting responses.

### 4. Using `HandlerExceptionResolver`

`HandlerExceptionResolver` is a Spring MVC interface that lets you customize how exceptions are resolved during request processing.

```java
@Component
public class CustomExceptionResolver
        implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler,
            Exception ex) {

        if (ex instanceof IllegalArgumentException) {
            try {
                response.setStatus(
                        HttpServletResponse.SC_BAD_REQUEST);
                response.getWriter().write("Invalid request");
                return new ModelAndView();
            } catch (IOException ioEx) {
                return null;
            }
        }

        return null;
    }
}
```

**Key points**
- Gives lower-level control over exception resolution in Spring MVC.
- Returning a non-null `ModelAndView` indicates that the exception has been handled; returning `null` allows other resolvers to try.
- Useful for custom framework-level behavior, but usually unnecessary for ordinary REST APIs.

### 5. Using `@ResponseStatus`

The `@ResponseStatus` annotation associates an exception with an HTTP status code.

```java
@ResponseStatus(
        value = HttpStatus.NOT_FOUND,
        reason = "User not found"
)
public class UserNotFoundException extends RuntimeException {

    public UserNotFoundException(String message) {
        super(message);
    }
}
```

When this exception is thrown, Spring MVC can map it to HTTP `404 Not Found`.

**Important:** For REST APIs that need a structured JSON error body, using `@RestControllerAdvice` with `ResponseEntity` is generally more flexible than relying on `reason`.

### 6. Using `ErrorPage` or Spring Boot Error Handling

For traditional web applications, an error page can provide a user-friendly page when an error occurs. In Spring Boot, the built-in error handling mechanism commonly routes errors to `/error`, which can be customized.

For example, in a traditional Spring Boot MVC application, you might create:

```text
src/main/resources/templates/error/404.html
src/main/resources/templates/error/500.html
```

These templates can be used for suitable HTML error responses when the application is configured with a compatible view technology.

**Important distinction:** `ErrorPage` or `/error` handling is not the usual first choice for handling business exceptions in REST APIs. For REST endpoints, use `@RestControllerAdvice` to return an appropriate JSON response.

### 7. Comparison of Exception-Handling Mechanisms

| Mechanism | Scope | Typical use |
|---|---|---|
| `try-catch` | Local method | Recover from or handle a specific operation failure |
| `@ExceptionHandler` | Controller or advice | Handle selected exception types |
| `@ControllerAdvice` | Multiple controllers | Centralized MVC exception handling |
| `@RestControllerAdvice` | Multiple REST controllers | Centralized JSON error responses |
| `HandlerExceptionResolver` | Spring MVC request processing | Custom exception-resolution behavior |
| `@ResponseStatus` | Exception class or handler | Associate an exception with an HTTP status |
| Error page / `/error` | Web application error handling | Render user-friendly error pages or fallback responses |

### Interview Question: How do you handle exceptions in a Spring Boot microservices application?

**Sample interview answer:**

"In Spring Boot, I handle exceptions at different levels depending on their purpose. I use `try-catch` for local recovery when an operation can be handled meaningfully. For REST APIs, I create custom exceptions such as `UserNotFoundException` and use `@RestControllerAdvice` with `@ExceptionHandler` to handle them centrally.

I return appropriate HTTP status codes, such as 400 for invalid requests, 404 for missing resources, and 500 for unexpected server errors. I also maintain a consistent error-response structure, log unexpected failures, and avoid exposing internal implementation details to clients. In a microservices architecture, I apply this approach consistently across services so that API consumers can interpret errors reliably."

### Quick Revision

- **Local exception handling:** `try-catch`
- **Handle a specific exception:** `@ExceptionHandler`
- **Centralized REST exception handling:** `@RestControllerAdvice`
- **Map an exception to an HTTP status:** `@ResponseStatus`
- **Customize Spring MVC exception resolution:** `HandlerExceptionResolver`
- **Render web error pages:** `/error` or configured error pages

**Interview tip:** If asked for the preferred approach in a Spring Boot REST API, start with **custom exceptions + `@RestControllerAdvice` + `@ExceptionHandler`**.


---


# Global Exception Handling in Spring Boot Using `@ControllerAdvice`

## 1. What is `@ControllerAdvice`?

`@ControllerAdvice` is a Spring annotation used to define **global logic for controllers**, including centralized exception handling, data binding, and model attributes.

In REST APIs, it is commonly used with `@ExceptionHandler` to handle exceptions thrown by multiple controllers in one place and return consistent HTTP error responses.

For example, instead of writing separate `try-catch` blocks in every controller, you can handle exceptions such as `EmployeeNotFoundException`, validation errors, and unexpected server errors in a centralized class.

### Important annotations

| Annotation | Purpose |
|---|---|
| `@ControllerAdvice` | Provides cross-controller exception handling and other MVC advice. |
| `@RestControllerAdvice` | Combines `@ControllerAdvice` and `@ResponseBody`, making handler responses suitable for REST APIs. |
| `@ExceptionHandler` | Maps specified exception types to handler methods. |
| `@ResponseStatus` | Associates an exception or handler response with an HTTP status. |
| `@ResponseBody` | Writes a handler's return value to the HTTP response body. |
| `ResponseEntity<T>` | Allows you to control the HTTP status, headers, and response body. |

For a Spring Boot REST API, `@RestControllerAdvice` is usually the most convenient choice.

## 2. How does global exception handling work?

<box border radius="lg" padding={3} align="center" gap={2}>
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **Client sends HTTP request**

    <text size="sm">For example, `GET /employees/100`</text>
  </box>
  <icon name="arrow-down" size="lg" color="secondary" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **Controller → Service → Repository**

    <text size="sm">The application processes the request.</text>
  </box>
  <icon name="arrow-down" size="lg" color="secondary" />
  <box border radius="md" padding={3} width="100%" align="center">
    **Exception is thrown**

    <text size="sm">For example, `EmployeeNotFoundException`</text>
  </box>
  <icon name="arrow-down" size="lg" color="secondary" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **Global exception handler**

    <text size="sm">Spring selects a matching `@ExceptionHandler` method.</text>
  </box>
  <icon name="arrow-down" size="lg" color="secondary" />
  <box background="surface" border radius="md" padding={3} width="100%" align="center">
    **Consistent HTTP error response**

    <text size="sm">For example, `404 Not Found` with a JSON error body.</text>
  </box>
</box>

Spring MVC resolves exceptions through its exception-handling infrastructure. A matching local controller handler generally takes precedence over a global advice handler. The global handler applies when the exception is not handled earlier in the request-processing chain.

## 3. Step-by-step implementation

Let's implement exception handling for an Employee REST API.

### Step 1: Create a custom exception

`EmployeeNotFoundException.java`

```java
public class EmployeeNotFoundException extends RuntimeException {

    public EmployeeNotFoundException(Long id) {
        super("Employee not found with id: " + id);
    }
}
```

We extend `RuntimeException` because this is an unchecked exception. It can propagate through the service and controller without requiring a `throws` declaration.

### Step 2: Create an error response DTO

`ErrorResponse.java`

```java
import java.time.Instant;

public record ErrorResponse(
        Instant timestamp,
        int status,
        String error,
        String message,
        String path
) {}
```

This record defines a consistent JSON structure for API errors.

Example response:

```json
{
  "timestamp": "2026-10-19T00:00:00Z",
  "status": 404,
  "error": "Not Found",
  "message": "Employee not found with id: 100",
  "path": "/employees/100"
}
```

The timestamp above is illustrative.

### Step 3: Create the global exception handler

`GlobalExceptionHandler.java`

```java
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.Instant;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleEmployeeNotFound(
            EmployeeNotFoundException ex,
            HttpServletRequest request) {

        ErrorResponse error = new ErrorResponse(
                Instant.now(),
                HttpStatus.NOT_FOUND.value(),
                HttpStatus.NOT_FOUND.getReasonPhrase(),
                ex.getMessage(),
                request.getRequestURI()
        );

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(error);
    }
}
```

**How this works:**

- `@RestControllerAdvice` registers the class as global REST controller advice.
- `@ExceptionHandler(EmployeeNotFoundException.class)` specifies the exception type handled by this method.
- `HttpServletRequest` provides request details, such as the URI.
- `ResponseEntity` returns both the JSON body and HTTP status.
- The client receives `404 Not Found` instead of an unhandled server error.

### Step 4: Throw the exception from the service

```java
@Service
public class EmployeeService {

    public Employee getEmployeeById(Long id) {
        return employeeRepository.findById(id)
                .orElseThrow(() ->
                        new EmployeeNotFoundException(id));
    }
}
```

The service throws the exception when the employee does not exist. The exception propagates to Spring MVC, where the global handler processes it.

### Step 5: Keep the controller simple

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    private final EmployeeService employeeService;

    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    @GetMapping("/{id}")
    public Employee getEmployeeById(@PathVariable Long id) {
        return employeeService.getEmployeeById(id);
    }
}
```

No `try-catch` block is required in the controller for this exception. The global handler is responsible for producing the error response.

## 4. Handle multiple exception types

A production API normally handles more than one exception type.

You can add methods to the same advice class:

```java
@ExceptionHandler(IllegalArgumentException.class)
public ResponseEntity<ErrorResponse> handleBadRequest(
        IllegalArgumentException ex,
        HttpServletRequest request) {

    ErrorResponse error = new ErrorResponse(
            Instant.now(),
            HttpStatus.BAD_REQUEST.value(),
            HttpStatus.BAD_REQUEST.getReasonPhrase(),
            ex.getMessage(),
            request.getRequestURI()
    );

    return ResponseEntity
            .badRequest()
            .body(error);
}
```

You can also handle validation errors, such as those raised when request DTOs use `@Valid`. In Spring MVC, validation may raise `MethodArgumentNotValidException`; other validation paths can raise different exceptions.

For a Spring Boot REST API, override the appropriate validation-exception handling method or add a specific handler. Ensure that validation failures produce a clear `400 Bad Request` response without exposing internal implementation details.

## 5. Handle unexpected exceptions

You may want a fallback for unexpected exceptions. For example:

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<ErrorResponse> handleUnexpectedException(
        Exception ex,
        HttpServletRequest request) {

    // Log the full exception and stack trace using your logger.
    // Do not expose internal exception details to the client.

    ErrorResponse error = new ErrorResponse(
            Instant.now(),
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase(),
            "An unexpected error occurred",
            request.getRequestURI()
    );

    return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(error);
}
```

In production, log the exception with a correlation or trace ID so that engineers can investigate it. Avoid returning `ex.getMessage()` for unexpected server errors because it can disclose SQL details, internal paths, or sensitive implementation information.

A broad `Exception.class` handler should be a last-resort fallback, not a replacement for handling known exceptions appropriately.

## 6. `@ControllerAdvice` vs `@RestControllerAdvice`

| `@ControllerAdvice` | `@RestControllerAdvice` |
|---|---|
| Global advice for Spring MVC controllers. | Global advice with response-body semantics. |
| Useful for MVC applications returning views. | Convenient for REST APIs returning JSON or other response bodies. |
| A handler may return a view unless it uses `@ResponseBody` or `ResponseEntity`. | Handler return values are written to the response body by default. |

For a traditional MVC application, you might return an error page from a handler. For a REST API, you would normally return a structured JSON error response.

## 7. Can exception handling be restricted to a package?

Yes. Use `basePackages` to limit the advice to controllers in a specific package.

```java
@RestControllerAdvice(basePackages = "com.example.myapp.controllers")
public class ApiExceptionHandler {
    // Exception handler methods
}
```

Other useful scoping options include `basePackageClasses`, `assignableTypes`, and `annotations`. If you do not specify a scope, the advice generally applies across controllers in the application context, subject to Spring MVC's handler-selection rules.

## 8. Do we need to implement `HandlerExceptionResolver`?

Usually, no.

Spring MVC already provides exception-resolution infrastructure, and `@ControllerAdvice` with `@ExceptionHandler` is the idiomatic approach for most Spring Boot applications.

`HandlerExceptionResolver` and `AbstractHandlerExceptionResolver` are lower-level extension points. Use them when you need custom exception-resolution behavior that cannot be addressed cleanly with controller advice.

You also generally do not need to register your advice manually as a `HandlerExceptionResolver` bean or autowire it into a configuration class. A correctly component-scanned advice class is discovered by Spring.

`ResponseEntityExceptionHandler` is another option: it provides handling for many Spring MVC exceptions and can be extended when you want to customize framework-generated error responses.

## 9. Best practices for exception handling

1. **Use specific custom exceptions** for expected business failures, such as an employee not being found.
2. **Centralize common API error handling** with `@RestControllerAdvice`.
3. **Choose the correct HTTP status** — `400` for invalid requests, `404` for missing resources, `409` for applicable conflicts, and `500` for unexpected server failures.
4. **Use a consistent error schema** so clients can process errors predictably.
5. **Never expose stack traces or sensitive internal details** in production responses.
6. **Log unexpected exceptions** with trace IDs and appropriate context.
7. **Handle validation exceptions** and return actionable but safe messages.
8. **Avoid swallowing exceptions** or returning success responses when an operation failed.
9. **Keep controllers clean** and place business validation and exception decisions in the appropriate layers.
10. **Test error responses** to verify status codes, response schemas, and sensitive-data handling.

## 10. Interview-ready answer

**Question: How do you handle exceptions in Spring Boot?**

I use centralized exception handling with `@RestControllerAdvice` and `@ExceptionHandler` for REST APIs. I define custom exceptions for business scenarios, such as `EmployeeNotFoundException`, and handle them globally by returning a consistent error DTO with an appropriate HTTP status through `ResponseEntity`.

For example, if an employee does not exist, the service throws `EmployeeNotFoundException`, and the global handler returns `404 Not Found`. I also handle validation errors, log unexpected exceptions with correlation IDs, and avoid exposing internal stack traces or sensitive information to clients.

For special cases, I can use local `@ExceptionHandler` methods or extend `ResponseEntityExceptionHandler` to customize Spring MVC's built-in exception responses.

## Quick revision

- **`@ControllerAdvice`:** Shared controller advice.
- **`@RestControllerAdvice`:** Global advice with response-body semantics.
- **`@ExceptionHandler`:** Maps exceptions to handler methods.
- **`ResponseEntity`:** Controls HTTP status, headers, and body.
- **Custom exceptions:** Represent meaningful business failures.
- **`@ResponseStatus`:** Associates an exception or handler with an HTTP status.
- **`HandlerExceptionResolver`:** Lower-level extension point, usually unnecessary for standard cases.
- **Production rule:** Return safe errors, log diagnostic details, and use appropriate status codes.



---

