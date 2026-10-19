---
title: Design a Rate Limiter
parent: System Design
nav_order: 4
---

# Design a Rate Limiter
{: .no_toc }

High-Level Design (HLD), Low-Level Design (LLD), algorithms, Java implementation, and distributed rate limiting.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }

1. TOC
{:toc}
</details>

---

## 1. What is a Rate Limiter?

A **rate limiter** controls how many requests a client can make to a service within a specified period.

For example:

- A user can create at most 2 posts per second.
- An IP address can create at most 10 accounts per day.
- A user can attempt to log in at most 5 times per minute.
- An API client can make at most 1,000 requests per hour.

When a request exceeds the configured limit, the rate limiter rejects it or delays its processing, depending on the policy.

### Why do we need a Rate Limiter?

1. **Prevent abuse and DoS attacks:** Restrict excessive requests from a client or IP address.
2. **Protect backend services:** Prevent sudden traffic spikes from overwhelming application servers.
3. **Control third-party API costs:** Limit paid external API calls.
4. **Ensure fair usage:** Prevent one client from consuming most of the available resources.
5. **Protect critical operations:** Apply stricter limits to login, payment, registration, and other sensitive endpoints.

### Example

Suppose an API permits 3 requests per second.

| Request | Result |
|---|---|
| Request 1 | Allowed |
| Request 2 | Allowed |
| Request 3 | Allowed |
| Request 4 | Rejected with HTTP 429 |

The exact result depends on the selected algorithm, the request timestamps, and the configured policy.

---

## 2. Functional and Non-Functional Requirements

### Functional requirements

- Configure limits for different APIs and clients.
- Support limits based on user ID, IP address, API key, or endpoint.
- Allow requests within the configured limit.
- Reject requests that exceed the limit.
- Return useful rate-limit metadata to clients.
- Support different time units, such as seconds, minutes, hours, and days.
- Support distributed deployments with multiple application servers.

### Non-functional requirements

- **Low latency:** Rate-limit evaluation should add minimal request-processing overhead.
- **Scalability:** Support a large number of clients and concurrent requests.
- **Accuracy:** Enforce the required quota within the selected algorithm's guarantees.
- **Memory efficiency:** Avoid storing unnecessary request history.
- **Availability:** Handle failures in the rate-limiting infrastructure according to an explicit fallback policy.
- **Concurrency safety:** Prevent concurrent requests from incorrectly exceeding the limit.
- **Observability:** Measure allowed requests, rejected requests, latency, and infrastructure failures.

### Assumptions

For the following design, assume:

- The system exposes HTTP APIs.
- Multiple stateless application instances run behind a load balancer.
- Redis stores distributed rate-limit state.
- Rate-limit policies are configurable.
- The rate limiter can identify the client and the requested API.
- HTTP 429 is used when a request exceeds its quota.

These are design assumptions, not mandatory requirements for every implementation.

---

## 3. High-Level Design (HLD)

### 3.1 Where should a Rate Limiter be placed?

There are three common approaches.

#### Client-side rate limiting

The client restricts its own outgoing requests.

Advantages:
- Reduces unnecessary requests.
- Helps prevent accidental API abuse.
- Can reduce network traffic.

Disadvantages:
- Cannot enforce a security boundary because clients can bypass the restriction.
- Requires control over the client implementation.

Client-side rate limiting is useful as an optimization, but it should not be the only enforcement mechanism.

#### Application-level rate limiting

The application checks the quota before executing business logic.

Advantages:
- Complete control over custom policies.
- Easy integration with user authentication and business rules.
- Suitable for application-specific restrictions.

Disadvantages:
- Each application may implement different logic.
- Distributed deployments require shared state or another coordination strategy.
- Rate-limit checks consume application resources.

#### API gateway or middleware rate limiting

A gateway or middleware checks requests before forwarding them to backend services.

Advantages:
- Centralized enforcement.
- Protects backend services from excessive traffic.
- Can apply common rules across multiple APIs.
- Integrates naturally with authentication and routing.

Disadvantages:
- Adds another processing component.
- May require distributed state for accurate global quotas.
- Gateway capabilities and supported algorithms vary by product.

**Design decision:** Use gateway or middleware enforcement for common API protection, with application-level checks for business-specific quotas when necessary.

Avoid enforcing the same quota independently in multiple layers unless the policies are intentionally different.

### 3.2 High-Level Architecture

Image reference: `media/img/rate-limiter-hld.png`

```text
                         +------------------+
                         |      Client      |
                         +--------+---------+
                                  |
                                  v
                         +------------------+
                         |  Load Balancer   |
                         +--------+---------+
                                  |
                                  v
                         +------------------+
                         | API Gateway /    |
                         | Rate Limiter     |
                         +---+----------+---+
                             |          |
                 Load rules  |          | Check quota
                             v          v
                     +---------------+  +---------------+
                     | Rules Cache   |  | Redis         |
                     |               |  | Counters /    |
                     |               |  | Bucket State  |
                     +---------------+  +---------------+
                             ^
                             |
                     +---------------+
                     | Policy Store  |
                     +---------------+

                 Allowed requests
                         |
                         v
                 +-------------------+
                 | Backend Services  |
                 +-------------------+
                         |
                         v
                 +-------------------+
                 | Response to Client|
                 +-------------------+
```

The diagram represents logical components. A deployment may combine the gateway and rate limiter or use a managed API gateway.

### 3.3 Request flow

1. The client sends an HTTP request.
2. The gateway identifies the client and requested endpoint.
3. The rate limiter retrieves the applicable policy from its local cache.
4. It constructs a rate-limit key.
5. It checks and updates the corresponding state atomically in Redis.
6. If the request is allowed, the gateway forwards it to the backend.
7. If the request is rejected, the gateway returns HTTP 429.
8. Metrics record the decision and relevant latency.

### 3.4 Rate-limit key design

The key identifies the quota being enforced.

For example:

```text
rate-limit:{user-123}:POST:/api/posts
rate-limit:{ip-203.0.113.10}:POST:/api/accounts
rate-limit:{api-key-456}:GET:/api/products
```

