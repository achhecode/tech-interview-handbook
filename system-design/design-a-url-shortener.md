---
title: Design a URL Shortener
parent: System Design
nav_order: 3
---

# Design a URL Shortener
{: .no_toc }

High-Level Design (HLD), Low-Level Design (LLD), Java implementation, database schema, caching, and scalability.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }

1. TOC
{:toc}
</details>

---

## 1. What is a URL Shortener?

A **URL shortener** converts a long URL into a shorter URL that redirects users to the original destination.

For example:

```text
Original URL:
https://www.example.com/products/electronics/smartphones?category=mobile&id=12345

Short URL:
https://sho.rt/aB3xY9
```

When a user opens the short URL, the system retrieves the original URL and redirects the user to it.

Popular examples include Bitly and TinyURL.

### Why do we need a URL Shortener?

- **Short links:** Make long URLs easier to share.
- **Better user experience:** Create compact links for messages, emails, and social media.
- **Analytics:** Track clicks, referrers, and usage patterns.
- **Custom aliases:** Allow users to create memorable links.
- **Link management:** Support expiration, deactivation, and destination updates.

### Core operations

A URL shortener primarily supports two operations:

1. **Create a short URL:** Accept a long URL and generate a unique short code.
2. **Redirect:** Resolve the short code and redirect the user to the original URL.

Optional features include custom aliases, expiration, link deactivation, analytics, and authenticated link management.

---

## 2. Requirements and Design Scope

### 2.1 Functional requirements

- Generate a unique short URL for a valid long URL.
- Redirect a short URL to its original destination.
- Support custom aliases when available.
- Support URL expiration.
- Allow authorized users to deactivate or update their links.
- Track click events and basic analytics.
- Reject invalid URLs and unavailable aliases.

### 2.2 Non-functional requirements

- **High availability:** Redirects should remain available.
- **Low latency:** Redirects should complete quickly.
- **Scalability:** Handle large numbers of URLs and clicks.
- **Uniqueness:** Never map one active short code to two different destinations.
- **Durability:** Persist URL mappings.
- **Security:** Prevent abuse, malicious redirects, and unauthorized link modification.
- **Consistency:** Ensure newly created mappings can be resolved correctly.

### 2.3 Assumptions

For this design, assume:

- Short links use a domain such as `sho.rt`.
- Each generated code contains alphanumeric characters.
- URLs are stored in a persistent database.
- Redis caches frequently accessed mappings.
- Redirect traffic is significantly higher than URL-creation traffic.
- Analytics can be processed asynchronously.

These assumptions guide the architecture; production capacity and storage requirements must be calculated from actual traffic estimates.

---

## 3. Capacity Estimation

Capacity estimation helps determine database size, storage requirements, cache capacity, and traffic-handling needs.

Assume the following illustrative workload:

```text
New URLs per day       = 1 million
Redirects per day      = 100 million
Retention period       = 5 years
Average stored record  = 500 bytes
```

These are example assumptions, not measured production statistics.

### 3.1 Write throughput

```text
URLs per day = 1,000,000

Average writes per second
= 1,000,000 / 86,400
≈ 11.6 writes/second
```

Actual peak traffic may be several times higher than the daily average.

### 3.2 Read throughput

```text
Redirects per day = 100,000,000

Average reads per second
= 100,000,000 / 86,400
≈ 1,157 reads/second
```

Redirect traffic is approximately 100 times the creation traffic in this example.

This read-heavy workload makes caching especially valuable.

### 3.3 Storage estimation

```text
URLs over 5 years
= 1,000,000 × 365 × 5
= 1.825 billion URLs
```

At 500 bytes per record:

```text
Storage
= 1.825 billion × 500 bytes
≈ 912.5 GB
```

This is raw record storage only. Indexes, replication, database overhead, backups, and analytics data increase the total requirement.

### 3.4 Short-code length

For a Base62 alphabet containing 62 characters, the number of possible codes of length \(L\) is:

\[
N = 62^L
\]

For seven characters:

\[
62^7 = 3,521,614,606,208
\]

Seven Base62 characters provide approximately 3.52 trillion possible codes.

This is the size of the code space, not the number of codes that can be generated safely by a random generator without collision handling.

---

## 4. High-Level Design (HLD)

### 4.1 Architecture

Image reference: `media/img/url-shortener-hld.png`

