---
title: Caching Strategies
parent: System Design
nav_order: 2
---

# Caching Strategies and Eviction Policies
{: .no_toc }

Cache-aside, read-through, write-through, write-around, write-back, TTL, LRU, LFU, FIFO, and other caching techniques.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }

1. TOC
{:toc}
</details>

---

## 1. What is Caching?

**Caching** is a technique of storing frequently accessed data in a faster storage layer so that future requests can retrieve it quickly instead of accessing the original data source.

For example, an application may retrieve user details from a database and store them in Redis. Subsequent requests can retrieve the details from Redis rather than querying the database every time.

### Why do we need caching?

- **Reduce latency:** Retrieve data from memory faster than from a remote database in typical workloads.
- **Reduce database load:** Avoid repeated queries for the same data.
- **Improve throughput:** Handle more requests with available infrastructure.
- **Improve scalability:** Reduce pressure on downstream services.
- **Reduce cost:** Minimize expensive database queries and external API calls.
- **Improve resilience:** Cached data may remain available during certain downstream failures, depending on the design.

### Example without caching

```text
Client
  |
  v
Application
  |
  v
Database
  |
  v
Response
```

Every request queries the database, even when the requested data has not changed.

### Example with caching

```text
Client
  |
  v
Application
  |
  v
Cache (Redis)
  |
  +---- Cache Hit ----> Return Data
  |
  +---- Cache Miss ---> Database
                           |
                           v
                      Update Cache
                           |
                           v
                      Return Data
```

When the data is present in the cache, the application can avoid the database query.

---

## 2. Types of Caching

### 2.1 Client-side caching

The client stores responses locally and reuses them when permitted.

Examples:
- Browser cache.
- Mobile application cache.
- HTTP caching through `Cache-Control`, `ETag`, and `Last-Modified`.

**Use case:** Static assets such as images, CSS, JavaScript, and other cacheable HTTP responses.

### 2.2 Application-level caching

The application stores frequently used objects in memory or in a shared cache.

Examples:
- Java `ConcurrentHashMap`.
- Spring Cache.
- Caffeine.
- Redis.

**Use case:** User profiles, product details, configuration, and frequently accessed reference data.

### 2.3 Database caching

Frequently accessed database pages, query results, or records are retained in memory by a database engine or a separate cache layer.

**Use case:** Reducing repeated disk reads or expensive queries.

### 2.4 Distributed caching

A cache is shared by multiple application instances.

Examples:
- Redis.
- Memcached.

```text
             Load Balancer
                  |
          +-------+-------+
          |               |
          v               v
      App Server 1     App Server 2
          |               |
          +-------+-------+
                  |
                  v
             Redis Cache
                  |
                  v
               Database
```

**Use case:** Multiple Spring Boot instances need access to the same cached data.

### 2.5 Local vs. distributed cache

| Feature | Local cache | Distributed cache |
|---|---|---|
| Storage | Application process memory | Separate cache infrastructure |
| Latency | Very low | Low, but includes network overhead |
| Sharing | Usually isolated per instance | Shared across instances |
| Capacity | Limited by instance memory | Can be scaled independently |
| Invalidation | Must coordinate across instances if needed | Can use shared invalidation mechanisms |
| Failure | Lost when the process stops | Depends on cache deployment and persistence configuration |
| Examples | Caffeine, `ConcurrentHashMap` | Redis, Memcached |

A local cache is often ideal for frequently accessed, relatively stable data. A distributed cache is useful when data must be shared across multiple application instances.

---

## 3. Common Caching Strategies

Caching strategies define how data moves between the cache and the underlying data source.

### 3.1 Cache-Aside (Lazy Loading)

The application checks the cache first. If the data is missing, it loads the data from the database and populates the cache.

**Flow:**

```text
1. Application checks cache
2. Cache hit  -> Return cached data
3. Cache miss -> Query database
4. Store result in cache
5. Return data
```

**Example:**

```java
public User getUser(Long id) {
    String key = "user:" + id;

    User cachedUser = cache.get(key, User.class);

    if (cachedUser != null) {
        return cachedUser;
    }

    User user = userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));

    cache.put(key, user);

    return user;
}
```

The `cache.get` and `cache.put` methods above represent a simplified cache abstraction.

**Advantages:**
- Simple to implement.
- Only requested data is loaded into the cache.
- Cache failures can often be handled by falling back to the database.

**Disadvantages:**
- The first request after a cache miss is slower.
- Cached data can become stale.
- Concurrent misses can trigger duplicate database queries.