A key can be based on:

- User ID
- Client IP address
- API key
- HTTP method
- API endpoint
- Tenant ID
- A combination of these values

Use a stable, normalized representation so equivalent requests map to the same key.

Do not trust arbitrary client-supplied identity headers. Derive identity from authenticated credentials or trusted proxy metadata. IP-based policies must account for NAT, reverse proxies, and spoofed forwarding headers.

---

## 4. Rate-Limiting Algorithms

Five commonly used algorithms are:

1. Token Bucket
2. Leaky Bucket
3. Fixed Window Counter
4. Sliding Window Log
5. Sliding Window Counter

Each algorithm has different behavior regarding burst traffic, accuracy, memory usage, and implementation complexity.

### 4.1 Token Bucket

#### Concept

A bucket holds a limited number of tokens. Tokens are replenished at a configured rate.

Each accepted request consumes one token.

Parameters:

- **Bucket capacity:** Maximum number of available tokens.
- **Refill rate:** Number of tokens replenished per unit of time.

A request is allowed if at least one token is available.

#### Example

Suppose:

```text
Bucket capacity = 5 tokens
Refill rate     = 2 tokens per second
Initial tokens  = 5
```

The first five requests can be accepted immediately. Once the bucket is empty, a request must wait for a token to become available or be rejected.

Image reference: `media/img/token-bucket.png`

#### Java implementation

```java
public final class TokenBucket {

    private final long capacity;
    private final double refillTokensPerNano;
    private double tokens;
    private long lastRefillNanos;

    public TokenBucket(long capacity, double refillPerSecond) {
        if (capacity <= 0 || refillPerSecond <= 0) {
            throw new IllegalArgumentException(
                "Capacity and refill rate must be positive"
            );
        }

        this.capacity = capacity;
        this.refillTokensPerNano =
            refillPerSecond / 1_000_000_000.0;

        this.tokens = capacity;
        this.lastRefillNanos = System.nanoTime();
    }

    public synchronized boolean allowRequest() {
        long now = System.nanoTime();
        long elapsed = Math.max(0, now - lastRefillNanos);

        tokens = Math.min(
            capacity,
            tokens + elapsed * refillTokensPerNano
        );

        lastRefillNanos = now;

        if (tokens >= 1.0) {
            tokens -= 1.0;
            return true;
        }

        return false;
    }
}
```

Usage:

```java
TokenBucket bucket = new TokenBucket(5, 2);

if (bucket.allowRequest()) {
    System.out.println("Request allowed");
} else {
    System.out.println("Request rejected");
}
```

**Important:** This implementation is thread-safe within a single JVM, but each application instance has its own bucket. It does not enforce a shared quota across multiple servers.

For distributed enforcement, store bucket state centrally or use a coordinated implementation.

#### Advantages

- Simple to understand and implement.
- Memory-efficient.
- Allows controlled bursts.
- Suitable for APIs that need an average request rate with short bursts.

#### Disadvantages

- Capacity and refill rate must be tuned correctly.
- Independent local buckets do not provide a global limit.
- Distributed updates require atomic state transitions.

#### Best use cases

- Public APIs
- Gateway request throttling
- Services that permit short bursts of traffic

---

### 4.2 Leaky Bucket

#### Concept

The leaky bucket processes queued requests at a controlled, fixed rate.

A bounded FIFO queue stores incoming requests. The system removes requests from the queue at a configured rate.

- If the queue has capacity, the request is enqueued.
- If the queue is full, the request is rejected.
- A worker drains the queue at a controlled rate.

Image reference: `media/img/leaky-bucket.png`

#### Example

```text
Queue capacity = 10 requests
Processing rate = 2 requests per second
```

Even if 10 requests arrive together, they are processed gradually rather than all at once.

#### Simplified Java implementation

This example models a leaky bucket using a queue and a scheduled worker.

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;
import java.util.function.Consumer;

public final class LeakyBucket implements AutoCloseable {

    private final BlockingQueue<Runnable> queue;
    private final ScheduledExecutorService worker;
    private final Consumer<Runnable> processor;

    public LeakyBucket(
            int capacity,
            long intervalMillis,
            Consumer<Runnable> processor) {

        if (capacity <= 0 || intervalMillis <= 0) {
            throw new IllegalArgumentException(
                "Capacity and interval must be positive"
            );
        }

        this.queue = new ArrayBlockingQueue<>(capacity);
        this.processor = processor;

        this.worker = Executors.newSingleThreadScheduledExecutor();

        worker.scheduleAtFixedRate(
            this::processOne,
            0,
            intervalMillis,
            TimeUnit.MILLISECONDS
        );
    }

    public boolean submit(Runnable request) {
        return queue.offer(request);
    }

    private void processOne() {
        Runnable request = queue.poll();

        if (request != null) {
            try {
                processor.accept(request);
            } catch (RuntimeException exception) {
                // Record the failure and apply the application's
                // error-handling policy.
            }
        }
    }

    @Override
    public void close() {
        worker.shutdown();
    }
}
```

Usage:

```java
LeakyBucket bucket = new LeakyBucket(
    10,
    500,
    request -> request.run()
);

boolean accepted = bucket.submit(
    () -> System.out.println("Processing request")
);

if (!accepted) {
    System.out.println("Queue full; request rejected");
}
```

The interval of 500 milliseconds schedules approximately two processing attempts per second. Actual processing throughput depends on the processor and scheduler behavior.

This is a simplified demonstration, not a production-ready HTTP request queue. A production implementation needs lifecycle management, task timeouts, failure handling, backpressure, and bounded execution resources.

#### Advantages

- Smooths traffic before it reaches downstream processing.
- Uses a bounded queue.
- Suitable when a stable processing rate is required.

#### Disadvantages

- Queued requests can become stale.
- Queue overflow causes rejection.
- Queueing increases latency.
- A slow processor can reduce effective throughput.

#### Best use cases

- Background jobs
- Controlled message processing
- Workloads where requests may wait in a queue

**Distinction:** A leaky bucket that queues requests is not identical to an immediate-rejection rate limiter. Choose the behavior according to whether requests may be delayed.

---

### 4.3 Fixed Window Counter

#### Concept

Divide time into fixed-size windows and maintain a counter for each client and policy.

For a limit of 100 requests per minute:

- The counter starts at zero for a new minute.
- Every request increments the counter.
- Requests are accepted until the counter reaches 100.
- Further requests are rejected until the next window.

Image reference: `media/img/fixed-window-counter.png`

#### Example

```text
Window 1: 10:00:00 - 10:00:59
Allowed requests: 100

