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

# How to Handle Security in Spring Boot?

## 1. What is Spring Security?

**Spring Security** is a framework that provides authentication, authorization, and protection against common web application security attacks.

In a Spring Boot application, we commonly use Spring Security to protect REST APIs, secure web pages, authenticate users, and restrict access based on roles or permissions.

The two most important concepts are:

- **Authentication:** Who are you? Verifies the user's identity.
- **Authorization:** What are you allowed to do? Determines which resources or operations the authenticated user can access.

For example, a user may successfully log in but still be denied access to an administrator endpoint.

## 2. How to add Spring Security to Spring Boot

Add the Spring Security starter to your Maven project:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Spring Boot configures security defaults when the starter is present. For a typical web application, requests are protected by default unless you configure different rules.

For a REST API, you usually define a `SecurityFilterChain` bean to specify your authentication mechanism and access rules.

## 3. Configure Spring Security using `SecurityFilterChain`

Your original example uses `WebSecurityConfigurerAdapter`. That approach is obsolete in modern Spring Security and was deprecated in Spring Security 5.7, then removed in Spring Security 6.

The recommended approach is to declare a `SecurityFilterChain` bean.

### Example: Role-based security

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());

        return http.build();
    }
}
```

### Explanation

| Configuration | Purpose |
|---|---|
| `permitAll()` | Allows access without authentication. |
| `hasRole("ADMIN")` | Requires the `ROLE_ADMIN` authority. |
| `hasAnyRole("USER", "ADMIN")` | Requires either `ROLE_USER` or `ROLE_ADMIN`. |
| `authenticated()` | Requires the caller to be authenticated. |
| `httpBasic()` | Enables HTTP Basic authentication. |

This example demonstrates access-control rules. HTTP Basic sends credentials with requests, so use HTTPS in production. A production API may instead use OAuth 2.0 resource-server JWT authentication.

## 4. Authentication using `UserDetailsService` and `PasswordEncoder`

Spring Security supports username/password authentication through `UserDetailsService`, which loads user information, and `PasswordEncoder`, which verifies stored password hashes.

### Configure a password encoder

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class PasswordConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

Passwords should be stored using an appropriate password-hashing algorithm, such as BCrypt or Argon2, rather than plaintext or reversible encryption.

### Example user service

```java
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String username) {
        // In production, retrieve the user and password hash
        // from a database or another trusted identity store.

        if ("admin".equals(username)) {
            return User.withUsername("admin")
                    .password("$2a$10$replaceWithARealBCryptHash")
                    .roles("ADMIN")
                    .build();
        }

        throw new UsernameNotFoundException("User not found");
    }
}
```

The hash above is a placeholder and must be replaced with a valid hash corresponding to the stored password. Real applications should retrieve users and their password hashes from a database or identity provider.

When a `UserDetailsService` and `PasswordEncoder` are configured, Spring Security can use them for username/password authentication. Do not hardcode real user credentials in application code.

## 5. How do you secure REST APIs using JWT?

JWT stands for JSON Web Token. It is commonly used to carry claims about an authenticated identity between services.

A typical token-based authentication flow is:

<box border radius="lg" padding={3} gap={2} align="center">
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **1. Client authenticates**

    <text size="sm">Sends credentials to an identity provider or authentication endpoint.</text>
  </box>
  <icon name="arrow-down" color="secondary" size="lg" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **2. Token issued**

    <text size="sm">The trusted issuer creates a signed access token.</text>
  </box>
  <icon name="arrow-down" color="secondary" size="lg" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **3. Client calls API**

    <text size="sm">Sends `Authorization: Bearer &lt;token&gt;`.</text>
  </box>
  <icon name="arrow-down" color="secondary" size="lg" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **4. Spring Security validates the token**

    <text size="sm">Checks signature, issuer, expiration, and configured claims.</text>
  </box>
  <icon name="arrow-down" color="secondary" size="lg" />
  <box border radius="md" padding={3} width="100%" align="center">
    **5. Authorization**

    <text size="sm">Allows or rejects the request based on authorities and access rules.</text>
  </box>
</box>

### Configure a JWT resource server

Add the OAuth2 resource-server starter:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

Then configure the resource server to validate tokens from a trusted issuer.

```java
@Configuration
public class ApiSecurityConfig {

    @Bean
    public SecurityFilterChain apiSecurityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 ->
                oauth2.jwt(Customizer.withDefaults())
            );

        return http.build();
    }
}
```

Configure the issuer in `application.properties`:

```properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://identity.example.com/issuer
```

Replace the example issuer URL with your real identity provider's issuer. The provider must support the required discovery or key configuration.

This setup validates incoming JWTs; it does not itself provide a login endpoint or issue tokens. Token issuance is typically handled by an identity provider or a separately designed authentication service.

For a stateless bearer-token API, you would also commonly configure stateless session management and decide on CSRF protection based on how credentials are transmitted.

## 6. Method-level security

In addition to URL-based rules, Spring Security can protect individual service methods.

Enable method security:

```java
@Configuration
@EnableMethodSecurity
public class MethodSecurityConfig {
}
```

Apply a role restriction:

```java
@Service
public class AdminService {