**Use cases:** User profiles, product catalogs, URL shorteners, and read-heavy APIs.

### 3.2 Read-Through Caching

The application requests data from the cache layer. If the data is missing, the cache layer itself loads the data from the underlying source.

```text
Application
     |
     v
Read-Through Cache
     |
     +---- Hit ----> Return Data
     |
     +---- Miss ---> Load from Database
                           |
                           v
                     Populate Cache
                           |
                           v
                     Return Data
```

**Advantages:**
- Cache loading logic is centralized.
- Application code is simpler.
- Cache population follows a consistent policy.

**Disadvantages:**
- Requires a cache implementation or provider that supports loader-based behavior.
- Cache configuration and error handling can become more complex.

**Use cases:** Systems using a cache provider with a configured loader, such as a Caffeine loading cache.

**Cache-aside vs. read-through:** With cache-aside, the application manages the miss-and-load sequence. With read-through, the cache layer manages it.

### 3.3 Write-Through Caching

Every write updates the cache and the underlying data source as part of the write operation.

```text
Application
     |
     v
Write-Through Cache
     |
     +-------> Cache
     |
     +-------> Database
```

The exact ordering and atomicity depend on the implementation. Updating two separate systems does not automatically create a distributed transaction.

**Advantages:**
- Cached values are usually fresh after successful coordinated writes.
- Read requests are less likely to encounter a cold cache after a write.

**Disadvantages:**
- Write latency may increase.
- Every write incurs cache-management overhead.
- Partial failures can leave the cache and database inconsistent.

**Use cases:** User preferences, product information, and data requiring frequently available fresh reads.

### 3.4 Write-Around Caching

Writes go directly to the database, bypassing the cache. The cache is populated later when the data is read.

```text
Write Request
      |
      v
   Database

Read Request
      |
      v
     Cache
      |
      +---- Hit ----> Return Data
      |
      +---- Miss ---> Database -> Populate Cache
```

**Advantages:**
- Avoids filling the cache with data that may never be read.
- Reduces cache churn from write-heavy workloads.

**Disadvantages:**
- The first read after a write may be slower.
- Cache population is deferred until a read occurs.

**Use cases:** Bulk imports, write-heavy datasets, and records that are infrequently read after creation.

### 3.5 Write-Back (Write-Behind) Caching

Writes initially go to the cache and are persisted to the database asynchronously or in batches.

```text
Application
     |
     v
   Cache
     |
     +----> Acknowledge Write
     |
     v
 Background Worker
     |
     v
  Database
```

**Advantages:**
- Lower write latency in suitable implementations.
- Batching can improve database write efficiency.
- Reduces the number of immediate database writes.

**Disadvantages:**
- Data can be lost if acknowledged changes are not durably recorded before a failure.
- Database state may lag behind cache state.
- Retry, ordering, and recovery logic are essential.

**Use cases:** High-throughput counters, telemetry, and workloads where delayed persistence is acceptable.

For critical data, use a durable write-ahead mechanism, a transactional outbox, or another recovery strategy rather than relying on volatile cache memory alone.

### 3.6 Read-Ahead / Refresh-Ahead Caching

The system proactively loads or refreshes data before the next request needs it.

```text
Background Refresh
        |
        v
      Cache
        |
        v
  Incoming Request
        |
        v
  Return Cached Data
```

**Advantages:**
- Can reduce cache-miss latency.
- Useful for predictable access patterns.
- Helps keep popular data warm.

**Disadvantages:**
- May load data that is never requested.
- Requires refresh scheduling and failure handling.
- Refreshing too aggressively can increase database load.

**Use cases:** Frequently accessed dashboards, scheduled reports, and predictable product or reference data.

---

## 4. Comparison of Caching Strategies

| Strategy | Read behavior | Write behavior | Main benefit |
|---|---|---|---|
| Cache-aside | Application checks cache and loads on miss | Usually updates DB and invalidates cache | Simplicity and control |
| Read-through | Cache loads missing data | Depends on write policy | Centralized cache loading |
| Write-through | Reads use cache | Updates cache and DB | Fresh cache after coordinated writes |
| Write-around | Reads use cache and load on miss | Writes directly to DB | Avoids caching unused data |
| Write-back | Reads use cache | Persists changes asynchronously | Fast writes and batching |
| Refresh-ahead | Reads usually use warmed cache | Background refreshes | Reduced miss latency |