Window 2: 10:01:00 - 10:01:59
Allowed requests: 100
```

A client can send 100 requests at 10:00:59 and another 100 at 10:01:00. Consequently, 200 requests can pass within a short interval around the window boundary.

#### Java implementation

```java
import java.time.Clock;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

public final class FixedWindowRateLimiter {

    private static final class WindowCounter {
        private final long windowId;
        private final AtomicLong count = new AtomicLong();

        private WindowCounter(long windowId) {
            this.windowId = windowId;
        }
    }

    private final long limit;
    private final long windowMillis;
    private final Clock clock;

    private final ConcurrentHashMap<String, WindowCounter> counters =
        new ConcurrentHashMap<>();

    public FixedWindowRateLimiter(
            long limit,
            long windowMillis,
            Clock clock) {

        if (limit <= 0 || windowMillis <= 0) {
            throw new IllegalArgumentException(
                "Limit and window must be positive"
            );
        }

        this.limit = limit;
        this.windowMillis = windowMillis;
        this.clock = clock;
    }

    public boolean allowRequest(String key) {
        long windowId = clock.millis() / windowMillis;

        WindowCounter counter = counters.compute(key, (k, current) -> {
            if (current == null || current.windowId != windowId) {
                return new WindowCounter(windowId);
            }
            return current;
        });

        while (true) {
            long currentCount = counter.count.get();

            if (currentCount >= limit) {
                return false;
            }

            if (counter.count.compareAndSet(
                    currentCount, currentCount + 1)) {
                return true;
            }
        }
    }
}
```

Usage:

```java
FixedWindowRateLimiter limiter =
    new FixedWindowRateLimiter(
        100,
        60_000,
        Clock.systemUTC()
    );

if (limiter.allowRequest("user-123")) {
    System.out.println("Allowed");
} else {
    System.out.println("Rate limit exceeded");
}
```

This example uses in-memory state and does not implement automatic eviction of inactive keys. It is intended to illustrate the algorithm, not distributed production enforcement.

#### Advantages

- Simple implementation.
- Low memory usage.
- Easy to understand and monitor.
- Natural fit for quotas that reset at fixed boundaries.

#### Disadvantages

- Allows traffic spikes around window boundaries.
- Requires careful handling of counter expiration and concurrent updates.
- Separate application instances need coordinated state for a shared quota.

#### Best use cases

- Basic per-minute API quotas
- Daily account creation limits
- Workloads where fixed-boundary behavior is acceptable

---

### 4.4 Sliding Window Log

#### Concept

Maintain timestamps for requests within a rolling time window.

For every incoming request:

1. Remove timestamps older than the current window.
2. Count the remaining timestamps.
3. Reject the request if the limit has already been reached.
4. Otherwise, record the new timestamp and accept the request.

Image reference: `media/img/sliding-window-log.png`

#### Example

```text
Limit = 3 requests per 60 seconds

Existing timestamps:
10:00:10
10:00:25
10:00:40

New request:
10:00:50

Result:
Rejected, because 3 requests already occurred
within the previous 60 seconds.
```

#### Java implementation

```java
import java.time.Clock;
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.HashMap;
import java.util.Map;

public final class SlidingWindowLogRateLimiter {

    private final int limit;
    private final long windowMillis;
    private final Clock clock;

    private final Map<String, Deque<Long>> requestLogs =
        new HashMap<>();

    public SlidingWindowLogRateLimiter(
            int limit,
            long windowMillis,
            Clock clock) {

        if (limit <= 0 || windowMillis <= 0) {
            throw new IllegalArgumentException(
                "Limit and window must be positive"
            );
        }

        this.limit = limit;
        this.windowMillis = windowMillis;
        this.clock = clock;
    }

    public synchronized boolean allowRequest(String key) {
        long now = clock.millis();
        long windowStart = now - windowMillis;

        Deque<Long> timestamps = requestLogs.computeIfAbsent(
            key,
            ignored -> new ArrayDeque<>()
        );

        while (!timestamps.isEmpty()
                && timestamps.peekFirst() <= windowStart) {
            timestamps.removeFirst();
        }

        if (timestamps.size() >= limit) {
            return false;
        }

        timestamps.addLast(now);
        return true;
    }
}
```

The timestamp is added only when the request is accepted. This prevents rejected requests from unnecessarily consuming the allowed-request history in this implementation.

Usage:

```java
SlidingWindowLogRateLimiter limiter =
    new SlidingWindowLogRateLimiter(
        3,
        60_000,
        Clock.systemUTC()
    );

System.out.println(limiter.allowRequest("user-123"));
```

#### Advantages

- Accurately enforces the rolling-window request count.
- Avoids fixed-window boundary spikes.
- Provides clear request-history semantics.

#### Disadvantages

- Stores individual request timestamps.
- Memory consumption increases with the number of accepted requests.
- Requires efficient cleanup of expired timestamps.
- Distributed concurrent updates require atomic operations.

#### Best use cases

- Strict rolling-window quotas
- Sensitive APIs with precise request-count requirements
- Systems where the memory cost is acceptable

---

### 4.5 Sliding Window Counter

#### Concept

The sliding window counter combines the fixed window counter with an estimate of the previous window's contribution.

Instead of storing every request timestamp, it stores:

- The count for the previous window.
- The count for the current window.
- The current position within the current window.

The previous window is weighted by the fraction that overlaps the rolling window.

#### Formula

```text
Estimated Requests =
    Current Window Count
    + Previous Window Count × Previous Window Weight