```text
                     +--------------------+
                     |       Client       |
                     +---------+----------+
                               |
                               v
                     +--------------------+
                     | Load Balancer /    |
                     | API Gateway        |
                     +---------+----------+
                               |
                 +-------------+-------------+
                 |                           |
                 v                           v
       +-------------------+       +-------------------+
       | URL Creation      |       | Redirect Service  |
       | Service           |       |                   |
       +---------+---------+       +---------+---------+
                 |                           |
                 v                           v
       +-------------------+       +-------------------+
       | ID Generator      |       | Redis Cache       |
       +---------+---------+       +---------+---------+
                 |                           |
                 v                           v
       +------------------------------------------------+
       |              URL Mapping Database              |
       +------------------------------------------------+
                               |
                               v
                     +--------------------+
                     | Analytics Events   |
                     | Queue / Stream     |
                     +---------+----------+
                               |
                               v
                     +--------------------+
                     | Analytics Workers  |
                     +--------------------+
```

The URL creation and redirect paths are separated logically because their traffic patterns differ. They may be deployed as separate services or as modules within one application.

### 4.2 URL creation flow

1. The client submits a long URL.
2. The API validates the URL and applies security checks.
3. The system checks custom-alias availability if an alias was requested.
4. The ID generator produces a unique identifier or code.
5. The system persists the mapping in the database.
6. The API returns the completed short URL.

The mapping must be persisted before the short URL is returned as successfully created.

### 4.3 Redirect flow

1. A user opens a short URL.
2. The redirect service extracts the short code.
3. The service checks Redis for the mapping.
4. On a cache hit, it retrieves the destination.
5. On a cache miss, it queries the database and populates the cache.
6. The service validates the mapping's active and expiration status.
7. The server returns an HTTP redirect.
8. A click event may be published asynchronously for analytics.

### 4.4 Why separate the read and write paths?

URL creation usually requires validation, ID generation, and durable storage. Redirects require fast mapping lookups.

Separating the paths allows independent scaling and optimization:

- Scale redirect instances according to read traffic.
- Scale creation instances according to write traffic.
- Use caching aggressively on the redirect path.
- Process analytics asynchronously without blocking redirects.

---

## 5. Short-Code Generation Algorithms

Generating a unique short code is one of the most important design decisions.

Common approaches include:

1. Base62 encoding of a unique numeric ID.
2. Random code generation.
3. Hash-based generation.
4. Pre-generated ID allocation.

### 5.1 Base62 encoding

Base62 uses the following alphabet:

```text
0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ
```

Each character represents one of 62 possible values.

A numeric identifier can be converted to a compact representation using Base62.

For example:

```text
Numeric ID: 125
Base62:     21
```

The precise output depends on the digit alphabet and encoding convention.

#### Java implementation

```java
public final class Base62Encoder {

    private static final String ALPHABET =
        "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

    private static final int BASE = ALPHABET.length();

    private Base62Encoder() {}

    public static String encode(long number) {
        if (number < 0) {
            throw new IllegalArgumentException(
                "Number must be non-negative"
            );
        }

        if (number == 0) {
            return "0";
        }

        StringBuilder result = new StringBuilder();

        while (number > 0) {
            int remainder = (int) (number % BASE);
            result.append(ALPHABET.charAt(remainder));
            number /= BASE;
        }

        return result.reverse().toString();
    }
}
```

Usage:

```java
String shortCode = Base62Encoder.encode(125);
System.out.println(shortCode);
```

Output:

```text
21
```

#### Advantages

- Compact representation.
- Deterministic conversion.
- No collisions when the underlying numeric IDs are unique.
- Easy to implement and test.

#### Disadvantages

- Sequential IDs may reveal approximate creation volume.
- A distributed unique-ID generator is needed at scale.
- Codes are not inherently secret or unguessable.

**Important:** Base62 does not create uniqueness by itself. Uniqueness comes from the numeric ID generator.

### 5.2 Random code generation

Generate a random sequence of Base62 characters.

For example:

```text
aB3xY9
```

The system checks whether the generated code already exists. If it does, it generates another code.

#### Java implementation

```java
import java.security.SecureRandom;

public final class RandomShortCodeGenerator {

    private static final String ALPHABET =
        "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

    private static final SecureRandom RANDOM = new SecureRandom();

    private RandomShortCodeGenerator() {}

    public static String generate(int length) {
        if (length <= 0) {
            throw new IllegalArgumentException(
                "Length must be positive"
            );
        }

        StringBuilder code = new StringBuilder(length);

        for (int i = 0; i < length; i++) {
            int index = RANDOM.nextInt(ALPHABET.length());
            code.append(ALPHABET.charAt(index));
        }

        return code.toString();
    }
}
```

Usage:

```java
String code = RandomShortCodeGenerator.generate(7);
System.out.println(code);
```

The output varies on each invocation.

#### Advantages

- Does not expose sequential IDs.
- Simple to implement.
- Works well with a unique database constraint and retry logic.

#### Disadvantages

- Collisions become more likely as the code space fills.
- Requires database uniqueness checks.
- Generation can require retries under heavy load.

#### Collision probability

For a code space of size \(N\) and \(k\) generated codes, the birthday approximation gives:

\[
P(\text{collision}) \approx
1 - e^{-\frac{k(k-1)}{2N}}
\]

This approximation is useful when \(k\) is much smaller than \(N\).

A random generator must never assume that a large code space makes collisions impossible.

### 5.3 Hash-based generation

A hash function can convert a long URL into a fixed-length digest.

For example:

```text
Original URL
     |
     v
SHA-256
     |
     v
Digest
     |
     v
Truncated / encoded short code
```

However, two different URLs can produce the same truncated code.

Hash-based generation also raises a product question: should identical URLs always receive the same short code?

- If yes, the system needs a defined canonicalization and deduplication policy.
- If no, each creation request may need a unique identifier even for identical destinations.

#### Advantages

- Deterministic when the same input and encoding rules are used.
- Can support deduplication.
- Avoids storing the entire original URL in the code.

#### Disadvantages

- Truncation can cause collisions.
- URL normalization is difficult to define correctly.
- Deterministic codes may reveal relationships between inputs.

Use a unique constraint and collision-resolution strategy if hash-derived codes are used.

### 5.4 Pre-generated ID allocation

The system can allocate blocks of unique numeric IDs in advance.

For example:

```text
Generator A: IDs 1,000,000 - 1,099,999
Generator B: IDs 1,100,000 - 1,199,999
Generator C: IDs 1,200,000 - 1,299,999
```

Each generator consumes its own block without requesting a new ID from the central allocator for every URL.

#### Advantages

- Reduces contention on a central ID generator.
- Supports high-throughput creation.
- Works naturally with Base62 encoding.

#### Disadvantages

- Requires safe block allocation.
- Unused IDs may be wasted.
- Failover and allocation recovery need careful handling.

---

## 6. Database Design

### 6.1 URL mapping table

A relational schema can be used as follows:

```sql
CREATE TABLE url_mapping (
    id BIGINT PRIMARY KEY,
    short_code VARCHAR(16) NOT NULL UNIQUE,
    original_url TEXT NOT NULL,
    created_at TIMESTAMP NOT NULL,
    expires_at TIMESTAMP NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);
```

The unique constraint on `short_code` prevents two records from using the same code.

For a very large deployment, the schema, indexes, partitioning, and database engine should be selected based on actual query patterns and capacity requirements.

### 6.2 Example record

| Column | Example |
|---|---|
| id | 125 |
| short_code | 21 |
| original_url | `https://example.com/products` |
| created_at | `2026-10-19 10:00:00` |
| expires_at | `NULL` |
| is_active | `true` |

### 6.3 Indexing

The most frequent lookup is:

```sql
SELECT original_url, expires_at, is_active
FROM url_mapping
WHERE short_code = ?;
```

The unique constraint on `short_code` provides an index suitable for this lookup in common relational databases.

The primary key supports ID-based retrieval and joins.

### 6.4 SQL vs. NoSQL

| Requirement | Relational database | NoSQL key-value/document store |
|---|---|---|
| Lookup by short code | Excellent with an index | Excellent with a suitable key |
| Unique short codes | Unique constraints | Depends on key design and database guarantees |
| Transactions | Mature support | Varies by product |
| Horizontal scaling | Depends on database and architecture | Often designed for distributed scaling |
| Flexible metadata | Supported | Often convenient |
| Operational complexity | Depends on deployment | Depends on deployment |

Both approaches can work. The key requirement is a durable, uniquely addressable mapping store.

### 6.5 Handling concurrent code creation

Two requests might generate the same random code concurrently.

A pre-check such as `SELECT` followed by `INSERT` is insufficient to guarantee uniqueness because both requests may observe that the code is unused.

The database unique constraint is the final authority.

Recommended flow:

1. Generate a code.
2. Attempt to insert the mapping.
3. If insertion succeeds, return the short URL.
4. If the short-code unique constraint fails, generate a new code and retry.
5. If another constraint or database error occurs, handle it separately.

Limit retries and return an appropriate error if code generation repeatedly fails.

---

## 7. Low-Level Design (LLD)

### 7.1 Main components

