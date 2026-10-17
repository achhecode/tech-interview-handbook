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

## How do you handle exceptions globally in Spring Boot?

Medium
{: .label .label-yellow }

**Answer**

Use `@RestControllerAdvice` with `@ExceptionHandler` methods.

```java
@RestControllerAdvice
class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    ResponseEntity<ApiError> notFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ApiError("NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    ResponseEntity<ApiError> invalid(MethodArgumentNotValidException ex) {
        return ResponseEntity.badRequest().body(new ApiError("VALIDATION", "Invalid request"));
    }
}
record ApiError(String code, String message) {}
```

Validate input with `@Valid` on `@RequestBody` and constraints such as `@NotBlank`, `@Email`, `@Size`.

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