These strategies can be combined. For example, an application may use cache-aside for reads and cache invalidation after database writes.

---

## 5. Cache Eviction Policies

An eviction policy determines which cached entry should be removed when space is needed.

Eviction is different from expiration:
- **Eviction:** Removes an entry to free capacity.
- **Expiration:** Removes or invalidates an entry because its lifetime has ended.

### 5.1 LRU (Least Recently Used)

Removes the entry that has not been accessed for the longest time.

Example:

```text
Cache capacity = 3

Cache: [A, B, C]

Access A:
Cache order: [B, C, A]

Insert D:
Evict B

Cache: [C, A, D]
```

The ordering above lists entries from least recently used to most recently used.

**Advantages:**
- Works well when recently accessed items are likely to be accessed again.
- Widely understood and commonly implemented.

**Disadvantages:**
- A one-time scan of many items can displace useful entries.
- Recency tracking adds overhead.

**Use cases:** General-purpose application caches, browser caches, and workloads with temporal locality.

### 5.2 LFU (Least Frequently Used)

Removes the entry with the lowest access frequency.

Example:

```text
Cache entries:

A -> 10 accesses
B -> 2 accesses
C -> 7 accesses

Insert D:
Evict B
```

**Advantages:**
- Preserves frequently accessed entries.
- Useful when popular items remain popular for long periods.

**Disadvantages:**
- Old popularity can dominate after traffic patterns change.
- Frequency counters and tie-breaking rules add complexity.

**Use cases:** Skewed workloads with stable popularity patterns.

### 5.3 FIFO (First In, First Out)

Removes the entry that entered the cache earliest, regardless of how often it was accessed.

```text
Insertion order:
A -> B -> C

Insert D:
Evict A
```

**Advantages:**
- Simple to implement.
- Low bookkeeping overhead.

**Disadvantages:**
- Ignores recent access and frequency.
- May evict a frequently used entry.

**Use cases:** Simple bounded queues or caches where simplicity matters more than hit ratio.

### 5.4 LIFO (Last In, First Out)

Removes the most recently inserted entry first.

**Advantages:**
- Simple ordering model.

**Disadvantages:**
- Often unsuitable for general-purpose caches.
- Can repeatedly evict newly inserted data that may soon be needed.

**Use cases:** Specialized workloads rather than typical application caches.

### 5.5 Random Replacement

Evicts a randomly selected entry.

**Advantages:**
- Simple policy with low tracking overhead.
- Can perform reasonably well for some workloads.

**Disadvantages:**
- May remove a highly valuable entry.
- Hit ratio depends on the workload and capacity.

**Use cases:** Situations where low implementation overhead is important.

### 5.6 MRU (Most Recently Used)

Evicts the entry accessed most recently.

This is the opposite of LRU.

**Advantages:**
- Can work well in specific cyclic access patterns where recently accessed items are unlikely to be needed again immediately.

**Disadvantages:**
- Often performs poorly for typical workloads with temporal locality.

**Use cases:** Specialized sequential or cyclic workloads.

### 5.7 Comparison of Eviction Policies

| Policy | Eviction rule | Tracks | Main limitation |
|---|---|---|---|
| LRU | Least recently accessed | Recency | Vulnerable to scan pollution |
| LFU | Least frequently accessed | Frequency | Old popularity may persist |
| FIFO | Earliest inserted | Insertion order | Ignores access patterns |
| LIFO | Latest inserted | Insertion order | Can discard useful new data |
| Random | Random entry | Minimal state | May evict a hot entry |
| MRU | Most recently accessed | Recency | Poor for many common workloads |

---

## 6. TTL (Time to Live) and Expiration

TTL defines how long a cache entry remains valid before it expires.

Example:

```text
Key: user:101
Value: User details
TTL: 300 seconds
```

After five minutes, the cache entry expires according to the cache's expiration semantics.

### Why use TTL?

- Prevents entries from remaining indefinitely.
- Limits the lifetime of stale values.
- Controls cache retention.
- Reduces the need for perfectly coordinated invalidation.

### Example with Redis

```text
SET user:101 "{\"name\":\"Danish\"}" EX 300
```

The entry expires after 300 seconds.

### TTL vs. eviction

| TTL | Eviction |
|---|---|
| Based on entry lifetime | Usually based on capacity pressure or policy |
| Can occur even when cache has free space | Typically occurs when the cache needs room or applies a configured limit |
| Helps bound staleness | Helps manage memory |