| Component | Responsibility |
|---|---|
| `UrlController` | Exposes creation and redirect endpoints |
| `UrlService` | Coordinates URL creation and resolution |
| `ShortCodeGenerator` | Generates unique short codes |
| `Base62Encoder` | Converts numeric IDs to Base62 |
| `UrlRepository` | Persists and retrieves mappings |
| `UrlCache` | Caches short-code-to-URL mappings |
| `UrlValidator` | Validates URL syntax and allowed destinations |
| `RedirectService` | Resolves mappings and determines redirect behavior |
| `ClickEventPublisher` | Publishes analytics events |
| `AnalyticsConsumer` | Processes click events asynchronously |

### 7.2 Class diagram

Image reference: `media/img/url-shortener-lld.png`

```text
                 +-------------------+
                 |   UrlController   |
                 +---------+---------+
                           |
                           v
                 +-------------------+
                 |     UrlService    |
                 +---+-----------+---+
                     |           |
                     v           v
           +----------------+  +------------------+
           | ShortCode      |  | UrlValidator     |
           | Generator      |  +------------------+
           +-------+--------+
                   |
                   v
           +-------------------+
           |   UrlRepository   |
           +-------------------+
                   |
                   v
           +-------------------+
           | Database          |
           +-------------------+

                 Redirect Path
                       |
                       v
             +-------------------+
             | RedirectService   |
             +----+----------+---+
                  |          |
                  v          v
             +---------+ +-----------+
             | UrlCache| | Repository|
             +---------+ +-----------+
                  |
                  v
             +-------------------+
             | ClickEventPublisher|
             +-------------------+
```

### 7.3 Design patterns

**Strategy Pattern**

Use different short-code generation strategies, such as random codes or Base62-encoded IDs, behind a common interface.

**Repository Pattern**

Keep database-specific operations behind `UrlRepository`.

**Factory Pattern**

Use a factory when generation strategy selection depends on configuration or deployment requirements.

**Dependency Injection**

Inject services, repositories, validators, and caches to simplify testing and replacement.

**Event-driven architecture**

Publish click events asynchronously so analytics processing does not block the redirect response.

---

## 8. Java LLD Implementation

The following examples illustrate the main abstractions. They are building blocks rather than a complete, runnable application; persistence, transaction management, cache configuration, and application wiring must be supplied by the Spring Boot project.

### 8.1 UrlMapping

```java
import java.time.Instant;

public record UrlMapping(
    Long id,
    String shortCode,
    String originalUrl,
    Instant createdAt,
    Instant expiresAt,
    boolean active
) {
    public boolean isExpired(Instant now) {
        return expiresAt != null && !now.isBefore(expiresAt);
    }

    public boolean isAvailable(Instant now) {
        return active && !isExpired(now);
    }
}
```

### 8.2 ShortCodeGenerator

```java
public interface ShortCodeGenerator {

    String generate();
}
```

Random implementation:

```java
import java.security.SecureRandom;

public class SecureRandomShortCodeGenerator
        implements ShortCodeGenerator {

    private static final String ALPHABET =
        "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

    private static final SecureRandom RANDOM = new SecureRandom();

    private final int length;

    public SecureRandomShortCodeGenerator(int length) {
        if (length <= 0) {
            throw new IllegalArgumentException(
                "Length must be positive"
            );
        }

        this.length = length;
    }

    @Override
    public String generate() {
        StringBuilder result = new StringBuilder(length);

        for (int i = 0; i < length; i++) {
            result.append(
                ALPHABET.charAt(
                    RANDOM.nextInt(ALPHABET.length())
                )
            );
        }

        return result.toString();
    }
}
```

This generator produces candidate codes, not guaranteed unique codes. The repository must enforce uniqueness.

### 8.3 UrlRepository

```java
import java.util.Optional;

public interface UrlRepository {

    Optional<UrlMapping> findByShortCode(String shortCode);

    UrlMapping save(UrlMapping mapping);
}
```

A Spring Data JPA implementation can use a unique database constraint on `shortCode`.

### 8.4 UrlCache

```java
import java.util.Optional;

public interface UrlCache {

    Optional<UrlMapping> get(String shortCode);

    void put(String shortCode, UrlMapping mapping);

    void evict(String shortCode);
}
```

The cache should have a bounded capacity or expiration policy. Negative caching for missing codes can also be useful, provided it has a short TTL and does not interfere with newly created links.

### 8.5 UrlValidator