```

Where:

```text
Previous Window Weight =
    1 - (Elapsed Time in Current Window / Window Duration)
```

#### Example

Assume:

```text
Previous window count = 5
Current window count  = 3
Window duration       = 60 seconds
Elapsed current time  = 18 seconds
```

Then:

```text
Previous window weight = 1 - (18 / 60)
                       = 0.7

Estimated requests = 3 + (5 × 0.7)
                   = 6.5
```

If the policy allows 7 requests, the estimated count is below the limit. Whether the next request is accepted depends on the algorithm's precise comparison and counting convention.

Image reference: `media/img/sliding-window-counter.png`

#### Java implementation

```java
import java.time.Clock;
import java.util.concurrent.ConcurrentHashMap;

public final class SlidingWindowCounterRateLimiter {

    private static final class WindowState {
        private long windowId = Long.MIN_VALUE;
        private long previousCount;
        private long currentCount;
    }

    private final long limit;
    private final long windowMillis;
    private final Clock clock;

    private final ConcurrentHashMap<String, WindowState> states =
        new ConcurrentHashMap<>();

    public SlidingWindowCounterRateLimiter(
            long limit,
            long windowMillis,
            Clock clock) {

        if (limit <= 0 || windowMillis <= 0) {
            throw new IllegalArgumentException(
                "Limit and window must be positive"
            );
        }

        this.limit = limit;
        this.windowMillis = windowMillis;
        this.clock = clock;
    }

    public boolean allowRequest(String key) {
        WindowState state = states.computeIfAbsent(
            key,
            ignored -> new WindowState()
        );

        synchronized (state) {
            long now = clock.millis();
            long windowId = now / windowMillis;

            if (state.windowId != windowId) {
                if (state.windowId == windowId - 1) {
                    state.previousCount = state.currentCount;
                } else {
                    state.previousCount = 0;
                }

                state.currentCount = 0;
                state.windowId = windowId;
            }

            long elapsed = now % windowMillis;

            double previousWeight =
                1.0 - ((double) elapsed / windowMillis);

            double estimatedCount =
                state.currentCount
                + state.previousCount * previousWeight;

            if (estimatedCount >= limit) {
                return false;
            }

            state.currentCount++;
            return true;
        }
    }
}
```

This implementation uses a weighted estimate and is not an exact sliding-window log. The estimate assumes that requests in the previous window are distributed approximately evenly. The code also needs bounded state eviction for a production deployment.

#### Advantages

- Uses less memory than a timestamp log.
- Reduces fixed-window boundary spikes.
- Works well for high-throughput APIs that tolerate approximate counting.

#### Disadvantages

- It is an approximation rather than an exact rolling-window count.
- Accuracy depends on the distribution of requests in the previous window.
- State must be synchronized or updated atomically in distributed deployments.

#### Best use cases

- High-volume APIs
- General-purpose request throttling
- Systems that need a balance between memory efficiency and accuracy

---

## 5. Algorithm Comparison

| Algorithm | Memory usage | Burst handling | Accuracy | Main limitation |
|---|---|---|---|---|
| Token Bucket | Low | Allows configured bursts | Enforces token policy | Requires tuning |
| Leaky Bucket | Low to moderate | Buffers bursts up to queue capacity | Controls processing rate | Queueing increases latency |
| Fixed Window | Low | Can allow boundary spikes | Accurate within fixed windows | Boundary problem |
| Sliding Window Log | High | No fixed-window boundary spike | Exact rolling count | Stores timestamps |
| Sliding Window Counter | Low | Smooths boundary spikes | Approximate | Assumes a distribution for previous-window traffic |

There is no universally best algorithm.

- Choose **Token Bucket** when controlled bursts are acceptable.
- Choose **Leaky Bucket** when smoothing processing throughput is important.
- Choose **Fixed Window** when simplicity matters and boundary spikes are acceptable.
- Choose **Sliding Window Log** when exact rolling-window counting is required.
- Choose **Sliding Window Counter** when memory efficiency and smoother traffic control are priorities.

---

## 6. Low-Level Design (LLD)

### 6.1 Main components

The design separates policy, algorithm selection, state management, and HTTP integration.

| Component | Responsibility |
|---|---|
| `RateLimitPolicy` | Defines the limit, time window, key strategy, and algorithm |
| `RateLimiter` | Common interface for evaluating requests |
| `TokenBucketRateLimiter` | Implements token-bucket decisions |
| `FixedWindowRateLimiter` | Implements fixed-window counting |
| `SlidingWindowLogRateLimiter` | Implements exact rolling-window counting |
| `SlidingWindowCounterRateLimiter` | Implements approximate rolling-window counting |
| `RateLimitKeyResolver` | Identifies the client and policy key |
| `RateLimitService` | Coordinates policy lookup and quota evaluation |
| `RateLimitFilter` | Integrates rate limiting into HTTP processing |
| `RateLimitStateStore` | Abstracts shared state storage |
| `RedisRateLimitStateStore` | Implements distributed state operations |
| `RateLimitDecision` | Contains the decision and remaining quota |
| `RateLimitPolicyRepository` | Loads rate-limit policies |

The leaky-bucket implementation may additionally require a bounded queue and worker component if requests are delayed rather than immediately rejected.

### 6.2 Class diagram

Image reference: `media/img/rate-limiter-lld.png`

```text
                 +----------------------+
                 |   RateLimitPolicy    |
                 +----------------------+
                           |
                           v
                 +----------------------+
                 |   RateLimitService   |
                 +----------------------+
                     |              |
                     v              v
          +------------------+  +--------------------+
          | RateLimitKey     |  | RateLimitPolicy    |
          | Resolver         |  | Repository         |
          +------------------+  +--------------------+
                     |
                     v
          +--------------------------+
          |       RateLimiter        |
          |       <<interface>>      |
          +--------------------------+
                     ^
          +----------+-----------+----------------+
          |          |           |                |
          v          v           v                v
      +--------+ +--------+ +-----------+ +---------------+
      | Token  | | Fixed  | | Sliding   | | Sliding       |
      | Bucket | | Window | | Log       | | Window Counter|
      +--------+ +--------+ +-----------+ +---------------+
                     |
                     v
          +--------------------------+
          |   RateLimitStateStore    |
          |      <<interface>>       |
          +--------------------------+
                     ^
                     |
          +--------------------------+
          | RedisRateLimitStateStore |
          +--------------------------+