    @PreAuthorize("hasRole('ADMIN')")
    public String getSensitiveData() {
        return "Sensitive data";
    }
}
```

`@PreAuthorize` evaluates authorization before method execution. This is useful when access restrictions should apply regardless of which controller invokes the service.

Method security also operates through Spring infrastructure, so proxy limitations such as self-invocation should be considered.

## 7. Other important Spring Security features

| Feature | Purpose |
|---|---|
| CSRF protection | Helps prevent cross-site request forgery in browser-based applications using automatically attached credentials. |
| CORS | Controls which browser origins may make cross-origin requests; it is not authentication. |
| Password hashing | Protects stored passwords using algorithms such as BCrypt or Argon2. |
| OAuth 2.0 / OpenID Connect | Supports delegated authorization and identity-provider-based authentication. |
| JWT resource server | Validates bearer access tokens and derives authorities from trusted claims. |
| Method security | Restricts access to service methods with annotations such as `@PreAuthorize`. |
| Security headers | Helps defend against certain browser-based attacks. |

**Important:** Do not disable CSRF indiscriminately. Disabling it is often appropriate for a carefully designed API that accepts only stateless bearer tokens in authorization headers, but browser sessions or cookies require a different assessment.

## 8. Common HTTP status codes

| Status | Meaning |
|---|---|
| `200 OK` | Request succeeded. |
| `401 Unauthorized` | Authentication is missing or invalid. |
| `403 Forbidden` | The caller is authenticated but lacks permission, or access is otherwise forbidden by the security configuration. |

Despite its name, HTTP `401 Unauthorized` usually indicates an authentication problem, while `403 Forbidden` typically indicates an authorization problem.

## 9. Interview-ready answer

**Question: How do you handle security in Spring Boot?**

I use Spring Security to implement authentication and authorization. Authentication verifies the user's identity, while authorization determines which endpoints or operations the user can access.

In modern Spring Boot applications, I configure a `SecurityFilterChain` bean with rules such as `permitAll()`, `authenticated()`, `hasRole()`, and `hasAuthority()`. For username/password authentication, I can use `UserDetailsService` and a suitable `PasswordEncoder`. For REST APIs and microservices, I commonly use OAuth 2.0 resource-server support to validate JWT access tokens issued by a trusted identity provider.

I can also apply method-level security using `@EnableMethodSecurity` and `@PreAuthorize`. In production, I pay attention to HTTPS, password hashing, CSRF and CORS configuration, token validation, least-privilege authorization, secret management, and keeping dependencies updated.

## Quick revision

- **Framework:** Spring Security
- **Authentication:** Who are you?
- **Authorization:** What can you access?
- **Modern configuration:** `SecurityFilterChain`
- **User loading:** `UserDetailsService`
- **Password hashing:** `PasswordEncoder`
- **URL security:** `authorizeHttpRequests()`
- **Method security:** `@EnableMethodSecurity`, `@PreAuthorize`
- **JWT APIs:** `oauth2ResourceServer().jwt()`
- **401 vs 403:** Authentication failure vs access denied.


---


## How Does a Filter Work in Spring Boot?

A **Filter** in Spring Boot is a web component that intercepts HTTP requests and responses before they reach the servlet and after the servlet processes them. Filters are part of the **Servlet API**, rather than the Spring MVC controller layer.

Filters are commonly used for request logging, security-related processing, CORS, request/response wrapping, and other cross-cutting concerns.

### 1. How Does a Filter Work?

A typical request passes through the following flow:

```text
Client Request
      |
      v
Filter 1 (Pre-processing)
      |
      v
Filter 2 (Pre-processing)
      |
      v
DispatcherServlet
      |
      v
Controller
      |
      v
Service Layer
      |
      v
Response travels back
      |
      v
Filter 2 (Post-processing)
      |
      v
Filter 1 (Post-processing)
      |
      v
Client Response
```

The exact flow depends on the configured filter chain and whether a filter continues processing.

**Important:** A filter can stop the request by not calling `filterChain.doFilter()`. In that case, the downstream filters and controller are not invoked through that chain.

### 2. Filter Lifecycle Methods

A filter typically implements the `jakarta.servlet.Filter` interface in Spring Boot 3.x and later. Spring Boot 2.x uses `javax.servlet.Filter`.

| Method | Purpose |
|---|---|
| `init(FilterConfig)` | Initializes the filter when it is placed into service. |
| `doFilter(request, response, chain)` | Processes each applicable request and response. |
| `destroy()` | Releases resources when the filter is taken out of service. |

### 3. Implementing a Custom Filter

The following example logs the HTTP method, URI, and request-processing time.

```java
package com.example.demo.filter;