```java
import java.net.URI;
import java.util.Set;

public class UrlValidator {

    private static final Set<String> ALLOWED_SCHEMES =
        Set.of("http", "https");

    public URI validate(String value) {
        if (value == null || value.isBlank()) {
            throw new IllegalArgumentException(
                "URL must not be empty"
            );
        }

        final URI uri;

        try {
            uri = URI.create(value);
        } catch (IllegalArgumentException exception) {
            throw new IllegalArgumentException(
                "Invalid URL format",
                exception
            );
        }

        String scheme = uri.getScheme();

        if (scheme == null
                || !ALLOWED_SCHEMES.contains(
                    scheme.toLowerCase())) {
            throw new IllegalArgumentException(
                "Only HTTP and HTTPS URLs are supported"
            );
        }

        if (uri.getHost() == null || uri.getHost().isBlank()) {
            throw new IllegalArgumentException(
                "URL must contain a valid host"
            );
        }

        return uri;
    }
}
```

This is a basic syntax and scheme validator. A production service should additionally address IDNs, destination policies, malformed host representations, abuse detection, and safe rendering of links.

If the backend fetches destination URLs to inspect them, it must also defend against server-side request forgery (SSRF), including private, loopback, and link-local destinations. A normal redirect service should not fetch arbitrary destination URLs merely to redirect users.

### 8.6 UrlService

```java
import java.time.Clock;
import java.time.Instant;
import java.util.Optional;

public class UrlService {

    private final ShortCodeGenerator codeGenerator;
    private final UrlRepository repository;
    private final UrlCache cache;
    private final UrlValidator validator;
    private final Clock clock;

    public UrlService(
            ShortCodeGenerator codeGenerator,
            UrlRepository repository,
            UrlCache cache,
            UrlValidator validator,
            Clock clock) {

        this.codeGenerator = codeGenerator;
        this.repository = repository;
        this.cache = cache;
        this.validator = validator;
        this.clock = clock;
    }

    public UrlMapping createShortUrl(
            String originalUrl,
            Instant expiresAt) {

        validator.validate(originalUrl);

        if (expiresAt != null
                && !expiresAt.isAfter(clock.instant())) {
            throw new IllegalArgumentException(
                "Expiration must be in the future"
            );
        }

        for (int attempt = 0; attempt < 5; attempt++) {
            String code = codeGenerator.generate();

            UrlMapping mapping = new UrlMapping(
                null,
                code,
                originalUrl,
                clock.instant(),
                expiresAt,
                true
            );

            try {
                UrlMapping saved = repository.save(mapping);
                cache.put(saved.shortCode(), saved);
                return saved;
            } catch (DuplicateShortCodeException exception) {
                // Retry only for a short-code uniqueness conflict.
            }
        }

        throw new IllegalStateException(
            "Unable to generate a unique short code"
        );
    }

    public UrlMapping resolve(String shortCode) {
        Optional<UrlMapping> cached = cache.get(shortCode);

        UrlMapping mapping = cached.orElseGet(() ->
            repository.findByShortCode(shortCode)
                .orElseThrow(() ->
                    new UrlNotFoundException(shortCode)
                )
        );

        if (!mapping.isAvailable(clock.instant())) {
            cache.evict(shortCode);
            throw new UrlNotFoundException(shortCode);
        }

        cache.put(shortCode, mapping);
        return mapping;
    }
}
```

`DuplicateShortCodeException` and `UrlNotFoundException` are application-specific exceptions that must be defined. The repository must translate only the relevant unique-constraint violation into `DuplicateShortCodeException`; unrelated persistence failures should propagate to normal error handling.

This service assumes a cache implementation that correctly handles expiry and invalidation. For highly concurrent creation, transaction boundaries and database uniqueness remain essential.

---

## 9. Spring Boot REST API

### 9.1 API endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/urls` | Create a short URL |
| GET | `/{shortCode}` | Redirect to the original URL |
| GET | `/api/urls/{shortCode}` | Retrieve link metadata |
| DELETE | `/api/urls/{shortCode}` | Deactivate a link |
| PATCH | `/api/urls/{shortCode}` | Update an authorized link |

The metadata, update, and delete endpoints require an appropriate authorization model.

### 9.2 Request and response

Request:

```http
POST /api/urls
Content-Type: application/json
```

```json
{
  "originalUrl": "https://example.com/products/123",
  "expiresAt": null
}
```

Response:

```json
{
  "shortCode": "aB3xY9",
  "shortUrl": "https://sho.rt/aB3xY9",
  "originalUrl": "https://example.com/products/123"
}
```

### 9.3 DTOs

```java
import java.time.Instant;

public record CreateUrlRequest(
    String originalUrl,
    Instant expiresAt
) {}
```