TTL does not guarantee that data remains fresh for the entire TTL. A write may make an entry stale before it expires, so applications often combine TTL with invalidation or versioning.

---

## 7. Cache Invalidation

Cache invalidation is the process of removing or updating cached data when the underlying data changes.

It is one of the most difficult aspects of caching because multiple application instances and cache layers may hold copies of the same value.

### 7.1 Time-based invalidation

Let entries expire after a configured TTL.

**Benefit:** Simple and limits the maximum cache-entry lifetime.

**Limitation:** Data can remain stale until expiration.

### 7.2 Explicit invalidation

Remove the cache entry after a successful database update.

```java
@Transactional
public User updateUser(Long id, UserRequest request) {
    User user = userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));

    user.setName(request.name());

    User saved = userRepository.save(user);

    cache.evict("user:" + id);

    return saved;
}
```

This example illustrates the idea, but the transaction boundary needs care. If invalidation occurs before the database transaction commits, another request can repopulate the cache with the old value. Production implementations should invalidate after commit or use an appropriate transactional coordination mechanism.

### 7.3 Write-through updates

Update the database and cache through a coordinated write path.

**Benefit:** Cached data can be updated immediately.

**Limitation:** Partial failures between the two systems still require recovery.

### 7.4 Event-driven invalidation

Publish an event after a database change.

```text
Database Update
      |
      v
 Publish Event
      |
      v
 Message Broker
      |
      v
 Cache Invalidation Consumers
      |
      v
 Remove Stale Entries
```

This approach helps coordinate invalidation across multiple application instances. Events may be delayed or duplicated, so consumers should be idempotent and the system should tolerate temporary staleness.

### 7.5 Cache stampede

A cache stampede occurs when a popular entry expires and many requests simultaneously query the database to rebuild it.

Mitigation strategies:
- Use request coalescing or single-flight loading.
- Apply a short randomized TTL variation.
- Refresh popular entries before expiration.
- Use stale-while-revalidate where permitted.
- Apply rate limits or concurrency limits to expensive loads.

### 7.6 Cache penetration

Cache penetration occurs when repeated requests for nonexistent records bypass the cache and repeatedly hit the database.

Mitigation strategies:
- Short-lived negative caching.
- Input validation.
- Rate limiting.
- Bloom filters for suitable large key spaces.

Negative caching must be invalidated when a corresponding record is created.

### 7.7 Cache avalanche

A cache avalanche occurs when many entries expire around the same time, causing a sudden spike in database traffic.

Mitigation strategies:
- Add random jitter to TTLs.
- Spread cache refreshes over time.
- Warm critical cache entries.
- Use rate limiting and controlled database fallbacks.

---

## 8. Spring Boot Caching

Spring provides an abstraction that allows application methods to use different cache providers.

### 8.1 Enable caching

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 8.2 Use `@Cacheable`

Caches a method result and returns the cached result for subsequent calls with the same cache key.

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Cacheable(value = "users", key = "#id")
    public User getUser(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}
```

The first successful invocation loads the user from the repository. Subsequent cache hits can return the cached value without executing the method body.

### 8.3 Use `@CachePut`

Always invokes the method and updates the cache with its result.

```java
import org.springframework.cache.annotation.CachePut;
import org.springframework.stereotype.Service;

@Service
public class UserUpdateService {

    @CachePut(value = "users", key = "#result.id")
    public User updateUser(User user) {
        return saveUserToDatabase(user);
    }

    private User saveUserToDatabase(User user) {
        // Persist the updated user using the repository.
        return user;
    }
}
```

The persistence method above is illustrative; a real implementation must use the actual repository and transaction configuration.

### 8.4 Use `@CacheEvict`

Removes an entry from the cache.

```java
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.stereotype.Service;