```

This is a logical class diagram. An implementation can choose a different class structure, particularly when the algorithm executes entirely inside a Redis script.

### 6.3 Design patterns

**Strategy Pattern**

Each algorithm implements a common `RateLimiter` interface. The service selects an implementation based on the configured policy.

**Factory Pattern**

A factory constructs or selects the correct rate limiter for a policy.

**Repository Pattern**

A repository abstracts loading policies from configuration or persistent storage.

**Dependency Injection**

The application injects the policy repository, state store, and rate-limiting service into the HTTP filter.

These patterns reduce coupling and make it easier to test or replace individual components.

---

## 7. Java LLD: Core Abstractions

The following code provides a framework for the service layer. Algorithm-specific logic and Redis operations can be implemented behind these interfaces.

### 7.1 RateLimitPolicy

```java
public record RateLimitPolicy(
    String policyId,
    String algorithm,
    long limit,
    long windowMillis,
    String keyStrategy
) {
    public RateLimitPolicy {
        if (policyId == null || policyId.isBlank()) {
            throw new IllegalArgumentException(
                "Policy ID is required"
            );
        }

        if (limit <= 0 || windowMillis <= 0) {
            throw new IllegalArgumentException(
                "Limit and window must be positive"
            );
        }
    }
}
```

Example policy:

```java
RateLimitPolicy loginPolicy = new RateLimitPolicy(
    "login-per-user",
    "FIXED_WINDOW",
    5,
    60_000,
    "USER_ID"
);
```

### 7.2 RateLimitDecision

```java
public record RateLimitDecision(
    boolean allowed,
    long limit,
    long remaining,
    long retryAfterSeconds
) {}
```

The decision contains the result and useful information for the HTTP response.

### 7.3 RateLimiter interface

```java
public interface RateLimiter {

    RateLimitDecision evaluate(
        String key,
        RateLimitPolicy policy
    );
}
```

Implementations can use different algorithms while exposing a consistent contract.

### 7.4 RateLimitKeyResolver

```java
public interface RateLimitKeyResolver {

    String resolve(
        String authenticatedUserId,
        String clientIp,
        String endpoint
    );
}
```

A simple implementation:

```java
public final class UserAndEndpointKeyResolver
        implements RateLimitKeyResolver {

    @Override
    public String resolve(
            String authenticatedUserId,
            String clientIp,
            String endpoint) {

        String identity = authenticatedUserId != null
            ? "user:" + authenticatedUserId
            : "ip:" + clientIp;

        return identity + ":" + endpoint;
    }
}
```

In production, normalize endpoints, validate identity sources, and avoid accepting untrusted proxy headers as authoritative client IP addresses.

### 7.5 RateLimitPolicyRepository

```java
import java.util.Optional;

public interface RateLimitPolicyRepository {

    Optional<RateLimitPolicy> findPolicy(
        String endpoint,
        String httpMethod
    );
}
```

Policies may be loaded from a configuration file, database, or centralized policy service and cached locally.

### 7.6 RateLimitService

```java
public final class RateLimitService {

    private final RateLimiter rateLimiter;
    private final RateLimitKeyResolver keyResolver;
    private final RateLimitPolicyRepository policyRepository;

    public RateLimitService(
            RateLimiter rateLimiter,
            RateLimitKeyResolver keyResolver,
            RateLimitPolicyRepository policyRepository) {

        this.rateLimiter = rateLimiter;
        this.keyResolver = keyResolver;
        this.policyRepository = policyRepository;
    }

    public RateLimitDecision evaluate(
            String userId,
            String clientIp,
            String endpoint,
            String httpMethod) {

        RateLimitPolicy policy = policyRepository
            .findPolicy(endpoint, httpMethod)
            .orElseThrow(() -> new IllegalArgumentException(
                "No rate-limit policy configured"
            ));

        String key = policy.policyId() + ":" +
            keyResolver.resolve(userId, clientIp, endpoint);

        return rateLimiter.evaluate(key, policy);
    }
}
```

For a real application, handle missing policies explicitly. Depending on the API, an absent policy may mean no limit, a default limit, or a configuration error.

---

## 8. Redis-Based Distributed Rate Limiting

### 8.1 Why Redis?

An in-memory counter in one JVM cannot enforce a global limit across multiple application instances.

Consider three application servers:

```text
Application Server A: 100 requests allowed
Application Server B: 100 requests allowed
Application Server C: 100 requests allowed
```

If the intended global limit is 100 requests, independent counters can permit up to 300 requests.

A shared store allows all instances to coordinate their decisions.

Redis is a common choice because it provides:

- In-memory operations with low latency.
- Atomic commands.
- Expiration of keys.
- Lua scripting for multi-step atomic operations.
- Data structures such as strings, sorted sets, and hashes.

Redis availability, replication, failover, and consistency still need to be considered when strict quota enforcement is required.

### 8.2 Redis key design

Fixed-window example:

```text
rate-limit:{user-123}:login:29400000
```

The final component identifies the time window.

Token-bucket example:

```text
rate-limit:{user-123}:posts:tokens
rate-limit:{user-123}:posts:last-refill
```

Sliding-window-log example:

```text
rate-limit:{user-123}:search:timestamps
```

For Redis Cluster, related keys used by the same Lua script should use a common hash tag, such as `{user-123}`, to ensure they reside in the same hash slot.

### 8.3 Atomic fixed-window counter using Lua

A separate `GET`, followed by `INCR`, is insufficient for strict enforcement because multiple requests may observe the same value before updating it.

Use one atomic operation to check the limit and increment the counter.

```lua
-- KEYS[1] = rate-limit counter key
-- ARGV[1] = maximum requests
-- ARGV[2] = window duration in milliseconds

local limit = tonumber(ARGV[1])
local windowMillis = tonumber(ARGV[2])