```java
public record CreateUrlResponse(
    String shortCode,
    String shortUrl,
    String originalUrl
) {}
```

### 9.4 UrlController

```java
import java.net.URI;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
public class UrlController {

    private final UrlService urlService;
    private final String publicBaseUrl;

    public UrlController(
            UrlService urlService,
            @Value("${app.short-url-base}") String publicBaseUrl) {

        this.urlService = urlService;
        this.publicBaseUrl =
            publicBaseUrl.replaceAll("/+$", "");
    }

    @PostMapping("/api/urls")
    public ResponseEntity<CreateUrlResponse> create(
            @RequestBody CreateUrlRequest request) {

        UrlMapping mapping = urlService.createShortUrl(
            request.originalUrl(),
            request.expiresAt()
        );

        CreateUrlResponse response = new CreateUrlResponse(
            mapping.shortCode(),
            publicBaseUrl + "/" + mapping.shortCode(),
            mapping.originalUrl()
        );

        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(response);
    }

    @GetMapping("/{shortCode:[a-zA-Z0-9]+}")
    public ResponseEntity<Void> redirect(
            @PathVariable String shortCode) {

        UrlMapping mapping = urlService.resolve(shortCode);

        return ResponseEntity
            .status(HttpStatus.FOUND)
            .location(URI.create(mapping.originalUrl()))
            .build();
    }
}
```

Configuration:

```yaml
app:
  short-url-base: https://sho.rt
```

This controller demonstrates the primary API flow. Define exception handlers for invalid requests, missing links, and persistence failures. Add request validation, authorization, and abuse protection before exposing the endpoint publicly.

The redirect example uses HTTP 302 (`FOUND`), which is appropriate when the destination can change or when the service wants clients to resolve the destination dynamically. Other redirect codes are discussed below.

---

## 10. Redirect HTTP Status Codes

The choice of redirect code affects client and intermediary caching behavior.

| Status | Meaning | Typical use |
|---|---|---|
| 301 Moved Permanently | Permanent redirect | Stable mappings where permanent caching is acceptable |
| 302 Found | Temporary redirect | Common default for flexible URL shorteners |
| 303 See Other | Redirect using GET | Certain post/redirect/get workflows |
| 307 Temporary Redirect | Temporary redirect preserving method | Method-preserving redirects |
| 308 Permanent Redirect | Permanent redirect preserving method | Method-preserving permanent redirects |

For a typical public URL shortener, GET requests commonly receive a 302 response.

A 301 may be cached by browsers and intermediaries, making later destination changes difficult to enforce. Select the code according to the product's semantics rather than treating all redirect statuses as interchangeable.

---

## 11. Caching Strategy

Redirect requests often outnumber creation requests, so caching is an important optimization.

### 11.1 Cache-aside pattern

Image reference: `media/img/url-shortener-cache-flow.png`

```text
          Redirect Request
                 |
                 v
           +-----------+
           | Redis     |
           +-----+-----+
                 |
         +-------+-------+
         |               |
      Cache Hit       Cache Miss
         |               |
         v               v
   Return Mapping    Database Lookup
                         |
                         v
                    Update Cache
                         |
                         v
                   Return Mapping
```

Flow:

1. Check Redis for the short code.
2. If the mapping exists, return it.
3. Otherwise, query the database.
4. If found, cache the mapping.
5. Return the mapping.

### 11.2 Cache key and value

Key:

```text
url:aB3xY9
```

Value:

```json
{
  "originalUrl": "https://example.com/products/123",
  "expiresAt": null,
  "active": true
}
```

### 11.3 TTL selection

A cache TTL should consider:

- Expected popularity of links.
- Available memory.
- Link expiration.
- Destination-update frequency.
- Deactivation requirements.

The cache TTL should not extend the effective lifetime of an expired link. Even with cached mappings, the application must enforce expiration correctly.

### 11.4 Cache invalidation

When a destination changes or a link is deactivated:

1. Update the authoritative database record.
2. Invalidate or update the cache.
3. Publish invalidation events if multiple cache layers exist.

If deactivation must take effect immediately, avoid relying solely on a long-lived cache entry without a reliable invalidation or authoritative-check mechanism.

### 11.5 Negative caching

Requests for nonexistent short codes can repeatedly hit the database.

A short-lived negative cache can reduce this load. Use a short TTL and ensure that creating a new mapping invalidates the corresponding negative entry.

Do not allow arbitrary invalid codes to create unbounded cache entries.

---