import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class RequestLoggingFilter implements Filter {

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain filterChain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest =
                (HttpServletRequest) request;

        long startTime = System.currentTimeMillis();

        System.out.println(
                "Incoming request: "
                + httpRequest.getMethod()
                + " "
                + httpRequest.getRequestURI()
        );

        try {
            // Continue to the next filter or servlet.
            filterChain.doFilter(request, response);
        } finally {
            long duration =
                    System.currentTimeMillis() - startTime;

            System.out.println(
                    "Request completed in "
                    + duration + " ms"
            );
        }
    }
}
```

Because the class is annotated with `@Component`, Spring Boot can register it as a servlet filter in a web application.

**How it works:**

1. The filter receives the incoming request.
2. It logs the HTTP method and URI.
3. It records the start time.
4. `filterChain.doFilter()` passes control to the next filter or servlet.
5. Once downstream processing returns, the `finally` block logs the elapsed time.

The `finally` block runs even if downstream processing throws an exception. However, the filter should not assume that every error has already been converted into the final HTTP response when its post-processing code executes.

### 4. Registering a Filter with `FilterRegistrationBean`

Use `FilterRegistrationBean` when you need explicit control over URL patterns, filter order, or registration settings.

```java
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<RequestLoggingFilter>
            requestLoggingFilterRegistration() {

        FilterRegistrationBean<RequestLoggingFilter> registration =
                new FilterRegistrationBean<>();

        registration.setFilter(new RequestLoggingFilter());
        registration.addUrlPatterns("/api/*");
        registration.setOrder(1);

        return registration;
    }
}
```

**Important:** Do not register the same filter twice. If the filter is already registered through `@Component`, explicitly registering another instance can result in duplicate execution. For this configuration, remove `@Component` from the filter class and register it only through the bean above.

### 5. Registering a Filter with `@WebFilter`

You can also use the Servlet API's `@WebFilter` annotation.

```java
@WebFilter(urlPatterns = "/api/*")
public class ApiFilter implements Filter {

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain)
            throws IOException, ServletException {

        System.out.println("API request received");

        chain.doFilter(request, response);
    }
}
```

Enable servlet component scanning with `@ServletComponentScan` on your Spring Boot application class:

```java
@SpringBootApplication
@ServletComponentScan
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

For Spring Boot applications, `FilterRegistrationBean` is often convenient when you need Spring-managed configuration and explicit ordering.

### 6. Common Uses of Filters

| Use case | Example |
|---|---|
| Logging and auditing | Record request methods, paths, and durations. |
| Security | Enforce request-level checks or integrate security infrastructure. |
| CORS | Add cross-origin response handling, preferably using Spring Security or Spring MVC configuration where appropriate. |
| Request/response wrapping | Wrap requests or responses to inspect or transform data. |
| Compression | Apply response compression where supported and configured. |
| Correlation IDs | Propagate request IDs for distributed tracing and log correlation. |

**Security note:** In Spring Security applications, authentication and authorization are generally handled by the Spring Security filter chain. Avoid implementing custom authentication from scratch unless there is a specific requirement.

### 7. Filter vs. Interceptor

This is a common Spring Boot interview question.

| Feature | Filter | HandlerInterceptor |
|---|---|---|
| API | Servlet API | Spring MVC |
| Execution point | Around servlet processing | Around handler/controller execution |
| Before request handling | `doFilter()` before `chain.doFilter()` | `preHandle()` |
| After controller execution | After `chain.doFilter()` returns | `postHandle()` after successful handler execution |
| After request completion | Can use `finally` around the chain | `afterCompletion()` for MVC request completion |
| Access to selected controller method | Not directly through MVC handler metadata | Can access the selected handler in MVC callbacks |
| Typical use | Request-level concerns and security-chain integration | Controller/handler-specific concerns |

**Remember:** A filter can run before the `DispatcherServlet`. An interceptor operates inside Spring MVC after handler mapping has selected a handler.

### 8. Filter vs. Spring Security Filter Chain

Spring Security itself relies on servlet filters to process security-related requests.

A typical simplified flow is:

```text
HTTP Request
     |
     v
Servlet Container Filters
     |
     v
DelegatingFilterProxy
     |
     v
Spring Security Filter Chain
     |
     v
DispatcherServlet
     |
     v
Controller
```

The actual ordering depends on application configuration. The Spring Security chain can authenticate requests, enforce authorization rules, and reject requests before a controller is called.

### Interview Answer

"A filter in Spring Boot is a Servlet API component used to intercept HTTP requests and responses. It implements the `Filter` interface, primarily using the `doFilter()` method. Before calling `filterChain.doFilter()`, we can perform pre-processing such as logging or request validation. After the chain returns, we can perform post-processing, such as recording execution time.

Filters can be registered using `FilterRegistrationBean`, `@WebFilter` with servlet component scanning, or Spring bean registration. Common use cases include logging, correlation IDs, request wrapping, and security-related processing. A filter differs from a `HandlerInterceptor` because a filter operates at the servlet level, while an interceptor works within Spring MVC around controller execution."

### Quick Revision

- **Filter interface:** `jakarta.servlet.Filter` in Spring Boot 3+.
- **Main method:** `doFilter()`.
- **Continue processing:** `filterChain.doFilter(request, response)`.
- **Stop downstream processing:** Do not invoke `filterChain.doFilter()`.
- **Explicit registration and ordering:** `FilterRegistrationBean`.
- **Annotation-based registration:** `@WebFilter` with `@ServletComponentScan`.
- **Controller-specific request handling:** Consider `HandlerInterceptor`.
- **Authentication and authorization:** Prefer Spring Security's filter chain.


---


# What Is a JWT Token, and How Does Spring Boot Fetch Its Information?