local current = tonumber(redis.call('GET', KEYS[1]) or '0')

if current >= limit then
    return {0, current, redis.call('PTTL', KEYS[1])}
end

current = redis.call('INCR', KEYS[1])

if current == 1 then
    redis.call('PEXPIRE', KEYS[1], windowMillis)
end

return {1, current, redis.call('PTTL', KEYS[1])}
```

The script returns:

```text
{allowed, currentCount, remainingTTL}
```

The counter key must already identify the correct fixed time window. The script sets expiration when the counter is first created.

A production implementation must validate script arguments and handle unexpected Redis results. A missing or non-positive TTL should not be blindly translated into a retry interval.

### 8.4 Why Lua scripts?

A distributed request can require multiple operations:

1. Read the current counter.
2. Check the quota.
3. Increment the counter.
4. Set expiration when necessary.
5. Return the decision.

If these operations execute independently, concurrent requests may interleave and produce incorrect decisions.

A Redis Lua script executes atomically relative to other Redis commands on that server, making it suitable for implementing these state transitions.

Atomicity does not by itself solve every distributed-systems issue. Failover, replication lag, multi-region operation, and loss of recently acknowledged writes can still affect quota accuracy.

### 8.5 Spring Data Redis integration

A Spring service can execute the Lua script using `StringRedisTemplate`.

```java
import java.util.List;

import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.stereotype.Component;

@Component
public class RedisFixedWindowRateLimiter {

    private static final String SCRIPT = """
        local limit = tonumber(ARGV[1])
        local windowMillis = tonumber(ARGV[2])
        local current = tonumber(redis.call(
            'GET', KEYS[1]) or '0')

        if current >= limit then
            return {0, current, redis.call('PTTL', KEYS[1])}
        end

        current = redis.call('INCR', KEYS[1])

        if current == 1 then
            redis.call('PEXPIRE', KEYS[1], windowMillis)
        end

        return {1, current, redis.call('PTTL', KEYS[1])}
        """;

    private final StringRedisTemplate redisTemplate;

    private final DefaultRedisScript<List> script;

    public RedisFixedWindowRateLimiter(
            StringRedisTemplate redisTemplate) {

        this.redisTemplate = redisTemplate;

        this.script = new DefaultRedisScript<>();
        this.script.setScriptText(SCRIPT);
        this.script.setResultType(List.class);
    }

    public List<?> execute(
            String key,
            long limit,
            long windowMillis) {

        if (limit <= 0 || windowMillis <= 0) {
            throw new IllegalArgumentException(
                "Limit and window must be positive"
            );
        }

        return redisTemplate.execute(
            script,
            List.of(key),
            Long.toString(limit),
            Long.toString(windowMillis)
        );
    }
}
```

This example demonstrates the integration boundary. A production implementation should convert the script response into a typed `RateLimitDecision`, validate returned values, compute retry metadata, and implement an explicit Redis failure policy.

The script uses a fixed-window counter. It is not a token bucket or a sliding-window implementation.

---

## 9. Spring Boot HTTP Integration

### 9.1 Request-processing flow

Image reference: `media/img/rate-limiter-request-flow.png`

```text
HTTP Request
     |
     v
RateLimitFilter
     |
     v
Resolve Client Identity
     |
     v
Load Applicable Policy
     |
     v
Execute Rate-Limit Algorithm
     |
     +---- Allowed ----> Continue Filter Chain
     |                           |
     |                           v
     |                    Backend Controller
     |
     +---- Rejected ---> HTTP 429
```

### 9.2 RateLimitFilter

The following example assumes a configured `RateLimitService` bean is available.

```java
import java.io.IOException;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import org.springframework.web.filter.OncePerRequestFilter;

public class RateLimitFilter extends OncePerRequestFilter {

    private final RateLimitService rateLimitService;

    public RateLimitFilter(
            RateLimitService rateLimitService) {
        this.rateLimitService = rateLimitService;
    }

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        String userId = resolveAuthenticatedUserId(request);
        String clientIp = request.getRemoteAddr();

        RateLimitDecision decision =
            rateLimitService.evaluate(
                userId,
                clientIp,
                request.getRequestURI(),
                request.getMethod()
            );

        response.setHeader(
            "X-RateLimit-Limit",
            Long.toString(decision.limit())
        );

        response.setHeader(
            "X-RateLimit-Remaining",
            Long.toString(decision.remaining())
        );

        if (!decision.allowed()) {
            response.setStatus(429);
            response.setHeader(
                "Retry-After",
                Long.toString(
                    Math.max(1, decision.retryAfterSeconds())
                )
            );
            response.setContentType("application/json");
            response.getWriter().write(
                """
                {"error":"too_many_requests",
                 "message":"Rate limit exceeded"}
                """
            );
            return;
        }

        filterChain.doFilter(request, response);
    }

    private String resolveAuthenticatedUserId(
            HttpServletRequest request) {

        // Integrate with the application's authentication system.
        // Return null for unauthenticated requests if IP-based
        // limiting is the intended fallback.
        return null;
    }
}
```

This is an integration example, not a complete drop-in configuration. Register the filter as a Spring bean or through the security filter chain, and ensure it runs after authentication if the policy depends on the authenticated user.

Additional production considerations:

- Exclude health checks or internal endpoints only when appropriate.
- Handle missing policies explicitly.
- Avoid logging credentials or sensitive request information.
- Avoid applying a broad catch-all policy that accidentally bypasses security limits.
- Implement the Redis failure policy at the service boundary.
- Use the trusted client IP, not an arbitrary request header.

### 9.3 HTTP response headers

A rate-limited response can include:

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 30
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0

{
  "error": "too_many_requests",
  "message": "Rate limit exceeded"
}
```

The example uses `Retry-After` as the standard retry header. `X-RateLimit-Limit` and `X-RateLimit-Remaining` are illustrative custom headers; header naming conventions should be documented consistently across the API.

`Retry-After` should reflect the selected algorithm. For a fixed window, it can indicate the time until the window resets. For a token bucket, it can indicate when a token is expected to become available. For a queue-based leaky bucket, retry behavior depends on the queue and service policy.