@Service
public class UserDeletionService {

    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        // Delete the user from the database.
    }
}
```

Replace the comment with the actual repository deletion logic. Ensure that cache eviction happens consistently with the database transaction.

### 8.5 Spring caching annotations

| Annotation | Purpose |
|---|---|
| `@EnableCaching` | Enables Spring's caching abstraction |
| `@Cacheable` | Reads a cached result or invokes the method on a miss |
| `@CachePut` | Invokes the method and updates the cache |
| `@CacheEvict` | Removes one or more cache entries |
| `@Caching` | Combines multiple cache operations |

**Important:** Spring caching annotations do not automatically guarantee distributed consistency, a particular TTL, or transaction atomicity. These depend on the cache provider and application design.

---

## 9. Choosing a Caching Strategy

| Scenario | Possible choice | Reason |
|---|---|---|
| Frequently read user profiles | Cache-aside + TTL | Simple and effective for read-heavy data |
| Popular product catalog | Cache-aside or read-through | Reuse frequently accessed product data |
| Data requiring fresh reads after updates | Write-through or update plus invalidation | Coordinates cache updates with writes |
| Bulk ingestion with infrequent reads | Write-around | Avoids caching unused data |
| High-volume aggregations | Write-back with durable recovery | Supports batching and asynchronous persistence |
| Frequently accessed stable configuration | Local cache or refresh-ahead | Low latency and predictable refresh |
| Shared cache across multiple instances | Redis or Memcached | Centralized cache access |
| Cache with limited memory | LRU, LFU, or provider-specific policy | Controls capacity based on workload |

There is no universally best caching strategy. Select one based on read/write ratios, freshness requirements, failure behavior, and workload measurements.

---

## 10. Common Interview Questions

### Q1. What is the difference between cache-aside and read-through?

In cache-aside, the application explicitly loads data on a cache miss. In read-through, the cache layer manages the loading operation.

### Q2. What is the difference between write-through and write-back?

Write-through updates the cache and data source as part of the write path. Write-back acknowledges or retains writes in the cache before persisting them asynchronously, depending on the implementation.

### Q3. What is the difference between write-through and write-around?

Write-through writes through the cache layer and updates the underlying store. Write-around sends writes directly to the underlying store and typically populates the cache on a later read.

### Q4. Which is better: LRU or LFU?

Neither is universally better. LRU is effective when recent access predicts future access. LFU can be effective when frequently accessed items remain popular over time.

### Q5. Why is cache invalidation difficult?

The same data may exist in several cache layers and application instances. Updates can race with cache reads, events may be delayed, and failures can leave stale entries.

### Q6. What happens when Redis goes down?

The application may fall back to the database if its capacity permits. Without protection, a cache outage can overload the database. Use timeouts, bounded fallbacks, concurrency controls, and monitoring.

### Q7. What is a cache stampede?

Many requests simultaneously attempt to load the same missing or expired entry, creating a burst of downstream traffic.

### Q8. What is negative caching?

Negative caching stores the result of a failed lookup, such as a missing record, for a limited period to avoid repeated expensive queries.

### Q9. Can a cache replace a database?

Usually not for authoritative business data. A cache is generally optimized for speed, while the database provides durable storage and authoritative records. Some systems use durable distributed stores with caching characteristics, but the durability and consistency guarantees must be evaluated explicitly.

### Q10. How do you improve the cache hit ratio?

Cache frequently accessed data, choose appropriate TTLs and capacity, avoid unnecessary invalidations, prevent scan pollution where possible, and monitor actual access patterns.

### Q11. Does TTL guarantee fresh data?

No. TTL bounds an entry's lifetime according to the configured expiration semantics. Data can become stale before the TTL expires.

### Q12. What is the difference between eviction and invalidation?

Eviction removes entries to manage cache capacity. Invalidation removes or refreshes entries because the underlying data has changed or the cached value is no longer valid.

---

## 11. Best Practices

1. Cache only data that benefits from reuse.
2. Measure cache hit ratio, latency, memory usage, and database load.
3. Set TTLs based on freshness requirements.
4. Use explicit invalidation for data that changes frequently.
5. Design for Redis or cache-provider failure.
6. Protect the database from stampedes and unbounded fallback traffic.
7. Use bounded cache sizes and suitable eviction policies.
8. Avoid caching sensitive information without appropriate security controls.
9. Consider cache consistency across application instances.
10. Test expiration, concurrent misses, invalidation races, and provider outages.

---

## 12. Summary

Caching improves application performance by retaining frequently accessed data in a faster storage layer.

The most important concepts are:

- **Caching strategies:** Cache-aside, read-through, write-through, write-around, write-back, and refresh-ahead.
- **Eviction policies:** LRU, LFU, FIFO, LIFO, random replacement, and MRU.
- **Expiration:** TTL controls how long a cache entry remains valid.
- **Invalidation:** Keeps cached values aligned with authoritative data.
- **Failure management:** Prevents cache outages and stampedes from overwhelming downstream systems.

For many Java and Spring Boot applications, **cache-aside with Redis, an appropriate TTL, and deliberate invalidation** is a strong starting point. Use other strategies when measured workload characteristics and consistency requirements justify them.