## 12. Analytics and Click Tracking

Analytics can include:

- Total click count.
- Click timestamps.
- Referrer.
- User-agent category.
- Approximate geography.
- Device category.

Collect only the data needed for the product and comply with applicable privacy requirements.

### 12.1 Why asynchronous analytics?

A redirect should not wait for expensive analytics processing.

Recommended flow:

```text
User Click
    |
    v
Redirect Service
    |
    +------> Return Redirect
    |
    v
Publish Click Event
    |
    v
Message Queue / Stream
    |
    v
Analytics Consumer
    |
    v
Analytics Store
```

The event publisher must have a clear delivery and failure strategy. If analytics are best-effort, a publishing failure should not normally block the redirect. If every click must be durably recorded, a stronger delivery design is needed.

### 12.2 Click event model

```java
import java.time.Instant;

public record ClickEvent(
    String shortCode,
    Instant timestamp,
    String referrer,
    String userAgent
) {}
```

Avoid storing full IP addresses or other identifying information unless there is a justified requirement and appropriate retention and access controls.

### 12.3 Analytics storage

Do not update a heavily contended relational row synchronously for every click if the traffic volume is high.

Possible approaches include:

- Aggregating click counts asynchronously.
- Publishing events to Kafka or another message broker.
- Maintaining counters in a suitable key-value store.
- Writing detailed events to an analytics database or data lake.

For approximate analytics, counters may be sufficient. For detailed reporting, an event stream provides greater flexibility at the cost of additional infrastructure.

---

## 13. Scalability and Reliability

### 13.1 Horizontal scaling

Run multiple stateless creation and redirect instances behind a load balancer.

Scale the redirect path independently if read traffic dominates.

### 13.2 Database scaling

Potential techniques include:

- Appropriate indexes.
- Read replicas for suitable read workloads.
- Partitioning or sharding when necessary.
- Connection pooling.
- Archiving or expiring records according to the product's retention policy.

If a link is created on the primary database and immediately read from a lagging replica, the redirect may temporarily fail. Read-after-write requirements should influence replica routing and consistency choices.

### 13.3 Unique ID generation at scale

For a large distributed deployment, consider:

- Database-generated sequences.
- Snowflake-style IDs.
- Pre-allocated ID ranges.
- Dedicated ID-generation services.

The choice depends on throughput, coordination requirements, operational complexity, and whether IDs need to reveal ordering.

### 13.4 Failure handling

| Failure | Handling strategy |
|---|---|
| Redis unavailable | Fall back to the database when safe and capacity permits |
| Database unavailable | Serve valid cache entries where policy allows; reject new creation requests |
| ID generator unavailable | Use a safe fallback generator or reject creation |
| Analytics broker unavailable | Apply a defined best-effort, retry, or durable-outbox strategy |
| Invalid short code | Return 404 |
| Expired or inactive mapping | Return the product-defined unavailable response |
| Duplicate generated code | Retry after a confirmed uniqueness conflict |

Do not retry all database exceptions as if they were duplicate-code collisions.

### 13.5 Availability and consistency

A URL shortener can often tolerate eventual consistency for analytics, but the mapping itself requires reliable persistence.

Separate the consistency requirements:

- **URL creation:** A successfully returned link must have a durable mapping.
- **Redirects:** Newly created links should resolve promptly.
- **Analytics:** Click counts may be eventually consistent.
- **Deactivation:** The required time for deactivation to take effect must be explicit.

---

## 14. Security Considerations

### 14.1 Open redirect abuse

A URL shortener intentionally redirects to arbitrary destinations, so it can be abused to disguise malicious links.

Mitigations include:

- Validate the destination scheme.
- Block malformed or unsupported destinations.
- Apply abuse detection and reporting.
- Rate-limit link creation.
- Scan or classify destinations when appropriate.
- Display destination previews where useful.
- Provide mechanisms to deactivate reported links.

Do not claim that checking for HTTP or HTTPS makes a destination safe.

### 14.2 Enumeration

Sequential IDs make it easier to guess nearby short codes.

Possible mitigations:

- Use random codes.
- Encode unique IDs with a reversible permutation.
- Enforce authorization on management endpoints.
- Rate-limit code lookups where abuse is a concern.

Random codes do not replace authorization.

### 14.3 Authorization

Only the owner or an authorized administrator should be able to edit or deactivate a protected link.

Possession of a short code should not automatically grant management privileges.

### 14.4 Expiration

Expired mappings must not redirect merely because a stale cache entry exists.

Use expiration-aware reads, cache TTLs, and invalidation to enforce the defined expiration policy.