---

## 10. Handling Excessive Requests

When the limit is exceeded, the system can use one of two broad strategies.

### Strategy 1: Reject the request

Return HTTP 429 and do not execute the protected operation.

This is appropriate for synchronous HTTP APIs, authentication attempts, and most public endpoints.

### Strategy 2: Queue the request

Enqueue work for later processing if the business operation supports delayed execution.

This is appropriate for some background jobs, order-processing workflows, or asynchronous tasks.

A queue must have bounded capacity, failure handling, and an explicit response contract. Queueing is not suitable for every API operation, especially when the caller expects an immediate result.

### Retry behavior

Clients should:

- Respect `Retry-After` when provided.
- Use exponential backoff for retryable failures.
- Add randomized jitter to avoid synchronized retry bursts.
- Limit the number of retries.
- Avoid retrying non-idempotent operations without an idempotency strategy.

Retries should not become a mechanism for continuously exceeding the quota.

---

## 11. Distributed-System Challenges

### 11.1 Race conditions

Suppose the current counter is 99 and the limit is 100.

Two requests arrive concurrently:

1. Both read 99.
2. Both decide that another request is allowed.
3. Both increment the counter.
4. Both requests are accepted.

The result is 101 requests, even though the limit is 100.

**Solution:** Use an atomic Redis operation, Lua script, transaction with suitable semantics, or another coordination mechanism that performs the quota decision and state update atomically.

### 11.2 Synchronization across multiple rate limiters

If several gateway instances use separate local counters, each instance sees only part of the traffic.

Possible approaches:

- Centralized Redis state.
- Sticky sessions, where appropriate but with limitations.
- Partitioned quotas with allocated local budgets.
- Approximate local limiting combined with a global enforcement layer.

A centralized store provides shared state but introduces network latency and a dependency on the store's availability.

### 11.3 Redis failure

Decide whether the system should **fail open** or **fail closed**.

| Strategy | Behavior | Trade-off |
|---|---|---|
| Fail open | Allow requests when rate-limit evaluation fails | Better availability, weaker protection |
| Fail closed | Reject requests when rate-limit evaluation fails | Stronger protection, reduced availability |

For low-risk read APIs, fail-open behavior may be acceptable for a limited period. For sensitive operations such as login protection or expensive payment-related APIs, fail-closed behavior may be appropriate.

The correct policy depends on the security, cost, and availability requirements.

### 11.4 Hot keys

A very popular user, tenant, or API key can produce heavy contention on one Redis key.

Possible mitigations include:

- Local pre-checks combined with global enforcement.
- Partitioning quotas when the business semantics permit it.
- Allocating local token budgets.
- Using dedicated infrastructure for exceptionally high-volume tenants.

Partitioning a counter without quota coordination can allow the aggregate limit to be exceeded.

### 11.5 Multi-region deployments

A globally shared quota introduces cross-region communication and consistency challenges.

Options include:

- A single authoritative region for each quota.
- Regional limits with a defined global budget.
- Distributed quota allocation.
- Approximate regional enforcement.

The appropriate choice depends on whether strict global enforcement or low latency is more important.

### 11.6 Clock behavior

Fixed and sliding windows depend on time.

- Use consistent time semantics for distributed state.
- Prefer server-side timestamps or a carefully defined clock source for shared algorithms.
- Use monotonic time for elapsed-time measurements within a process.
- Avoid assuming that clocks on different application servers are perfectly synchronized.

The local Java examples are educational implementations; distributed production versions need coordinated state transitions and appropriate time handling.

---

## 12. Performance Optimization

### 12.1 Cache policies locally

Policy configuration usually changes less frequently than request traffic.

Load policies into a local cache to avoid a database query on every request. Define a refresh or invalidation strategy so policy changes take effect within an acceptable interval.

### 12.2 Keep the request path lightweight

- Avoid database access during every rate-limit check.
- Use bounded and predictable data structures.
- Keep Redis keys short but unambiguous.
- Set appropriate expiration times.
- Avoid expensive synchronous logging for every request.
- Monitor the added latency.

### 12.3 Choose the algorithm based on the workload

A timestamp log provides accurate rolling-window enforcement but consumes more memory.

A token bucket provides efficient burst control.

A sliding window counter reduces memory consumption while approximating the rolling request count.

### 12.4 Use local budgets when justified

A gateway can allocate a portion of a global quota to individual instances. Local checks then avoid contacting Redis for every request until the allocated budget needs replenishment.

This can reduce latency and Redis load, but it complicates quota accounting. A failure or delayed budget update can affect enforcement accuracy.

### 12.5 Avoid unbounded state

Any implementation that creates one counter or bucket per client needs a cleanup strategy.

Use TTLs, eviction, bounded caches, or explicit lifecycle management. Ensure that eviction cannot silently undermine a strict quota requirement.

---

## 13. Monitoring and Observability

Monitor the following metrics:

| Metric | Purpose |
|---|---|
| Total requests evaluated | Measures rate-limiter traffic |
| Allowed requests | Measures successful quota decisions |
| Rejected requests | Identifies throttling frequency |
| Evaluation latency | Detects request-path overhead |
| Redis latency | Identifies shared-state bottlenecks |
| Redis errors and timeouts | Measures infrastructure reliability |
| Active rate-limit keys | Helps detect state growth |
| Policy cache misses | Measures configuration-cache effectiveness |
| Queue depth | Helps identify backlog in queued systems |

Use counters, histograms, and appropriately bounded labels.

Avoid using raw user IDs, API keys, or IP addresses as unbounded metric labels. Such labels can create excessive metric cardinality and expose sensitive information.

Rate-limit metrics can help determine whether policies are too strict, too permissive, or unsuitable for traffic bursts.

---

## 14. Security Considerations

Rate limiting is one layer of API protection, not a complete security solution.