## 1. What is JWT?

**JWT (JSON Web Token)** is a compact, URL-safe format for transferring claims between parties. It is commonly used for authentication and authorization in REST APIs and microservices.

After a user authenticates, an identity provider or authentication service can issue a JWT access token. The client sends that token with subsequent requests, and Spring Security validates it before allowing access to protected endpoints.

**Important:** A JWT is usually signed, not encrypted. Its payload can generally be decoded and read by anyone who possesses the token. Never put passwords, secrets, or other sensitive information in an ordinary signed JWT.

## 2. Structure of a JWT

A JWT commonly contains three parts separated by dots:

```text
xxxxx.yyyyy.zzzzz
  │      │      │
Header Payload Signature
```

For example, a token might represent these claims:

**Header**
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload**
```json
{
  "sub": "danish",
  "roles": ["USER"],
  "iat": 1790000000,
  "exp": 1790003600
}
```

**Signature**

The signature is calculated using the encoded header, encoded payload, and the signing key.

For HMAC-SHA256, the conceptual calculation is:

```text
HMAC-SHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secretKey
)
```

The actual JWT is:

```text
Base64Url(header).Base64Url(payload).Base64Url(signature)
```

The JSON values above are illustrative, and the timestamps are examples.

### What does each part do?

| Part | Purpose |
|---|---|
| Header | Identifies the token type and signing algorithm. |
| Payload | Contains claims such as subject, expiration, and possibly roles. |
| Signature | Allows the server to verify that the signed content has not been modified and was signed by a holder of the relevant key. |

Common claims include:

- `sub` — subject, often a user identifier.
- `iss` — issuer of the token.
- `aud` — intended audience.
- `iat` — time the token was issued.
- `exp` — token expiration time.
- `scope` or roles — permissions, depending on the issuer's token format.

## 3. How does Spring Boot receive and fetch JWT information?

Spring Boot does not automatically fetch user details from a JWT merely because the application uses JWT. With Spring Security's OAuth2 Resource Server support, the security filter chain receives the bearer token, validates it, and converts its trusted claims into an authenticated security context.

The typical flow is:

<box border radius="lg" padding={3} gap={2} align="center">
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **1. User logs in**

    <text size="sm">Credentials are sent to an authentication service or identity provider.</text>
  </box>
  <icon name="arrow-down" color="secondary" size="lg" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **2. JWT is issued**

    <text size="sm">The issuer signs a token containing claims and an expiration time.</text>
  </box>
  <icon name="arrow-down" color="secondary" size="lg" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **3. Client sends the token**

    <text size="sm">`Authorization: Bearer &lt;JWT&gt;`</text>
  </box>
  <icon name="arrow-down" color="secondary" size="lg" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **4. Spring Security filter chain**

    <text size="sm">The bearer-token filter extracts the token from the request.</text>
  </box>
  <icon name="arrow-down" color="secondary" size="lg" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **5. JWT validation**

    <text size="sm">Checks the signature, expiration and configured issuer/audience requirements.</text>
  </box>
  <icon name="arrow-down" color="secondary" size="lg" />
  <box background="surface-secondary" radius="md" padding={3} width="100%" align="center">
    **6. Authentication object created**

    <text size="sm">Spring creates an authenticated principal with claims and granted authorities.</text>
  </box>
  <icon name="arrow-down" color="secondary" size="lg" />
  <box border radius="md" padding={3} width="100%" align="center">
    **7. Authorization and controller**

    <text size="sm">Access rules are evaluated before the request reaches the protected resource.</text>
  </box>
</box>

If validation fails, Spring Security normally rejects the request with `401 Unauthorized`. If the token is valid but the caller lacks the required permission, the result is generally `403 Forbidden`.

## 4. How to implement JWT validation in modern Spring Boot

For a resource server, use Spring Security's built-in JWT support instead of writing your own token-parsing filter unless you have a specific requirement.

### Step 1: Add the dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

This brings in the resource-server support and the necessary Spring Security integration.

### Step 2: Configure the issuer

In `application.properties`:

```properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://identity.example.com/issuer
```

Replace this placeholder with your actual identity provider's issuer URI. Spring can use the provider's metadata to discover the public signing keys and validate tokens, where the provider supports this configuration.

For an HMAC-signed token, the resource server instead needs the correct shared secret configured securely. Never expose that secret to clients or commit it to source control.