### 14.5 Domain and deployment configuration

Use HTTPS for the public short domain, validate the configured base URL, and avoid constructing redirect destinations from untrusted host headers.

---

## 15. Testing Strategy

### Unit tests

Test:

- Base62 encoding and boundary values.
- Random-code length and allowed characters.
- URL validation.
- Expiration behavior.
- Cache hit and miss behavior.
- Missing short codes.
- Inactive mappings.
- Duplicate-code retries.

### Integration tests

Test:

- Database persistence.
- Unique constraints.
- Redis cache population.
- Cache invalidation after updates.
- Redirect status and `Location` header.
- Expiration behavior when mappings are cached.
- Database or Redis failure handling.

### Concurrency tests

Generate many concurrent creation requests and verify that:

- Every successful short code is unique.
- No mapping is overwritten by a conflicting code.
- Duplicate-code retries are bounded.
- Creation remains correct under contention.

### Load tests

Measure:

- Redirect requests per second.
- Creation throughput.
- Cache hit ratio.
- Database query latency.
- Redis latency.
- p95 and p99 response times.
- Analytics queue lag.
- Memory usage as the mapping count grows.

Test both popular hot links and uniformly distributed short-code lookups.

---

## 16. Common Interview Questions

### Why use Base62?

Base62 encodes numeric identifiers compactly using 62 characters. It produces shorter representations than decimal encoding for the same positive integer.

### Does Base62 guarantee uniqueness?

No. Base62 is an encoding. Uniqueness depends on the numeric ID generator and database constraints.

### Why not use a hash of the original URL?

Truncating a hash can produce collisions, and deterministic hashing may not match the product's desired behavior for repeated URLs. It can be used if the system has a suitable collision-resolution and deduplication policy.

### Why is the system read-heavy?

A single created link can be clicked many times. Therefore, redirect requests can greatly outnumber creation requests, making cache efficiency important.

### Why use Redis?

Redis can serve frequently accessed mappings quickly and reduce database read load. It should be treated as a cache rather than the only durable mapping store unless the overall data architecture explicitly provides the required durability.

### Why not store click counts in the URL mapping row?

Frequent updates to a popular row can create contention. Asynchronous event processing and aggregation can scale better for high-volume analytics.

### Should the redirect service use 301 or 302?

Use 302 when redirects should remain flexible and destinations may change. Use 301 when permanent caching behavior is intended and destination changes are not expected to require immediate propagation.

### How do you prevent duplicate short codes?

Use a database unique constraint as the final guarantee. Retry only when a confirmed short-code collision occurs.

### What if Redis fails?

The service may fall back to the database if capacity and availability permit. The fallback should be bounded so that a cache failure does not overload the database.

### How do you prevent malicious links?

Validate destinations, apply abuse controls, support link reporting and deactivation, and use appropriate security policies. Syntax validation alone cannot establish that a destination is trustworthy.

### How do you handle billions of URLs?

Use a scalable persistent store, efficient indexes, caching, a distributed ID-generation strategy, and partitioning or sharding where justified by measured load. Capacity estimates should guide the architecture.

---

## 17. Final Design Recommendation

For a scalable Java and Spring Boot URL shortener, a practical starting design is:

1. Use separate logical paths for URL creation and redirection.
2. Generate unique IDs and encode them using Base62, or use random codes with collision handling.
3. Enforce short-code uniqueness in the database.
4. Store mappings durably in a relational or suitable distributed database.
5. Use Redis cache-aside for popular mappings.
6. Return HTTP 302 by default when redirect destinations may change.
7. Process click analytics asynchronously.
8. Define expiration, deactivation, and cache invalidation semantics.
9. Apply validation, authorization, rate limiting, and abuse detection.
10. Monitor cache hit ratio, redirect latency, database load, and creation failures.

Start with the simplest architecture that meets the expected workload. Introduce sharding, distributed ID allocation, and more complex analytics pipelines when measured traffic or availability requirements justify them.

---

## 18. Summary

A URL shortener converts long URLs into compact identifiers and redirects users to their original destinations.

The main engineering challenges are:

- Generating unique short codes.
- Persisting and retrieving URL mappings efficiently.
- Handling a read-heavy workload.
- Maintaining correct behavior during cache misses, expiration, and deactivation.
- Processing analytics without slowing down redirects.
- Preventing abuse and unauthorized link management.

Base62 encoding, a persistent mapping store, Redis caching, and asynchronous analytics form a practical foundation. The design should evolve according to actual traffic, reliability, and security requirements.