- Authenticate requests before applying user-specific quotas.
- Validate client identity and proxy configuration.
- Apply separate limits to sensitive endpoints.
- Prevent attackers from bypassing limits by changing untrusted identity headers.
- Avoid exposing internal implementation details in error responses.
- Do not log authentication secrets or tokens.
- Consider account-level and IP-level limits together where appropriate.
- Use separate policies for anonymous and authenticated traffic.
- Protect Redis with authentication, network restrictions, and appropriate access controls.
- Define failure behavior explicitly so infrastructure errors do not unintentionally disable critical protections.

For login endpoints, combine rate limiting with other controls such as account lockout policies, abuse detection, and monitoring.

---

## 15. Testing Strategy

A production implementation should test both the algorithm and the distributed integration.

### Unit tests

Verify:

- Requests are allowed below the configured limit.
- Requests are rejected after the limit is reached.
- Counters or tokens recover as time advances.
- Different client keys maintain independent state.
- Different policies enforce independent quotas.
- Invalid configuration is rejected.
- Window boundaries behave as documented.

Use an injectable `Clock` for time-dependent tests instead of relying on arbitrary sleeps.

### Concurrency tests

Send many concurrent requests against one key and verify that the number of accepted requests respects the algorithm's contract.

For a strict fixed-window limit of 100, the accepted count within one window should not exceed 100 when all requests use the same authoritative state and the atomic operation is correct.

### Integration tests

Verify:

- Redis script execution.
- Key expiration.
- HTTP 429 responses.
- Response headers and retry metadata.
- Authentication and key resolution.
- Redis timeouts and fallback behavior.
- Behavior across multiple application instances.

### Load tests

Measure:

- Throughput.
- Median and tail latency.
- Redis CPU and memory.
- Rejection rates.
- Performance during traffic spikes.
- Recovery after infrastructure failures.

Test realistic traffic patterns, including bursts and many clients sharing a single hot key.

---

## 16. Hard vs. Soft Rate Limiting

### Hard rate limiting

The system does not allow requests beyond the configured threshold.

This requires a quota decision that enforces the intended limit within the chosen algorithm and consistency model.

### Soft rate limiting

The system permits temporary quota overruns or uses warnings and gradual throttling before strict enforcement.

Soft limiting can help handle temporary bursts, but it should have a defined maximum overrun and duration if resource protection is important.

---

## 17. Rate Limiting at Different Network Layers

Rate limiting can be applied at multiple layers.

- **Network layer:** IP-based controls at the network or firewall level.
- **Transport and connection layer:** Connection limits and connection-rate controls.
- **Application layer:** HTTP request limits based on identity, API, tenant, or business operation.

Application-layer rate limiting offers richer request context, while lower-layer controls can reject unwanted traffic before it consumes application resources.

These mechanisms complement each other rather than replacing one another.

---

## 18. Interview Discussion: Design Decisions

### Why not use a database for every counter?

A database can provide durable storage, but performing a database operation for every request may introduce unnecessary latency and load. An in-memory store such as Redis is often better suited to high-frequency counters with expiration.

### Why is `GET` followed by `INCR` unsafe?

The two operations are not atomic together. Concurrent requests can read the same counter and make decisions using stale values. Use an atomic operation or script for the decision and update.

### Which algorithm is best for a bursty API?

Token Bucket is a common choice because it allows a controlled burst up to the bucket capacity while maintaining an average refill rate.

### Which algorithm is most accurate for a rolling window?

Sliding Window Log provides an exact rolling-window count when timestamps and state updates are handled correctly.

### Why use a sliding window counter?

It uses substantially less memory than a timestamp log while reducing fixed-window boundary spikes. The trade-off is approximate counting.

### Why use Redis?

Redis provides fast shared state, expiration, atomic operations, and Lua scripting. It is useful when multiple gateway instances must coordinate rate-limit decisions.

### What happens if Redis is unavailable?

The system follows its configured failure policy: fail open, fail closed, or use a bounded fallback. The choice depends on security, cost, and availability requirements.

### Can a local rate limiter enforce a global quota?

Not by itself. Each application instance maintains independent state unless it uses a coordination mechanism or receives an allocated share of the global quota.

### How do you prevent a rate limiter from becoming a bottleneck?

Cache policies, minimize request-path work, use efficient atomic state operations, monitor tail latency, and consider local quota allocation where its accuracy trade-offs are acceptable.

### How should a client respond to HTTP 429?

Respect `Retry-After` when available, use bounded retries with exponential backoff and jitter, and avoid repeatedly retrying non-idempotent operations without an idempotency mechanism.

---

## 19. Final Design Recommendation

For a distributed Java and Spring Boot API platform, a practical starting design is:

1. Enforce common API policies in an API gateway or middleware.
2. Use Redis for shared rate-limit state.
3. Use an atomic Redis script for fixed-window counters.
4. Use Token Bucket when short bursts should be permitted.
5. Use Sliding Window Log when exact rolling-window counts justify the memory cost.
6. Use Sliding Window Counter when approximate rolling-window counting is sufficient.
7. Return HTTP 429 and meaningful retry metadata for rejected requests.
8. Cache policy configuration locally.
9. Define fail-open or fail-closed behavior for Redis failures.
10. Monitor request rejection rates, Redis latency, state growth, and policy effectiveness.

The final algorithm should be selected from the business requirements rather than treating one implementation as universally optimal.

---

## 20. Summary

A rate limiter protects services by controlling request frequency. The design must account for request identity, algorithm behavior, shared state, concurrency, latency, failure handling, and observability.

The five principal algorithms provide different trade-offs:

- **Token Bucket:** Efficient and burst-friendly.
- **Leaky Bucket:** Smooths processing throughput.
- **Fixed Window Counter:** Simple and memory-efficient.
- **Sliding Window Log:** Accurate rolling-window counting.
- **Sliding Window Counter:** Memory-efficient approximate rolling-window counting.

For distributed deployments, the critical design requirement is that the quota decision and state update must be coordinated correctly. Redis with atomic operations or Lua scripts is a practical implementation option, provided its availability and consistency limitations are also addressed.

A good design is not merely one that rejects excessive requests. It must do so predictably, with low latency, clear client feedback, and well-defined behavior when components fail.