### Step 3: Configure the security filter chain

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 ->
                oauth2.jwt(Customizer.withDefaults())
            );

        return http.build();
    }
}
```

This configuration:

- Allows public endpoints without a token.
- Requires the `ADMIN` role for `/admin/**`.
- Requires authentication for other endpoints.
- Enables JWT bearer-token validation.

The issuer and token's claims must be configured consistently. By default, roles are not necessarily mapped from a JWT claim named `roles`; Spring Security's default JWT authority converter commonly maps `scope` or `scp` claims to authorities prefixed with `SCOPE_`. If your issuer uses a `roles` claim, configure an appropriate authority converter.

## 5. How to read JWT information inside a controller

Once Spring Security has validated the token, you can access its claims using `Jwt` or access the current authenticated principal through the security context.

### Option A: Use `@AuthenticationPrincipal`

```java
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @GetMapping("/me")
    public String getCurrentUser(
            @AuthenticationPrincipal Jwt jwt) {

        String username = jwt.getSubject();
        String issuer = jwt.getIssuer().toString();

        return "User: " + username + ", Issuer: " + issuer;
    }
}
```

Here, Spring supplies the validated JWT principal to the controller. The subject comes from the token's `sub` claim.

### Option B: Use `SecurityContextHolder`

```java
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;

Authentication authentication =
        SecurityContextHolder.getContext().getAuthentication();

String username = authentication.getName();
```

This retrieves the current authentication object for the executing request.

For resource-server JWT authentication, the principal is commonly a `Jwt`, and the authentication is commonly a `JwtAuthenticationToken`.

**Key distinction:** Reading a claim from a JWT is not the same as fetching the user from a database. Spring can authenticate a request using validated token claims alone. If your business logic needs current user information, account status, or data that changes frequently, you may need an additional database or identity-provider lookup.

## 6. Does Spring Boot fetch the user from the database for every request?

**Not necessarily.** It depends on the authentication design.

| Approach | What happens per request? |
|---|---|
| JWT resource server | Validates the token and builds authentication from its claims. A database lookup is not inherently required. |
| JWT plus `UserDetailsService` lookup | Application can load current user details, for example to check account status or obtain current authorities. |
| Session-based authentication | The server uses the session's security context, with storage and retrieval depending on the session configuration. |

A JWT resource server may need to retrieve signing keys, but providers commonly publish public keys through a JWKS endpoint. Those keys are typically cached rather than downloaded for every request.

A valid signature alone is not sufficient for every security policy. The application should also validate expiration and relevant issuer/audience requirements, and it should consider whether permissions or account status need to be refreshed before the token expires.

## 7. How are roles and permissions extracted?

Suppose the JWT contains:

```json
{
  "sub": "danish",
  "roles": ["ADMIN"]
}
```

After validation, Spring can read the claim. However, to use `hasRole("ADMIN")`, you must map that claim to a granted authority named `ROLE_ADMIN`, or use a suitable custom converter.

For example, a custom converter can map a trusted `roles` claim to Spring authorities. The important distinction is:

- JWT claim: `roles: ["ADMIN"]`
- Spring authority expected by `hasRole("ADMIN")`: `ROLE_ADMIN`

Never trust a role merely because a client sends a JSON value. The token must be cryptographically validated, and the issuer must be trusted to assign the claim.

## 8. Is JWT stateless?

JWT authentication is often described as **stateless** because the server does not need to maintain a session for every client. The token carries claims that can be validated independently on each request.

However, stateless does not mean that no state is ever needed. Applications may still maintain:

- Refresh-token records.
- Token revocation or deny lists.
- User account status.
- Current roles and permissions.
- Audit records and login history.

A JWT also cannot ordinarily be revoked simply by deleting it from the client. Short access-token lifetimes, refresh-token rotation, and server-side revocation strategies can help address this limitation.

## 9. Security best practices

1. Use HTTPS for all production traffic.
2. Validate signature, expiration, issuer, and audience where applicable.
3. Use a trusted issuer and an approved algorithm; do not accept arbitrary algorithms from an untrusted header.
4. Keep signing keys and shared secrets outside source control.
5. Keep access tokens short-lived and design refresh-token handling carefully.
6. Avoid putting confidential data in the payload.
7. Apply authorization rules at the API and, where appropriate, service-method levels.
8. Consider how to revoke tokens and respond to account or permission changes.
9. If tokens are stored in cookies, assess CSRF risks; if using browser storage, assess token theft through cross-site scripting.
10. Do not log complete access tokens.

## 10. Interview-ready answer

**Question: What is a JWT token, and how does Spring Boot fetch its information?**

JWT stands for JSON Web Token. It contains a header, payload, and signature. The payload carries claims such as the user ID, issuer, expiration, and possibly roles. The signature allows the server to verify the token's integrity and authenticity.

When a client sends a JWT in the `Authorization: Bearer` header, Spring Security's bearer-token filter extracts it. The JWT decoder validates the signature and configured claims, then Spring creates an authentication object and stores it in the security context for the request. Authorization rules determine whether the user can access the endpoint.

Spring does not necessarily query the database for every request because validated claims can provide the identity and authorities. If the application needs current user information or updated permissions, it can perform an additional database or identity-provider lookup.

In modern Spring Boot applications, I prefer Spring Security's OAuth2 Resource Server support for validating JWTs rather than manually parsing tokens in a custom filter.

## Quick revision

- **JWT structure:** Header + Payload + Signature.
- **Client sends:** `Authorization: Bearer <token>`.
- **Token extraction:** Spring Security bearer-token filter.
- **Validation:** JWT decoder and configured validators.
- **Authentication:** Principal and authorities placed in the security context.
- **Authorization:** `hasRole()`, `hasAuthority()`, `@PreAuthorize`.
- **Database lookup:** Optional, based on application requirements.
- **Important limitation:** Signed JWT payloads are readable unless separately encrypted.
- **Modern approach:** `SecurityFilterChain` + OAuth2 Resource Server JWT support.


---


# How Does OAuth 2.0 Work?

## 1. What is OAuth 2.0?

**OAuth 2.0 is an authorization framework** that allows an application to access protected resources on behalf of a user without requiring the user to share their password with that application.

For example, suppose you build a fitness application that needs to access a user's Google Calendar. Instead of collecting the user's Google password, your application redirects the user to Google's authorization service. The user grants permission, and your application obtains an access token that can be used to access the permitted resources.

OAuth 2.0 is primarily about **authorization**, not authentication. For user authentication and login, applications commonly use **OpenID Connect (OIDC)**, an identity layer built on top of OAuth 2.0.

## 2. Important OAuth 2.0 terminology

| Term | Meaning |
|---|---|
| Resource Owner | The user who owns or controls the protected data. |
| Client | The application requesting access to the data. |
| Authorization Server | Authenticates the user, obtains consent when required, and issues tokens. |
| Resource Server | The API that hosts protected resources and validates access tokens. |
| Access Token | A credential used to access protected resources within its permitted scope. |
| Refresh Token | A credential that can be used to obtain a new access token when supported by the authorization server. |
| Scope | Defines the permissions requested or granted, such as reading a user's profile. |
| Client ID | An identifier for the application registered with the authorization server. |

For example, in a Spring Boot application integrating with Google:

- Google acts as the authorization server.
- Google's protected API acts as the resource server.
- Your Spring Boot application is the client.
- The user authorizes the requested access.

The same organization may operate both the authorization server and the resource server.

## 3. How does OAuth 2.0 work internally?

A common flow for a web application is the **Authorization Code flow**.

<box border radius="lg" padding={3} gap={3}>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **1. User opens the application**

    <text size="sm">The client needs permission to access a protected resource.</text>
  </box>
  <box align="center" gap={1}>
    <icon name="arrow-down" size="lg" color="secondary" />
    <text size="xs" color="secondary">Redirect</text>
  </box>
  <box background="surface" border radius="md" padding={3} align="center">
    **2. Authorization server**

    <text size="sm">The user signs in and approves the requested access, if consent is required.</text>
  </box>
  <box align="center" gap={1}>
    <icon name="arrow-down" size="lg" color="secondary" />
    <text size="xs" color="secondary">Authorization code returned to callback</text>
  </box>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **3. Client exchanges the code**

    <text size="sm">The client sends the code to the token endpoint. A confidential server-side client authenticates as required and verifies PKCE when used.</text>
  </box>
  <box align="center" gap={1}>
    <icon name="arrow-down" size="lg" color="secondary" />
    <text size="xs" color="secondary">Access token issued</text>
  </box>
  <box background="surface" border radius="md" padding={3} align="center">
    **4. Resource server**

    <text size="sm">The client sends the access token to the API.</text>
  </box>
  <box align="center" gap={1}>
    <icon name="arrow-down" size="lg" color="secondary" />
    <text size="xs" color="secondary">Token validated and scope checked</text>
  </box>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **5. Protected resource returned**

    <text size="sm">The API returns the permitted data if the request is authorized.</text>
  </box>
</box>

### Step-by-step explanation

1. The user clicks **Connect account** in your application.
2. Your application redirects the user's browser to the authorization server with parameters such as `client_id`, `redirect_uri`, `scope`, `response_type=code`, `state`, and, when using PKCE, a `code_challenge`.
3. The authorization server authenticates the user and obtains consent where required.
4. The authorization server redirects the browser back to the registered callback URL with an authorization code.
5. The client exchanges that code at the token endpoint. PKCE helps ensure that only the party holding the corresponding verifier can redeem the code.
6. The authorization server issues an access token if the exchange succeeds.
7. The client sends the access token to the resource server, typically using `Authorization: Bearer <access-token>`.
8. The resource server validates the token and checks that it grants the required access before returning data.

**Important:** In the Authorization Code flow, the browser normally receives an authorization code, not the access token directly. The client exchanges the code for the token. The exact handling depends on the client architecture.

## 4. What is the difference between OAuth 2.0 and JWT?

These technologies are related but serve different purposes.

| OAuth 2.0 | JWT |
|---|---|
| Authorization framework. | Token format. |
| Defines flows for obtaining and using access tokens. | Represents claims in a compact format. |
| Can use opaque access tokens or JWT access tokens. | Can be used as an access token or for other token purposes. |
| Defines concepts such as scopes, clients, and authorization servers. | Defines a way to encode claims and protect token integrity, commonly through a signature. |

OAuth 2.0 does **not** require JWT. An access token can be an opaque random string that the resource server validates through introspection or another server-side mechanism.

Similarly, having a JWT does not automatically mean an application is using OAuth 2.0.

## 5. OAuth 2.0 grant types you should know

| Grant / flow | Typical use |
|---|---|
| Authorization Code + PKCE | User-facing web, mobile, and single-page applications. |
| Client Credentials | Service-to-service access where the client acts on its own behalf rather than on behalf of a user. |
| Refresh Token | Obtain a replacement access token when the authorization server permits it. |
| Device Authorization Grant | Devices with limited input capabilities, such as some smart TVs. |

The **Resource Owner Password Credentials grant** is not recommended for modern applications. The Implicit flow is also generally avoided; use Authorization Code with PKCE instead.

## 6. How to implement OAuth 2.0 in Spring Boot

Spring Security provides different integrations depending on whether your application is a client, a resource server, or both.

### A. Spring Boot as an OAuth2 client

Use this when your application needs to redirect users to an external identity provider or access a third-party API on their behalf.

Add the dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

Configure the client in `application.yml`:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          myprovider:
            client-id: ${OAUTH_CLIENT_ID}
            client-secret: ${OAUTH_CLIENT_SECRET}
            scope:
              - profile
        provider:
          myprovider:
            authorization-uri: https://identity.example.com/oauth2/authorize
            token-uri: https://identity.example.com/oauth2/token
```

These URLs and scopes are illustrative. Use the actual endpoints and supported scopes supplied by your identity provider. A confidential server-side application should store its client secret securely.

With the appropriate OAuth2 client configuration, Spring Security can manage the authorization-code login flow and token exchange. For a standard OpenID Connect login, configure the provider's OIDC discovery or issuer details and the `openid` scope as required.

### B. Spring Boot as a resource server

Use this when your application exposes REST APIs and needs to validate access tokens from an authorization server.

Add the dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

Configure the trusted issuer:

```properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://identity.example.com/issuer
```

Then configure API authorization:

```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/api/**").authenticated()
                .anyRequest().denyAll()
            )
            .oauth2ResourceServer(oauth2 ->
                oauth2.jwt(Customizer.withDefaults())
            );

        return http.build();
    }
}
```

Import `Customizer`, `HttpSecurity`, `SecurityFilterChain`, `Bean`, and `Configuration` from their respective Spring packages.

In this setup, Spring Security extracts the bearer token, validates it using the configured JWT decoder and validators, creates an authentication object, and evaluates the endpoint's authorization rules. It does not issue the token itself.

If the authorization server issues opaque tokens rather than JWTs, Spring Security can instead be configured to use opaque-token introspection.

## 7. OAuth 2.0 vs OpenID Connect (OIDC)

This is a frequent interview question.

- **OAuth 2.0:** Allows an application to obtain authorized access to protected resources.
- **OpenID Connect:** Adds a standardized identity and login layer on top of OAuth 2.0, including an ID token and standardized user identity claims.

For example, when a user clicks **Sign in with Google**, the application typically uses OpenID Connect for authentication. If it also needs permission to access a Google API, OAuth 2.0 access-token authorization is involved.

An **ID token** is intended to convey authentication information to the client. An **access token** is intended to authorize access to a resource server. They are not interchangeable.

## 8. Security best practices

1. Use Authorization Code with PKCE for interactive authorization flows.
2. Validate `state` and use the appropriate OIDC `nonce` checks where applicable.
3. Register exact redirect URIs and avoid open redirects.
4. Store client secrets and refresh tokens securely.
5. Request only the scopes the application actually needs.
6. Validate tokens at the resource server; do not trust a token merely because it can be decoded.
7. Use HTTPS and appropriate token lifetimes.
8. Avoid putting access tokens in URLs or logging them.
9. Design token revocation, refresh-token rotation, and permission changes deliberately.

## 9. Interview-ready answer

**Question: How does OAuth 2.0 work?**

OAuth 2.0 is an authorization framework that allows an application to access protected resources without collecting the user's password directly. It involves a client, a resource owner, an authorization server, and a resource server.

In the common Authorization Code flow, the client redirects the user to the authorization server. After authentication and any required consent, the authorization server redirects the user back with an authorization code. The client exchanges the code for an access token and uses that token to call the resource server. The resource server validates the token and checks the granted permissions before returning the protected resource.

In Spring Boot, I use Spring Security OAuth2 Client when the application needs to obtain tokens or sign users in, and OAuth2 Resource Server when the application needs to validate bearer tokens for protected APIs. OAuth 2.0 is for authorization; OpenID Connect builds on it to provide standardized user authentication.

## Quick revision

- **OAuth 2.0:** Authorization framework.
- **Client:** Application requesting access.
- **Authorization server:** Issues tokens.
- **Resource server:** Protects APIs and validates tokens.
- **Access token:** Used to call protected APIs.
- **Authorization Code + PKCE:** Recommended interactive flow.
- **Client Credentials:** Service-to-service access.
- **JWT:** One possible token format, not a requirement of OAuth 2.0.
- **OIDC:** Adds authentication and ID tokens on top of OAuth 2.0.
- **Spring Boot:** OAuth2 Client for obtaining tokens; Resource Server for validating them.

---


# How Do You Ensure a JWT Token Has Not Been Tampered With?

## 1. The basic concept

A JWT's integrity is verified by **validating its cryptographic signature**.

A signed JWT contains three parts:

```text
Header.Payload.Signature
```

Suppose a JWT contains the following payload:

```json
{
  "sub": "danish",
  "role": "USER",
  "exp": 1790003600
}
```

An attacker modifies the payload to change the role:

```json
{
  "sub": "danish",
  "role": "ADMIN",
  "exp": 1790003600
}
```

Because the payload has changed, the original signature no longer matches the modified token. Signature validation fails, provided the attacker cannot produce a valid signature with the trusted signing key.

**Important:** Base64URL encoding is not encryption. Anyone who obtains a normal signed JWT can decode and read its payload. The signature protects integrity, not confidentiality.

## 2. How signature validation works internally

For an HMAC-SHA256-signed JWT, the issuer calculates the signature using the encoded header, encoded payload, and a secret key.

Conceptually:

```text
signature = HMAC-SHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secretKey
)
```

When the server receives the token, it:

1. Splits the JWT into its three parts.
2. Reads the header to identify the declared algorithm.
3. Checks that the algorithm is permitted by the server's trusted configuration.
4. Recomputes or cryptographically verifies the signature using the trusted key.
5. Compares the computed signature with the supplied signature using the appropriate cryptographic verification procedure.
6. Rejects the token if verification fails.
7. Validates claims such as expiration, issuer, and audience according to the application's configuration.

The server must **not blindly trust the algorithm specified in the token header**. It must enforce an appropriate allowlist and use a trusted key for the expected algorithm.

## 3. HMAC vs RSA signatures

| HMAC-SHA256 (HS256) | RSA-SHA256 (RS256) |
|---|---|
| Uses the same secret to sign and verify. | Uses a private key to sign and a corresponding public key to verify. |
| The secret must remain confidential across all parties that verify tokens. | Resource servers can verify tokens without possessing the private signing key. |
| Useful when the issuer and verifier securely share a key. | Useful when many services need to verify tokens issued by a central identity provider. |

In microservice architectures, asymmetric signing can be beneficial because resource servers only need the issuer's public key. A compromised verifier does not automatically gain the ability to issue new tokens.

## 4. How does Spring Boot detect a tampered JWT?

Spring Security can validate JWTs automatically using OAuth2 Resource Server support. You do not need to manually split the token and recreate its signature in your controller.

### Step 1: Add the dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

### Step 2: Configure the trusted issuer

In `application.properties`:

```properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://identity.example.com/issuer
```

Replace the example URI with your trusted identity provider's issuer. The issuer must publish compatible metadata and signing-key information for automatic discovery to work.

### Step 3: Configure the security filter chain

```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 ->
                oauth2.jwt(Customizer.withDefaults())
            );

        return http.build();
    }
}
```

With a correctly configured JWT decoder and validators, Spring Security checks the token's signature and configured claims before establishing authentication.

If someone changes the payload from `USER` to `ADMIN` without the signing key, validation fails and the protected request is rejected, normally with `401 Unauthorized`.

For production, ensure the configuration validates the expected issuer and audience, enforces permitted algorithms, and handles signing-key rotation appropriately. Issuer validation is commonly configured from `issuer-uri`; audience validation may need additional configuration.

## 5. Does a valid signature guarantee that the token is safe?

No. A valid signature establishes that the signed content has not been modified since it was signed by a holder of the corresponding signing key. It does not, by itself, guarantee that the token should still be accepted.

You should also validate:

- **Expiration (`exp`):** Has the token expired?
- **Issuer (`iss`):** Was it issued by the expected identity provider?
- **Audience (`aud`):** Is the token intended for this API?
- **Not-before (`nbf`):** Is the token valid yet, if this claim is present?
- **Algorithm:** Is the signing algorithm explicitly permitted?
- **Authorization claims:** Are the granted roles or scopes appropriate for the requested operation?

A token may be correctly signed but still be expired, issued for another service, or grant insufficient permissions.

## 6. How do you prevent JWT theft or misuse?

Signature validation protects against unauthorized modifications, but it does not stop someone from stealing a valid token and replaying it.

Follow these practices:

1. Use HTTPS/TLS for all token transmission.
2. Store signing keys and secrets in a secure secret manager.
3. Use short-lived access tokens where appropriate.
4. Rotate keys safely and support trusted signing-key updates.
5. Avoid logging complete tokens or exposing them in URLs.
6. Design revocation or account-disablement behavior where required.
7. Validate authorization on every protected request.
8. Use appropriate browser storage and CSRF protections for the chosen client architecture.

## 7. Interview-ready answer

**Question: How do you ensure that a JWT token has not been tampered with?**

I verify the JWT's cryptographic signature using a trusted key and an explicitly permitted signing algorithm. The signature is calculated over the encoded header and payload, so modifying either invalidates the original signature unless the attacker can generate a valid replacement.

In Spring Boot, I normally use Spring Security's OAuth2 Resource Server support rather than implementing signature validation manually. Spring validates the token through its JWT decoder and configured validators before creating the authentication object. I also validate claims such as expiration, issuer, and audience, and ensure that the signing keys remain secure.

Finally, I use HTTPS to protect tokens in transit and remember that signature validation prevents unauthorized modification, but does not by itself prevent token theft or replay.

## Quick revision

- **Integrity:** Verify the JWT signature.
- **HS256:** Shared secret for signing and verification.
- **RS256:** Private key signs; public key verifies.
- **Tampered payload:** Signature verification fails.
- **Spring Boot:** Configure Spring Security OAuth2 Resource Server.
- **Additional validation:** Expiration, issuer, audience, algorithm.
- **HTTPS:** Protects token transmission.
- **Remember:** A valid signature does not mean the token is unexpired, authorized for this API, or impossible to steal.


