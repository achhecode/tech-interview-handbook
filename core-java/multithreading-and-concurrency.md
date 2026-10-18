---
title: Multithreading and Concurrency
parent: Core Java
nav_order: 7
---

# Multithreading and Concurrency
{: .no_toc }

Threads, locks, executors and the Java Memory Model.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---


# Multithreading in Java: Thread Pools, Lifecycle, Deadlocks, and Thread Dumps

## 1. What Is Multithreading in Java?

**Multithreading** is a programming technique in which multiple threads execute concurrently within the same process.

A thread is an independent path of execution. Threads within a process share resources such as heap memory, but each thread has its own execution stack and program state.

### Why Do We Need Multithreading?

- **Responsiveness:** Keep an application responsive while background tasks execute.
- **Concurrency:** Handle multiple tasks during overlapping time periods.
- **Parallelism:** Execute tasks simultaneously on multiple CPU cores when possible.
- **Throughput:** Process more requests or jobs over time.
- **Resource utilization:** Perform useful work while other tasks wait for I/O.

### Example: Creating Threads

```java
public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(() ->
                System.out.println("Task 1: " +
                        Thread.currentThread().getName()));

        Thread t2 = new Thread(() ->
                System.out.println("Task 2: " +
                        Thread.currentThread().getName()));

        t1.start();
        t2.start();
    }
}
```

**Possible output:**

```text
Task 1: Thread-0
Task 2: Thread-1
```

The order is not guaranteed. The thread scheduler determines when the threads execute.

**Important:** Calling `start()` schedules a thread for execution. Calling `run()` directly executes the method on the current thread like an ordinary method call.

## 2. What Is the Difference Between Concurrency and Parallelism?

| Concurrency | Parallelism |
|---|---|
| Multiple tasks make progress over overlapping periods | Multiple tasks execute simultaneously |
| Can occur on a single CPU core through scheduling | Requires simultaneous execution resources, such as multiple CPU cores |
| Focuses on coordinating multiple tasks | Focuses on executing multiple tasks at the same time |

Java multithreading supports concurrency and can provide parallelism when the hardware and execution environment allow it.

## 3. What Is a Thread Pool in Java?

A **thread pool** is a group of reusable worker threads that execute submitted tasks.

Instead of creating a new thread for every task, an application submits tasks to an executor, which manages worker threads and task execution.

Java provides the `Executor`, `ExecutorService`, and `ScheduledExecutorService` APIs for this purpose.

### Why Do We Need Thread Pools?

1. Reduce the overhead of repeatedly creating and destroying threads.
2. Limit the number of concurrently executing tasks.
3. Reuse worker threads.
4. Manage task submission and completion.
5. Support graceful shutdown and task cancellation.
6. Improve resource management and application scalability.

### Example: Creating and Using a Thread Pool

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolDemo {
    public static void main(String[] args) {
        ExecutorService executor =
                Executors.newFixedThreadPool(3);

        try {
            for (int i = 1; i <= 6; i++) {
                int taskId = i;

                executor.submit(() -> {
                    System.out.println(
                            "Task " + taskId + " executed by "
                            + Thread.currentThread().getName());
                });
            }
        } finally {
            executor.shutdown();
        }
    }
}
```

**Possible output:**

```text
Task 1 executed by pool-1-thread-1
Task 2 executed by pool-1-thread-2
Task 3 executed by pool-1-thread-3
Task 4 executed by pool-1-thread-1
Task 5 executed by pool-1-thread-2
Task 6 executed by pool-1-thread-3
```

The order may vary. Three worker threads process six tasks, reusing workers as tasks finish.

### Important ExecutorService Methods

| Method | Purpose |
|---|---|
| `execute(task)` | Submits a `Runnable` without returning a task result |
| `submit(task)` | Submits a task and returns a `Future` |
| `shutdown()` | Stops accepting new tasks and allows submitted tasks to finish |
| `shutdownNow()` | Attempts to interrupt active tasks and returns tasks that never started |
| `awaitTermination()` | Waits for termination for a specified duration |

Calling `shutdown()` does not immediately terminate active tasks.

### Production Consideration

`Executors.newFixedThreadPool(n)` uses an unbounded work queue. If tasks arrive faster than workers can process them, queued tasks can accumulate and consume substantial memory.

For production workloads, consider a configured `ThreadPoolExecutor` with a bounded queue, a deliberate rejection policy, and appropriate thread and queue sizes.

## 4. How Is a Thread Pool Different From a Database Connection Pool?

These are two different kinds of resource pools.

- A **thread pool** reuses worker threads to execute tasks.
- A **database connection pool** reuses database connections to avoid repeatedly establishing expensive database sessions.

A thread pool does not create a database connection pool.

### Database Connection Pool Example Using Apache Commons DBCP

Add Apache Commons DBCP to the project using your build tool, then configure a `BasicDataSource`.

```java
import org.apache.commons.dbcp2.BasicDataSource;

import java.sql.Connection;
import java.sql.SQLException;

public class DatabasePoolDemo {
    public static void main(String[] args) throws SQLException {
        BasicDataSource dataSource = new BasicDataSource();

        dataSource.setUrl(
                "jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("user");
        dataSource.setPassword("password");

        dataSource.setInitialSize(5);
        dataSource.setMaxTotal(20);

        try (Connection connection =
                     dataSource.getConnection()) {
            System.out.println("Connection acquired");
            // Execute database operations here.
        } finally {
            dataSource.close();
        }
    }
}
```

The MySQL JDBC driver must also be available on the classpath.

### How Does the Connection Pool Work?

1. The application requests a connection using `dataSource.getConnection()`.
2. The pool provides an available connection or creates one if permitted.
3. The application executes SQL operations.
4. Closing the connection returns it to the pool rather than necessarily closing the underlying physical connection.
5. The pool can reuse that connection for another request.

In a Spring Boot application, a connection pool such as HikariCP is commonly configured through the `DataSource` abstraction.

### How Should Thread Pools and Connection Pools Be Sized?

The two pool sizes must be considered together.

For example, if an application has 100 request-processing threads but only 10 database connections, requests that need database access may have to wait for connections.

Increasing the number of threads without considering database capacity can increase contention rather than improve throughput.

Choose pool sizes based on workload, database limits, request latency, and performance measurements.

## 5. What Is the Lifecycle of a Thread in Java?

Java defines six thread states through the `Thread.State` enum.

| State | Meaning |
|---|---|
| `NEW` | Thread has been created but `start()` has not been called |
| `RUNNABLE` | Thread is eligible to run or is running; Java combines these concepts into one state |
| `BLOCKED` | Thread is waiting to acquire an intrinsic monitor lock |
| `WAITING` | Thread is waiting indefinitely for another thread's action |
| `TIMED_WAITING` | Thread is waiting for a limited period |
| `TERMINATED` | Thread has completed execution |

### Thread Lifecycle Diagram

```text
         NEW
          |
        start()
          |
          v
      RUNNABLE <-------------------+
          |                        |
          | Waiting for monitor    |
          v                        |
       BLOCKED --------------------+
          |
          | Acquires monitor
          v
      RUNNABLE
          |
          +---- wait()/join() ----> WAITING
          |                            |
          +---- sleep()/timed wait -> TIMED_WAITING
          |                            |
          +<------ condition met ------+
          |
          | run() completes
          v
      TERMINATED
```

This is a simplified illustration. Actual transitions depend on the operation involved; for example, `wait()` releases the relevant monitor, while `sleep()` does not release a monitor already held.

### Example: Observing Thread States

```java
public class ThreadLifecycleDemo {
    public static void main(String[] args)
            throws InterruptedException {

        Thread thread = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        System.out.println(thread.getState()); // NEW

        thread.start();

        Thread.sleep(100);

        System.out.println(thread.getState());
        // Usually TIMED_WAITING

        thread.join();

        System.out.println(thread.getState()); // TERMINATED
    }
}
```

**Important:** Thread states are snapshots. A state observed at one instant may change immediately afterward.

### What Is the Lifecycle of a Thread-Pool Worker?

A pool worker generally follows this conceptual cycle:

1. The executor creates a worker when needed.
2. The worker waits for a task.
3. It executes the task.
4. It returns to waiting for another task.
5. It may be removed when the executor shuts down or when pool-management rules require it.

These are conceptual worker activities, not additional values of `Thread.State`.

## 6. Why Is a Thread Pool Needed in Multithreading?

Without a pool, an application might create one thread for every incoming request or task.

For example, creating thousands of platform threads unnecessarily can increase memory use, scheduling overhead, and contention.

A thread pool provides:

- Controlled concurrency.
- Worker reuse.
- Task queuing.
- Centralized task lifecycle management.
- A way to limit resource consumption.

However, a thread pool does not automatically make code thread-safe. Shared mutable state still requires appropriate synchronization or concurrency-safe data structures.

### What About Virtual Threads?

Java 21 finalized virtual threads. They are lightweight threads designed to make high-concurrency, especially blocking I/O, applications easier to write.

For many I/O-bound tasks, virtual threads can be a better choice than pooling a small number of platform threads solely to avoid thread-creation costs.

```java
try (var executor =
         java.util.concurrent.Executors.newVirtualThreadPerTaskExecutor()) {

    executor.submit(() -> {
        System.out.println("Running in a virtual thread");
    });
}
```

Virtual threads do not make CPU-intensive work faster by themselves. CPU-bound workloads still need suitable concurrency limits based on available processing resources.

## 7. What Is a Deadlock in Java?

A **deadlock** occurs when two or more threads wait indefinitely for resources held by one another, so none can proceed.

A common scenario involves threads acquiring multiple locks in different orders.

### Example of Deadlock

```java
public class DeadlockDemo {
    private static final Object LOCK_A = new Object();
    private static final Object LOCK_B = new Object();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            synchronized (LOCK_A) {
                System.out.println("Thread 1 acquired A");

                synchronized (LOCK_B) {
                    System.out.println("Thread 1 acquired B");
                }
            }
        });

        Thread t2 = new Thread(() -> {
            synchronized (LOCK_B) {
                System.out.println("Thread 2 acquired B");

                synchronized (LOCK_A) {
                    System.out.println("Thread 2 acquired A");
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```

If thread 1 acquires `LOCK_A` while thread 2 acquires `LOCK_B`, both can wait indefinitely for the lock held by the other.

The deadlock depends on scheduling; the example does not guarantee that it will occur on every run.

### Four Conditions for Deadlock

The classic Coffman conditions are:

1. **Mutual exclusion:** A resource can be held exclusively.
2. **Hold and wait:** A thread holds one resource while waiting for another.
3. **No preemption:** A resource cannot simply be taken away from its holder.
4. **Circular wait:** A cycle exists in which each participant waits for a resource held by the next.

All four conditions must hold for this classic resource deadlock scenario.

### How to Prevent Deadlocks

- Acquire multiple locks in a consistent global order.
- Avoid unnecessary nested locking.
- Keep synchronized sections short.
- Use `Lock.tryLock()` with a timeout when suitable.
- Avoid calling unknown or blocking code while holding locks.
- Inspect thread dumps when the application becomes unresponsive.

Example of using a consistent lock order:

```java
synchronized (LOCK_A) {
    synchronized (LOCK_B) {
        // Work that requires both locks
    }
}
```

If all code acquires these locks in the same order, the circular-wait condition is avoided for this lock pair.

## 8. What Is a Thread Dump in Java?

A **thread dump** is a snapshot of the JVM's threads, including their states and stack traces. It is useful for investigating:

- Deadlocks.
- Threads blocked on locks.
- Excessive waiting.
- Application hangs.
- Request-processing bottlenecks.
- Potential thread starvation.

A thread dump is particularly useful when an application appears unresponsive but has not crashed.

### How to Take a Thread Dump

First, find the Java process ID:

```bash
jcmd -l
```

Then request a thread dump:

```bash
jcmd <pid> Thread.print
```

Alternatively, use:

```bash
jstack <pid>
```

Replace `<pid>` with the actual Java process ID. These tools are generally available with a suitable JDK installation, and permissions or runtime conditions can affect whether the command succeeds.

### How to Analyze a Thread Dump

1. Capture multiple dumps several seconds apart when investigating a persistent issue.
2. Find threads in `BLOCKED`, `WAITING`, or `TIMED_WAITING` states.
3. Read the stack traces to identify what each thread is doing.
4. Examine lock ownership and dependencies.
5. Look for repeated stack traces across dumps.
6. Identify whether the issue is a deadlock, lock contention, slow external dependency, or normal waiting.
7. Correlate the findings with CPU, memory, application logs, and request latency.

**Important:** A `WAITING` thread is not necessarily problematic. Threads often wait normally for queue items, task completion, or other events.

### What Does a Deadlock Look Like in a Thread Dump?

A JVM thread dump may contain a deadlock report identifying threads that hold locks needed by other threads.

For example, one thread may be shown holding `LOCK_A` and waiting for `LOCK_B`, while another holds `LOCK_B` and waits for `LOCK_A`.

This ownership cycle is strong evidence of a monitor deadlock.

## 9. What Is a Race Condition?

A **race condition** occurs when the result of a program depends on the timing or interleaving of concurrent operations.

Example:

```java
class Counter {
    int count = 0;

    void increment() {
        count++;
    }
}
```

If two threads execute `increment()` simultaneously, updates can be lost because `count++` is a read-modify-write operation rather than one atomic operation.

### Solution Using synchronized

```java
class Counter {
    private int count = 0;

    synchronized void increment() {
        count++;
    }

    synchronized int getCount() {
        return count;
    }
}
```

The intrinsic monitor ensures that only one thread at a time executes these synchronized instance methods on the same `Counter` object.

For a simple numeric counter, `AtomicInteger` is another option:

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
    private final AtomicInteger count = new AtomicInteger();

    void increment() {
        count.incrementAndGet();
    }

    int getCount() {
        return count.get();
    }
}
```

Choose the mechanism based on whether the operation involves one variable or a larger shared-state invariant.

## 10. Thread Pool vs Database Connection Pool

| Feature | Thread pool | Database connection pool |
|---|---|---|
| Resource managed | Worker threads | Database connections |
| Main purpose | Execute tasks concurrently | Reuse connections to the database |
| Typical Java API | `ExecutorService` | `DataSource` implementations |
| Common bottleneck | CPU, scheduling, queue capacity | Database capacity, connection acquisition, query latency |
| Resource returned | Worker becomes available for another task | Connection is returned to the pool when closed |

These pools solve different problems, but both should be sized and monitored according to the application's workload.

## 11. Common Interview Questions

### Q1. What is the difference between `start()` and `run()`?

`start()` initiates a new thread of execution. Calling `run()` directly executes its code on the current thread.

### Q2. Can we restart a terminated thread?

No. A Java `Thread` instance can be started only once. Starting it again throws `IllegalThreadStateException`. Create a new thread or submit another task to an executor.

### Q3. What is the difference between `sleep()` and `wait()`?

`Thread.sleep()` pauses the current thread for a duration without releasing monitors it holds. `Object.wait()` waits for notification or timeout and releases the monitor associated with that object while waiting.

### Q4. Is `BLOCKED` the same as `WAITING`?

No. `BLOCKED` specifically means waiting to acquire an intrinsic monitor lock. `WAITING` means waiting for another thread's action without a timeout.

### Q5. Does a thread pool guarantee that tasks execute in submission order?

No. Execution order depends on the executor, scheduling, and task durations. A single-thread executor executes tasks sequentially; general multi-threaded pools do not guarantee completion order.

### Q6. What is thread starvation?

Thread starvation occurs when a thread cannot obtain the CPU time, locks, connections, or other resources it needs to make progress. It differs from deadlock because progress may still be possible for other threads.

### Q7. How can you diagnose high CPU usage?

Capture repeated thread dumps, identify threads that remain in CPU-intensive stack traces, and correlate them with per-thread CPU measurements using suitable operating-system or profiling tools. A thread dump alone is not a direct measurement of CPU consumption.

### Q8. What is the difference between a deadlock and a livelock?

- **Deadlock:** Threads are blocked waiting for resources and cannot proceed.
- **Livelock:** Threads continue executing and reacting to each other but make no useful progress.

## 12. One-Line Interview Answer

Multithreading in Java enables concurrent execution through multiple threads; thread pools manage reusable workers, synchronization protects shared state, thread dumps help diagnose runtime problems, and deadlock prevention requires careful resource and lock management.



---

# Deadlock Detection and Prevention in Java

## 1. How Do You Check Whether a Java Application Has a Deadlock?

A **deadlock** occurs when two or more threads are permanently blocked because each is waiting for a resource held by another thread in the same dependency cycle.

The most practical way to detect a JVM monitor deadlock is to capture a **thread dump** and inspect the locks, thread states, and ownership relationships.

### Method 1: Use `jstack`

First, find the Java process ID:

```bash
jcmd -l
```

Then capture a thread dump:

```bash
jstack <pid>
```

Alternatively:

```bash
jcmd <pid> Thread.print
```

Replace `<pid>` with the Java process ID.

Look for output resembling:

```text
Found one Java-level deadlock:

"Thread-1":
  waiting to lock monitor 0x... (object 0x...)
  which is held by "Thread-2"

"Thread-2":
  waiting to lock monitor 0x... (object 0x...)
  which is held by "Thread-1"
```

The exact output depends on the JVM version and lock type. The important evidence is a circular dependency: each thread waits for a lock held by another thread in the cycle.

**Interview point:** The JVM can detect many intrinsic-monitor and ownable-synchronizer deadlocks, but not every possible application-level deadlock or resource dependency.

### Method 2: Use `ThreadMXBean` Programmatically

Java provides the `ThreadMXBean` management API to inspect thread information and detect deadlocks.

```java
import java.lang.management.ManagementFactory;
import java.lang.management.ThreadMXBean;
import java.util.Arrays;

public class DeadlockDetector {

    public static void main(String[] args) {
        ThreadMXBean threadMXBean =
                ManagementFactory.getThreadMXBean();

        long[] deadlockedThreads =
                threadMXBean.findDeadlockedThreads();

        if (deadlockedThreads == null) {
            System.out.println("No deadlock detected.");
            return;
        }

        System.out.println("Deadlock detected!");

        Arrays.stream(deadlockedThreads)
                .forEach(id -> {
                    var info =
                            threadMXBean.getThreadInfo(id);

                    if (info != null) {
                        System.out.println(
                                "Thread: " + info.getThreadName());
                        System.out.println(
                                "State: " + info.getThreadState());
                        System.out.println(
                                "Lock: " + info.getLockName());
                        System.out.println(
                                "Lock owner: "
                                        + info.getLockOwnerName());
                    }
                });
    }
}
```

**How it works:**

1. `ManagementFactory.getThreadMXBean()` obtains the JVM's thread-management interface.
2. `findDeadlockedThreads()` checks for deadlocks involving object monitors and ownable synchronizers, such as `ReentrantLock`.
3. If a deadlock is found, the method returns the IDs of threads involved.
4. If none is found, it returns `null`.

`findMonitorDeadlockedThreads()` checks only monitor-based deadlocks. Prefer `findDeadlockedThreads()` when you also want to detect ownable synchronizer deadlocks.

This is a snapshot, not a guarantee that the application is free from all deadlocks. Recheck if the problem is intermittent.

### Method 3: Use Profiling and Monitoring Tools

Tools such as Java Flight Recorder, VisualVM, JConsole, and commercial Java profilers can help investigate thread behavior, lock contention, and application hangs.

For production troubleshooting, capture multiple thread dumps several seconds apart. Comparing them helps distinguish a persistent deadlock from ordinary waiting or slow operations.

## 2. What Are the Symptoms of a Deadlock?

| Symptom | What to investigate |
|---|---|
| Application hangs | Requests stop completing or threads remain stuck |
| Requests time out | Request-processing threads may be waiting for locks or other resources |
| Threads remain `BLOCKED` | Inspect the monitor lock each thread is waiting to acquire |
| Circular lock dependencies | One thread waits for a lock held by another, which waits for a lock held by the first |
| High CPU usage | Possible in some related concurrency problems, but not a defining symptom of deadlock |
| Database or connection-pool exhaustion | Threads may be waiting for connections or holding resources while waiting for other resources |

**Important:** Deadlock does not necessarily cause high CPU usage. A typical deadlocked thread is blocked and may consume very little CPU. High CPU usage may instead indicate a busy loop, livelock, or CPU-intensive task.

Also, `WAITING` and `TIMED_WAITING` states alone do not prove a deadlock. Thread state, lock ownership, and the dependency cycle must be examined together.

## 3. How Do You Prevent Deadlocks in Java?

### Technique 1: Acquire Locks in a Consistent Order

This is one of the most effective prevention techniques.

**Risky approach:** Two threads acquire the same locks in opposite orders.

```java
// Thread 1
synchronized (lockA) {
    synchronized (lockB) {
        // Perform work
    }
}

// Thread 2
synchronized (lockB) {
    synchronized (lockA) {
        // Perform work
    }
}
```

If each thread acquires its first lock before the other thread does, both may wait indefinitely.

**Safer approach:** Every thread acquires the locks in the same order.

```java
// Both threads follow the same lock order.
synchronized (lockA) {
    synchronized (lockB) {
        // Perform work requiring both locks
    }
}
```

For this pair of locks, consistently acquiring `lockA` before `lockB` eliminates the circular-wait condition.

In a larger application, define a global lock ordering and follow it consistently across all code paths.

### Technique 2: Use Timed Lock Acquisition

The `Lock` interface supports `tryLock()`, which allows a thread to avoid waiting indefinitely for a lock.

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class TimedLockExample {

    private static final Lock lockA = new ReentrantLock();
    private static final Lock lockB = new ReentrantLock();

    static boolean performTask() throws InterruptedException {
        if (!lockA.tryLock(500, TimeUnit.MILLISECONDS)) {
            return false;
        }

        try {
            if (!lockB.tryLock(500, TimeUnit.MILLISECONDS)) {
                return false;
            }

            try {
                System.out.println("Task executed successfully");
                return true;
            } finally {
                lockB.unlock();
            }
        } finally {
            lockA.unlock();
        }
    }
}
```

**Why is this safer?**

- The thread does not wait indefinitely for either lock.
- If the second lock cannot be acquired, the first lock is released.
- The calling code can retry, fail gracefully, or report the problem.

This example avoids indefinite waiting in this lock-acquisition path, but it does not guarantee that the overall application is deadlock-free. Retries must be designed carefully to avoid livelock.

### Technique 3: Avoid Unnecessary Nested Locks

Nested locking increases the number of possible lock dependencies.

Instead of holding one lock while acquiring another, consider whether the code can:

- Perform independent work outside synchronized sections.
- Copy the required state while holding a lock, then release it.
- Use a single lock for a shared invariant.
- Redesign the data structures to reduce shared mutable state.

Do not remove synchronization blindly: doing so can introduce race conditions or inconsistent data.

### Technique 4: Use Higher-Level Concurrency Utilities

Java provides utilities that can simplify coordination:

| Utility | Purpose |
|---|---|
| `ExecutorService` | Manages asynchronous task execution |
| `ConcurrentHashMap` | Supports concurrent map operations |
| `BlockingQueue` | Coordinates producers and consumers |
| `Semaphore` | Controls access to a limited number of permits |
| `CountDownLatch` | Allows threads to wait for a count to reach zero |
| `ReentrantLock` | Provides explicit locking, timed acquisition, and interruptible acquisition |

These tools can reduce the need for manually coordinated locking, but they do not automatically eliminate deadlocks. For example, two threads can still deadlock if they acquire multiple locks in inconsistent orders.

### Technique 5: Keep Critical Sections Short

Avoid performing slow network calls, database queries, or lengthy computations while holding locks unless necessary.

Long critical sections increase contention and the chance that one thread holds a resource while waiting for another.

### Technique 6: Test and Monitor Concurrent Code

- Write repeatable tests for concurrent operations.
- Use stress tests to exercise different execution interleavings.
- Inspect thread dumps when tests hang.
- Monitor request latency, lock contention, connection-pool wait time, and thread counts.
- Use profilers and Java Flight Recorder to investigate production incidents.

Tools such as JUnit and JMeter are useful in testing, but they do not independently prove that an application is deadlock-free.

## 4. What Is the Difference Between Deadlock and Livelock?

A **livelock** occurs when threads remain active and keep responding to each other's actions, but they fail to make useful progress.

<text> </text>

| Characteristic | Deadlock | Livelock |
|---|---|---|
| Definition | Threads wait indefinitely for resources held in a dependency cycle | Threads keep changing state in response to each other without completing useful work |
| Thread activity | Often blocked or waiting | Often runnable and repeatedly executing |
| CPU usage | Can be low | Can be high, though not always |
| Progress | No participating thread progresses | Threads are active, but the intended task does not progress |
| Typical cause | Circular lock dependency | Repeated retries, yielding, or overly cooperative reactions |
| Prevention | Consistent lock ordering, reducing nested locks, timed acquisition | Backoff, randomized retry delays, bounded retries, and better coordination |

### Deadlock Example

- Thread A holds lock X and waits for lock Y.
- Thread B holds lock Y and waits for lock X.
- Neither can continue.

### Livelock Example

Imagine two threads that repeatedly attempt to acquire two resources. Whenever each notices that the other is also trying, both release their resources to be cooperative. They then retry simultaneously and repeat the same behavior.

Neither thread is permanently blocked, but neither finishes the operation.

**How to prevent livelock:** Introduce randomized backoff, vary retry timing, use bounded retries, or establish a priority rule so both threads do not repeat the same reaction indefinitely.

## 5. Deadlock vs Starvation

These terms describe different concurrency problems.

- **Deadlock:** Threads in a dependency cycle cannot proceed.
- **Starvation:** A thread repeatedly fails to obtain CPU time or a resource because other threads keep getting access.
- **Livelock:** Threads remain active but repeatedly fail to make useful progress.

A program may exhibit more than one of these problems at the same time.

## 6. Practical Troubleshooting Checklist

When a Java application appears hung:

1. Find the JVM process using `jcmd -l`.
2. Capture a thread dump using `jcmd <pid> Thread.print`.
3. Capture another dump after a short interval.
4. Look for a JVM deadlock report and circular lock dependencies.
5. Inspect stack traces, lock owners, and thread states.
6. If no deadlock is reported, investigate slow I/O, connection-pool exhaustion, starvation, and long-running tasks.
7. Correct the underlying resource dependency or synchronization design.
8. Retest under concurrent load and monitor the application afterward.

## 7. Common Interview Questions

### Q1. Can Java detect a deadlock automatically?

The JVM can detect certain deadlocks, and tools such as `jstack` and `ThreadMXBean` can report them. Detection does not automatically resolve the problem.

### Q2. Does using `tryLock()` prevent every deadlock?

No. Timed acquisition can prevent indefinite waiting along a particular acquisition path. Retrying carelessly can create livelock, and other resources or dependencies may still cause deadlocks.

### Q3. Does a deadlock always cause high CPU usage?

No. Deadlocked threads are commonly blocked and consume little CPU. High CPU usage may indicate livelock or another performance problem.

### Q4. Can `synchronized` cause a deadlock?

Yes. Intrinsic locks provide mutual exclusion, but acquiring multiple monitors in inconsistent orders can produce a deadlock.

### Q5. What is the best way to prevent deadlocks?

Design resource acquisition carefully. A consistent global lock order is especially effective for preventing circular waits. Combine it with short critical sections, suitable concurrency utilities, and diagnostic monitoring.

## 8. One-Line Interview Answer

To detect deadlocks in Java, inspect thread dumps or use `ThreadMXBean.findDeadlockedThreads()`; prevent them through consistent lock ordering, reduced nested locking, timed lock acquisition, and careful concurrency design. A deadlock blocks progress, whereas a livelock keeps threads active without useful progress.


---

# Ways to Create Threads in Java and Why Prefer ExecutorService?

## 1. What is a thread?

A thread is an independent path of execution within a process. Java supports multithreading, allowing multiple tasks to make progress concurrently.

Java provides several ways to define and execute work on threads.

## 2. Way 1: Extend the Thread class

Create a class that extends `Thread` and override its `run()` method.

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println(
            "Running on: " + Thread.currentThread().getName()
        );
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();
    }
}
```

Possible output:

```text
Running on: Thread-0
```

The actual thread name can vary.

**How it works:**

- `run()` contains the task's logic.
- `start()` asks the JVM to start a new thread.
- The JVM schedules the thread, and its execution invokes `run()`.

**Disadvantages:**

- Java supports single class inheritance, so extending `Thread` prevents extending another class.
- Task logic becomes coupled to thread management.
- Creating a new thread for every task can be expensive.

Use this approach mainly for simple demonstrations or when extending `Thread` is specifically justified.

## 3. Way 2: Implement the Runnable interface

`Runnable` separates the task from the thread that executes it.

```java
class MyTask implements Runnable {
    @Override
    public void run() {
        System.out.println(
            "Running on: " + Thread.currentThread().getName()
        );
    }
}

public class Main {
    public static void main(String[] args) {
        Runnable task = new MyTask();

        Thread thread = new Thread(task);
        thread.start();
    }
}
```

A lambda expression can make this shorter:

```java
public class Main {
    public static void main(String[] args) {
        Runnable task = () ->
            System.out.println("Task executing");

        new Thread(task).start();
    }
}
```

**Advantages:**

- Separates task logic from thread management.
- Allows the class to extend another class.
- Makes the task easier to submit to an executor.

**Limitation:** `Runnable.run()` does not return a result and cannot declare checked exceptions in its method signature.

## 4. Way 3: Implement Callable and use ExecutorService

`Callable<V>` is useful when a task needs to return a result or throw a checked exception.

Unlike `Runnable`, `Callable` cannot be passed directly to the `Thread` constructor. It is typically submitted to an `ExecutorService`.

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class Main {
    public static void main(String[] args)
            throws InterruptedException, ExecutionException {

        ExecutorService executor =
            Executors.newSingleThreadExecutor();

        try {
            Callable<Integer> task = () -> 10 + 20;

            Future<Integer> future = executor.submit(task);

            System.out.println(future.get());
        } finally {
            executor.shutdown();
        }
    }
}
```

Output:

```text
30
```

**How it works:**

1. `Callable` defines the computation.
2. `ExecutorService` schedules and executes the task.
3. `submit()` returns a `Future`.
4. `future.get()` retrieves the result, waiting if necessary.

This is a common pattern in production Java applications.

## 5. Way 4: Use ExecutorService with Runnable

You can submit `Runnable` tasks to an executor without creating a `Thread` manually.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        ExecutorService executor =
            Executors.newFixedThreadPool(2);

        try {
            executor.submit(() ->
                System.out.println(
                    Thread.currentThread().getName() + " - Task 1"
                )
            );

            executor.submit(() ->
                System.out.println(
                    Thread.currentThread().getName() + " - Task 2"
                )
            );
        } finally {
            executor.shutdown();
        }
    }
}
```

Possible output:

```text
pool-1-thread-1 - Task 1
pool-1-thread-2 - Task 2
```

The order and thread names are not guaranteed. Both tasks may also execute sequentially if scheduling or timing causes that outcome.

The executor creates or reuses worker threads according to its configuration.

## 6. Way 5: Use CompletableFuture

`CompletableFuture` is useful for asynchronous computation, chaining dependent tasks, combining results, and handling exceptions.

```java
import java.util.concurrent.CompletableFuture;

public class Main {
    public static void main(String[] args) {
        CompletableFuture<Integer> future =
            CompletableFuture.supplyAsync(() -> 10)
                .thenApply(number -> number * 2)
                .thenApply(number -> number + 5);

        System.out.println(future.join());
    }
}
```

Output:

```text
25
```

**How it works:**

1. `supplyAsync()` starts an asynchronous computation.
2. The first `thenApply()` multiplies the result by two.
3. The second `thenApply()` adds five.
4. `join()` waits for the final result.

By default, `supplyAsync()` uses the common `ForkJoinPool`. For production workloads, you can supply a dedicated executor when isolation, resource control, or workload characteristics require it.

## 7. Comparison of approaches

| Approach | Returns a result? | Creates/manages threads? | Best use |
|---|---|---|---|
| Extend `Thread` | No | You manage the thread | Simple demonstrations or specialized thread behavior |
| `Runnable` + `Thread` | No | You manage the thread | Simple independent task |
| `Callable` + `ExecutorService` | Yes | Executor manages execution | Background task with a result |
| `Runnable` + `ExecutorService` | Not from `Runnable` itself | Executor manages execution | Managing many independent tasks |
| `CompletableFuture` | Yes, through completion stages | Uses a default or supplied execution mechanism | Asynchronous pipelines and task composition |

Note that `Runnable` and `Callable` define tasks, whereas `Thread`, `ExecutorService`, and `CompletableFuture` serve different execution or result-management roles.

## 8. Why prefer ExecutorService?

### 1. Thread reuse

Creating a new operating-system-backed thread for every small task introduces overhead.

An executor can reuse worker threads from a pool.

Without a pool:

```java
for (int i = 0; i < 1000; i++) {
    new Thread(() -> processTask()).start();
}
```

This can create a large number of threads and consume significant resources.

With an executor:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(10);

for (int i = 0; i < 1000; i++) {
    executor.submit(() -> processTask());
}

executor.shutdown();
```

The pool limits the number of concurrently executing worker threads to ten. Pending tasks may wait in the executor's work queue.

**Production caution:** `newFixedThreadPool()` uses an unbounded work queue. For workloads that can submit tasks faster than they complete, consider a suitably configured `ThreadPoolExecutor` with a bounded queue and an explicit rejection policy.

### 2. Better resource management

An executor provides control over the number of worker threads, task submission, and shutdown.

You can select an execution strategy that matches the workload instead of creating threads indiscriminately.

### 3. Task results

`submit(Callable)` returns a `Future`, which lets you retrieve a result or attempt cancellation.

```java
Future<Integer> future =
    executor.submit(() -> 100 + 200);

Integer result = future.get();
```

### 4. Exception handling

If a task submitted using `submit()` fails, its exception is captured in the `Future`. The exception is generally observed when calling `get()`, through an `ExecutionException`.

If you submit a `Runnable` using `execute()`, an uncaught exception from the task is handled by the thread's uncaught-exception mechanism; it is not captured in a returned `Future`.

### 5. Lifecycle management

`ExecutorService` supports explicit shutdown.

```java
executor.shutdown();
```

This stops accepting new tasks while allowing previously submitted tasks to finish.

To wait for termination:

```java
executor.shutdown();

if (!executor.awaitTermination(30,
        java.util.concurrent.TimeUnit.SECONDS)) {
    executor.shutdownNow();
}
```

`shutdownNow()` requests interruption of running tasks and attempts to stop queued tasks from starting. It does not guarantee that every running task terminates.

## 9. Choosing the right executor

| Executor | Typical use |
|---|---|
| `newSingleThreadExecutor()` | Sequential execution of tasks on one worker |
| `newFixedThreadPool(n)` | A fixed number of worker threads |
| `newCachedThreadPool()` | Many short-lived asynchronous tasks; thread growth must be considered |
| `newScheduledThreadPool(n)` | Delayed or recurring tasks |
| Custom `ThreadPoolExecutor` | Explicit control over pool size, queue, thread factory, and rejection policy |

Choose the pool size and queue based on the workload, resource limits, latency requirements, and whether tasks are CPU-bound or I/O-bound. No single pool configuration is best for every application.

Modern Java also supports virtual threads, introduced as a preview feature in Java 19 and finalized in Java 21. They are particularly useful for applications with many concurrent blocking I/O tasks. They do not eliminate the need to manage other scarce resources, such as database connections.

## 10. Common interview questions

### Q1. Which is better: extending Thread or implementing Runnable?

Implementing `Runnable` is generally preferred because it separates the task from the thread and preserves the option to extend another class.

### Q2. Can we call run() instead of start()?

Yes, but calling `run()` directly executes the method on the current thread. Calling `start()` initiates a new thread of execution.

### Q3. Why not create a new Thread for every task?

It adds thread-creation overhead and can exhaust system resources. An executor can reuse threads and control concurrency.

### Q4. Is ExecutorService itself a thread?

No. It is an abstraction for managing task execution. Implementations may use worker threads, pools, or other execution strategies.

### Q5. What is the difference between execute() and submit()?

- `execute(Runnable)` submits a task without returning a `Future`.
- `submit(Runnable)` or `submit(Callable)` returns a `Future` that represents task completion and, where applicable, its result.

### Q6. Does ExecutorService guarantee parallel execution?

No. Parallelism depends on the executor configuration, available resources, and task scheduling. A single-thread executor executes submitted tasks sequentially.

### Q7. When should we use CompletableFuture instead?

Use `CompletableFuture` when you need to compose asynchronous computations, combine results, or handle completion and errors through stages. For straightforward background tasks, `ExecutorService` with `Runnable` or `Callable` may be simpler.

## 11. Quick revision

- `Thread` is a mechanism for running code on a thread.
- `Runnable` represents a task with no return value.
- `Callable` represents a task that returns a value and can throw checked exceptions.
- `Future` provides access to a task's eventual result.
- `ExecutorService` manages task execution and worker resources.
- `CompletableFuture` supports asynchronous composition and completion stages.
- Prefer executors over manually creating many threads for ordinary application workloads.

---

---
title: volatile vs synchronized vs AtomicInteger in Java
parent: Core Java
nav_order: 17
---

# volatile vs synchronized vs AtomicInteger in Java

## 1. What is the difference?

Java provides several mechanisms to coordinate access to shared data between multiple threads. `volatile`, `synchronized`, and `AtomicInteger` solve related but different problems.

| Feature | `volatile` | `synchronized` | `AtomicInteger` |
|---|---|---|---|
| Primary purpose | Visibility and ordering | Mutual exclusion and visibility | Atomic operations on an integer |
| Visibility guarantee | Yes | Yes, through monitor synchronization | Yes, through atomic/concurrency guarantees |
| Atomic increment (`++`) | No | Yes, when the complete operation is synchronized consistently | Yes, using `incrementAndGet()` |
| Allows only one thread in a critical section | No | Yes, for the same monitor | Not a general-purpose locking mechanism |
| Can protect multiple variables together | No, not by itself | Yes | No, not as a single atomic transaction across separate variables |
| Blocking | No intrinsic lock acquisition | Threads may block waiting for a monitor | Ordinary atomic operations are generally non-blocking |
| Typical use | Stop flags, status flags | Multi-step shared-state updates | Counters, sequences, atomic numeric updates |

## 2. volatile keyword

The `volatile` keyword ensures that reads and writes of a variable participate in Java's memory-visibility and ordering guarantees.

When one thread writes to a volatile variable, a subsequent read of that variable by another thread observes the write or a later write according to the Java Memory Model.

### Example: visibility problem

```java
class Worker {
    private volatile boolean running = true;

    public void work() {
        while (running) {
            // Perform work
        }

        System.out.println("Worker stopped");
    }

    public void stop() {
        running = false;
    }
}
```

Usage:

```java
Worker worker = new Worker();

Thread thread = new Thread(worker::work);
thread.start();

worker.stop();
```

Because `running` is volatile, the worker thread will observe the updated flag and can exit the loop.

Without `volatile` or another appropriate synchronization mechanism, the worker may continue to observe a stale value.

### Important limitation: volatile does not make compound operations atomic

```java
class Counter {
    volatile int count = 0;

    void increment() {
        count++;
    }
}
```

This code is **not thread-safe** when multiple threads increment `count` concurrently.

Why?

The expression `count++` involves three conceptual steps:

1. Read the current value.
2. Add one.
3. Write the new value.

Two threads can read the same value and overwrite each other's updates.

For example, if `count` is 5, both threads might read 5 and write 6. The final value is 6 instead of 7.

**Use `volatile` when you need visibility for an individual variable, not atomic read-modify-write operations.**

## 3. synchronized keyword

The `synchronized` keyword provides mutual exclusion and memory-visibility guarantees through a monitor lock.

Only one thread at a time can hold the same monitor. Other threads trying to enter a synchronized region guarded by that monitor must wait.

### Example: thread-safe counter

```java
class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}
```

Usage:

```java
Counter counter = new Counter();

Runnable task = () -> {
    for (int i = 0; i < 100_000; i++) {
        counter.increment();
    }
};

Thread t1 = new Thread(task);
Thread t2 = new Thread(task);

t1.start();
t2.start();

try {
    t1.join();
    t2.join();
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}

System.out.println(counter.getCount());
```

Output:

```text
200000
```

Both threads use the same `Counter` instance and therefore the same monitor for its synchronized instance methods. Each increment is protected from concurrent increments by another thread using that monitor.

### Types of synchronized methods and blocks

**Synchronized instance method**

```java
public synchronized void update() {
    // Locks this object
}
```

**Synchronized static method**

```java
public static synchronized void updateGlobalState() {
    // Locks the Class object for this class
}
```

**Synchronized block**

```java
public void update() {
    synchronized (lock) {
        // Critical section
    }
}
```

A synchronized block can protect a smaller section of code. All threads accessing the same shared state must consistently use the appropriate lock.

### Advantages

- Protects compound operations and invariants.
- Provides visibility between threads using the same monitor.
- Can coordinate updates to multiple variables under one lock.
- Supports monitor-based coordination using `wait()`, `notify()`, and `notifyAll()`.

### Disadvantages

- Contending threads may block.
- Incorrect locking can cause deadlocks.
- Large critical sections can reduce concurrency.

## 4. AtomicInteger

`AtomicInteger` belongs to `java.util.concurrent.atomic`. It supports atomic operations on an `int`, including increments, decrements, compare-and-set, and updates.

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
    private final AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet();
    }

    public int getCount() {
        return count.get();
    }
}
```

Usage:

```java
Counter counter = new Counter();

Runnable task = () -> {
    for (int i = 0; i < 100_000; i++) {
        counter.increment();
    }
};

Thread t1 = new Thread(task);
Thread t2 = new Thread(task);

t1.start();
t2.start();

try {
    t1.join();
    t2.join();
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}

System.out.println(counter.getCount());
```

Output:

```text
200000
```

### Important AtomicInteger methods

| Method | Purpose |
|---|---|
| `get()` | Reads the current value |
| `set(value)` | Sets the value |
| `incrementAndGet()` | Increments and returns the new value |
| `getAndIncrement()` | Returns the old value, then increments |
| `decrementAndGet()` | Decrements and returns the new value |
| `addAndGet(delta)` | Adds a value and returns the new value |
| `compareAndSet(expected, update)` | Updates only if the current value equals the expected value |
| `updateAndGet(function)` | Atomically applies an update function and returns the result |

### Understanding compareAndSet()

```java
AtomicInteger value = new AtomicInteger(10);

boolean updated = value.compareAndSet(10, 20);

System.out.println(updated);
System.out.println(value.get());
```

Output:

```text
true
20
```

The update succeeds because the value was 10 when the comparison occurred. If another thread had already changed it, the operation would fail and return `false`.

`AtomicInteger` commonly uses compare-and-set and related atomic primitives. Its operations are designed to be thread-safe without an explicit Java monitor for each operation, although this does not make every sequence of operations atomic.

## 5. Why AtomicInteger is not always enough

Consider this code:

```java
if (count.get() < 100) {
    count.incrementAndGet();
}
```

Each atomic operation is thread-safe, but the **combined check-and-increment is not atomic**. Multiple threads can observe a value below 100 and then increment it, potentially exceeding the limit.

If the condition and update must be one indivisible operation, use a compare-and-set loop:

```java
import java.util.concurrent.atomic.AtomicInteger;

class LimitedCounter {
    private final AtomicInteger count = new AtomicInteger();

    public boolean tryIncrement() {
        while (true) {
            int current = count.get();

            if (current >= 100) {
                return false;
            }

            if (count.compareAndSet(current, current + 1)) {
                return true;
            }
        }
    }
}
```

Alternatively, use `synchronized` if the operation involves more complex shared state or multiple variables.

## 6. Which one should you choose?

<box gap={3}>
  <row align=start gap={3}>
    <box background="surface-secondary" radius="lg" size="44px" align=center justify=center>
      <icon name="eye" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **Use `volatile`**

      When one thread updates a flag and other threads need to see the update, such as a shutdown flag. It is not enough for a shared counter that uses `++`.
    </box>
  </row>
  <divider color="subtle" />
  <row align=start gap={3}>
    <box background="surface-secondary" radius="lg" size="44px" align=center justify=center>
      <icon name="lock-keyhole" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **Use `synchronized`**

      When several statements, variables, or business rules must be protected together as one critical section.
    </box>
  </row>
  <divider color="subtle" />
  <row align=start gap={3}>
    <box background="surface-secondary" radius="lg" size="44px" align=center justify=center>
      <icon name="zap" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **Use `AtomicInteger`**

      When you need a thread-safe integer counter, sequence value, or conditional numeric update without explicitly locking a monitor.
    </box>
  </row>
</box>

## 7. Common interview questions

### Q1. Is volatile thread-safe?

It provides visibility and ordering guarantees for accesses to the variable, but does not make compound operations such as `count++` atomic.

### Q2. Is synchronized reentrant?

Yes. A thread that already owns an object's monitor can acquire that same monitor again. This is called reentrant locking.

### Q3. Is AtomicInteger always faster than synchronized?

No. Performance depends on contention, the operation, and the workload. Atomic operations are often efficient for simple counters, while synchronized code may be clearer or more suitable for compound state changes. Measure performance rather than assuming one is always faster.

### Q4. Can we use volatile with AtomicInteger?

An `AtomicInteger` variable reference can itself be declared `volatile`, but that is usually unnecessary for a `final` field initialized once. The atomic object's methods already provide the relevant thread-safety guarantees for its integer value.

### Q5. What is the difference between incrementAndGet() and getAndIncrement()?

```java
AtomicInteger number = new AtomicInteger(5);

System.out.println(number.incrementAndGet()); // 6
System.out.println(number.getAndIncrement()); // 6
System.out.println(number.get());             // 7
```

- `incrementAndGet()` increments first, then returns the new value.
- `getAndIncrement()` returns the old value, then increments.

### Q6. Can AtomicInteger protect two variables together?

No. Each atomic operation applies to its own value. For a consistent update involving multiple variables, use a suitable lock or redesign the state into a single atomic representation if appropriate.

### Q7. What is the difference between AtomicInteger and LongAdder?

`AtomicInteger` provides an exact, immediately readable integer value and supports compare-and-set operations. `LongAdder` is often better for highly contended statistical counters, but its `sum()` is not an atomic snapshot of all concurrent updates.

## 8. Quick revision

- `volatile` = visibility and ordering, not compound-operation atomicity.
- `synchronized` = mutual exclusion plus visibility through a monitor.
- `AtomicInteger` = atomic integer operations such as increment and compare-and-set.
- Use a lock when multiple state changes must be coordinated together.
- Use the simplest mechanism that correctly protects the shared-state invariant.


---

# Runnable vs Callable vs Future vs CompletableFuture

## 1. Introduction

Java provides several interfaces and classes for executing tasks asynchronously and retrieving their results.

The four concepts are related but serve different purposes:

- **Runnable:** Represents a task that does not return a result.
- **Callable:** Represents a task that returns a result and can throw checked exceptions.
- **Future:** Represents the result of an asynchronous computation and allows you to wait for completion.
- **CompletableFuture:** Supports asynchronous computation, chaining, combining tasks, callbacks, and exception handling.

## 2. Key differences

| Feature | Runnable | Callable | Future | CompletableFuture |
|---|---|---|---|---|
| Type | Interface | Interface | Interface | Class implementing `Future` and `CompletionStage` |
| Returns a result | No | Yes | Retrieves a task's result | Yes, can represent a result |
| Checked exceptions | Cannot declare checked exceptions in `run()` | Can declare checked exceptions in `call()` | `get()` can throw `ExecutionException` and `InterruptedException` | Supports exceptional completion and recovery stages |
| Asynchronous execution | Can be submitted to an executor | Can be submitted to an executor | Represents a result; does not itself execute a task | Can run or represent asynchronous computations |
| Chaining | No built-in chaining | No built-in chaining | No built-in chaining | Yes |
| Combine tasks | No built-in support | No built-in support | No built-in composition | Yes |
| Callback support | No built-in support | No built-in support | No built-in completion callbacks | Yes |
| Cancellation | Depends on how it is executed | Depends on how it is executed | Supports cancellation | Supports cancellation, with different semantics |

## 3. Runnable

`Runnable` represents a task that does not return a result.

It has one abstract method:

```java
void run();
```

### Example

```java
public class Main {
    public static void main(String[] args) {
        Runnable task = () ->
            System.out.println("Task executed");

        Thread thread = new Thread(task);
        thread.start();
    }
}
```

Possible output:

```text
Task executed
```

The task runs on a separate thread, and the caller does not receive a result from `run()`.

A `Runnable` cannot declare checked exceptions in its method signature. It can still throw unchecked exceptions.

### Runnable with ExecutorService

```java
ExecutorService executor =
    Executors.newSingleThreadExecutor();

try {
    Future<?> future = executor.submit(() -> {
        System.out.println("Processing task");
    });

    future.get(); // Wait for task completion
} finally {
    executor.shutdown();
}
```

When a `Runnable` is submitted using `submit()`, the executor returns a `Future<?>`. Its result is normally `null` when the task completes successfully.

## 4. Callable

`Callable` represents a task that returns a value and can throw checked exceptions.

Its abstract method is:

```java
V call() throws Exception;
```

### Example

```java
import java.util.concurrent.Callable;

public class Main {
    public static void main(String[] args) throws Exception {
        Callable<Integer> task = () -> {
            int a = 10;
            int b = 20;

            return a + b;
        };

        System.out.println(task.call());
    }
}
```

Output:

```text
30
```

Here, `call()` is invoked directly, so the example runs synchronously. Submitting the task to an executor allows it to run asynchronously.

### Callable with ExecutorService and Future

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args)
            throws InterruptedException, ExecutionException {

        ExecutorService executor =
            Executors.newSingleThreadExecutor();

        try {
            Callable<Integer> task = () -> 10 + 20;

            Future<Integer> future = executor.submit(task);

            Integer result = future.get();

            System.out.println(result);
        } finally {
            executor.shutdown();
        }
    }
}
```

Output:

```text
30
```

**Important:** `future.get()` blocks the calling thread until the result is available, unless the computation has already completed.

## 5. Future

`Future` represents the result of a computation that may not have completed yet.

It is commonly returned by `ExecutorService.submit()`.

Important methods:

| Method | Purpose |
|---|---|
| `get()` | Waits for completion and retrieves the result |
| `get(timeout, unit)` | Waits up to a specified time |
| `isDone()` | Checks whether the task has completed |
| `isCancelled()` | Checks whether the task was cancelled |
| `cancel(mayInterruptIfRunning)` | Attempts to cancel the task |

### Example

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args)
            throws InterruptedException, ExecutionException {

        ExecutorService executor =
            Executors.newSingleThreadExecutor();

        try {
            Future<Integer> future = executor.submit(() -> {
                Thread.sleep(1000);
                return 100;
            });

            System.out.println(future.isDone());

            Integer result = future.get();

            System.out.println(result);
            System.out.println(future.isDone());
        } finally {
            executor.shutdown();
        }
    }
}
```

Output:

```text
false
100
true
```

The first `isDone()` will normally return `false` because the task sleeps for one second. The `get()` call waits for the result.

### Limitations of Future

Traditional `Future` has some limitations:

- `get()` can block the calling thread.
- It does not provide built-in methods for chaining dependent computations.
- Combining multiple futures requires additional coordination.
- It does not provide the rich callback and recovery stages available with `CompletableFuture`.

## 6. CompletableFuture

`CompletableFuture`, introduced in Java 8, implements both `Future` and `CompletionStage`.

It supports asynchronous tasks and lets you build a pipeline of dependent computations.

Its capabilities include:

- Transforming a result with `thenApply()`.
- Executing an action with `thenAccept()`.
- Chaining asynchronous tasks with `thenCompose()`.
- Combining independent computations with `thenCombine()`.
- Handling errors with `exceptionally()` and `handle()`.
- Waiting for multiple computations with `allOf()` or `anyOf()`.

### Example 1: Supply an asynchronous result

```java
import java.util.concurrent.CompletableFuture;

public class Main {
    public static void main(String[] args) {
        CompletableFuture<Integer> future =
            CompletableFuture.supplyAsync(() -> 10 + 20);

        System.out.println(future.join());
    }
}
```

Output:

```text
30
```

`supplyAsync()` runs a supplier asynchronously using the default executor unless an executor is supplied. For this simple example, `join()` waits for completion and retrieves the result.

Unlike `Future.get()`, `join()` does not declare checked exceptions. If the computation fails, it can throw an unchecked `CompletionException`.

### Example 2: Chain operations

```java
import java.util.concurrent.CompletableFuture;

public class Main {
    public static void main(String[] args) {
        CompletableFuture<Integer> future =
            CompletableFuture.supplyAsync(() -> 10)
                .thenApply(number -> number * 2)
                .thenApply(number -> number + 5);

        System.out.println(future.join());
    }
}
```

Output:

```text
25
```

Execution flow:

1. `supplyAsync()` produces `10`.
2. `thenApply()` multiplies it by `2`, producing `20`.
3. The next `thenApply()` adds `5`, producing `25`.
4. `join()` waits for the final result.

The dependent stages are triggered by completion of the preceding stages. The non-async variants such as `thenApply()` do not guarantee execution on a separate thread.

### Example 3: Handle exceptions

```java
import java.util.concurrent.CompletableFuture;

public class Main {
    public static void main(String[] args) {
        CompletableFuture<Integer> future =
            CompletableFuture.supplyAsync(() -> {
                if (true) {
                    throw new RuntimeException("Something went wrong");
                }

                return 100;
            }).exceptionally(ex -> {
                System.out.println("Error: " + ex.getMessage());
                return 0;
            });

        System.out.println(future.join());
    }
}
```

The exact exception message may be wrapped in a `CompletionException`, so the printed message can vary by how the exception is propagated.

The final result is:

```text
0
```

`exceptionally()` recovers from an exceptional completion by supplying a replacement result.

## 7. thenApply() vs thenCompose() vs thenCombine()

These methods are common interview questions.

| Method | Purpose |
|---|---|
| `thenApply()` | Transforms a result into another value |
| `thenCompose()` | Chains a function that returns another `CompletionStage`, flattening the result |
| `thenCombine()` | Combines the results of two independent stages |

### thenApply()

```java
CompletableFuture<Integer> future =
    CompletableFuture.supplyAsync(() -> 10)
        .thenApply(number -> number * 2);
```

Result: `20`.

### thenCompose()

```java
CompletableFuture<Integer> future =
    CompletableFuture.supplyAsync(() -> 10)
        .thenCompose(number ->
            CompletableFuture.supplyAsync(() -> number * 2)
        );
```

Result: `20`.

`thenCompose()` avoids producing a nested `CompletableFuture<CompletableFuture<Integer>>`.

### thenCombine()

```java
CompletableFuture<Integer> first =
    CompletableFuture.supplyAsync(() -> 10);

CompletableFuture<Integer> second =
    CompletableFuture.supplyAsync(() -> 20);

CompletableFuture<Integer> combined =
    first.thenCombine(second, Integer::sum);

System.out.println(combined.join());
```

Output:

```text
30
```

Both computations can proceed independently, and their results are combined when both are complete.

## 8. Runnable vs Callable vs Future vs CompletableFuture: when to use each?

| Requirement | Best fit |
|---|---|
| Run a task without returning a result | `Runnable` |
| Run a task that returns a result | `Callable` |
| Wait for or cancel an executor-submitted task | `Future` |
| Chain asynchronous operations | `CompletableFuture` |
| Combine asynchronous results | `CompletableFuture` |
| Handle asynchronous failures in a pipeline | `CompletableFuture` |

## 9. Common interview questions

**Q1. Can Runnable return a value?**

No. Its `run()` method returns `void`. Use `Callable` when the task needs to return a value.

**Q2. Can Callable throw checked exceptions?**

Yes. Its `call()` method declares `throws Exception`.

**Q3. Is CompletableFuture a replacement for Future?**

It extends the `Future` interface and adds composition and completion capabilities. It is useful for asynchronous pipelines, but a traditional `Future` can still be appropriate for simple executor tasks.

**Q4. Does CompletableFuture always create a new thread?**

No. The non-async dependent methods may execute in the thread that completes the preceding future or another thread calling a completion method. Async methods use a default executor or a supplied executor.

**Q5. What is the difference between get() and join()?**

Both wait for completion. `get()` declares checked exceptions such as `InterruptedException` and `ExecutionException`. `join()` throws unchecked `CompletionException` for exceptional completion.

**Q6. Does Future.get() make a task asynchronous?**

No. The executor or another mechanism runs the task. `Future` provides a way to observe its completion and retrieve its result.

## 10. Quick revision

- `Runnable`: task, no result.
- `Callable`: task, result, checked exceptions supported.
- `Future`: handle for retrieving and managing a computation's outcome.
- `CompletableFuture`: composable asynchronous result with callbacks and error handling.
- `get()` blocks and declares checked exceptions.
- `join()` blocks and uses unchecked exception propagation.
- `thenApply()` transforms; `thenCompose()` chains; `thenCombine()` combines.

---

---
title: Fail-Fast vs Fail-Safe Iterators in Java
parent: Core Java
nav_order: 18
---

# Fail-Fast vs Fail-Safe Iterators in Java

## 1. What is an iterator?

An `Iterator` in Java is used to traverse elements in a collection one by one.

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorExample {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>(
            List.of("Ali", "Danish", "Sara")
        );

        Iterator<String> iterator = names.iterator();

        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

Output:

```text
Ali
Danish
Sara
```

When a collection is modified during iteration, the iterator's behavior depends on the collection and its iterator implementation.

## 2. Fail-fast iterators

A fail-fast iterator attempts to detect certain structural modifications to its underlying collection outside the iterator's own supported modification methods. It may throw `ConcurrentModificationException` when it detects such a modification.

Common examples include iterators from:

- `ArrayList`
- `HashSet`
- `HashMap`

### Example

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class FailFastExample {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>(
            List.of(10, 20, 30, 40)
        );

        Iterator<Integer> iterator = numbers.iterator();

        while (iterator.hasNext()) {
            Integer number = iterator.next();

            if (number == 20) {
                numbers.remove(number);
            }

            System.out.println(number);
        }
    }
}
```

This code can throw `ConcurrentModificationException` because the list is modified directly while its iterator is in use. The exception is not guaranteed to occur at every possible modification point; fail-fast behavior is best-effort.

### Correct approach: use Iterator.remove()

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class SafeRemovalExample {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>(
            List.of(10, 20, 30, 40)
        );

        Iterator<Integer> iterator = numbers.iterator();

        while (iterator.hasNext()) {
            Integer number = iterator.next();

            if (number == 20) {
                iterator.remove();
            }
        }

        System.out.println(numbers);
    }
}
```

Output:

```text
[10, 30, 40]
```

`Iterator.remove()` performs the removal through the iterator's supported API, so it does not trigger the same concurrent-modification problem.

## 3. Fail-safe iterators

“Fail-safe” is an informal term often used for iterators that tolerate modifications to the underlying collection without throwing `ConcurrentModificationException` for those modifications.

However, these iterators do not all work the same way. Some use a snapshot; others provide weakly consistent traversal.

### A. Snapshot-based iterator: CopyOnWriteArrayList

`CopyOnWriteArrayList` creates a new underlying array when it is modified. Its iterator traverses a snapshot of the array taken when the iterator was created.

```java
import java.util.Iterator;
import java.util.concurrent.CopyOnWriteArrayList;

public class SnapshotIteratorExample {
    public static void main(String[] args) {
        CopyOnWriteArrayList<Integer> numbers =
            new CopyOnWriteArrayList<>(
                java.util.List.of(10, 20, 30)
            );

        Iterator<Integer> iterator = numbers.iterator();

        numbers.add(40);

        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }

        System.out.println("Current list: " + numbers);
    }
}
```

Output:

```text
10
20
30
Current list: [10, 20, 30, 40]
```

The iterator does not see `40` because it traverses the snapshot created before `40` was added.

**Trade-off:** Snapshot-based iteration is useful for read-heavy, concurrent workloads, but copying the underlying array on writes can be expensive. The iterator also does not support `remove()`.

### B. Weakly consistent iterator: ConcurrentHashMap

`ConcurrentHashMap` iterators can tolerate concurrent updates without throwing `ConcurrentModificationException`. They are weakly consistent: they may reflect some modifications made after iteration begins, but do not promise a snapshot of the entire map.

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.Iterator;
import java.util.Map;

public class WeaklyConsistentExample {
    public static void main(String[] args) {
        Map<Integer, String> map = new ConcurrentHashMap<>();

        map.put(1, "Ali");
        map.put(2, "Danish");

        Iterator<Integer> iterator = map.keySet().iterator();

        map.put(3, "Sara");

        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

The iterator can complete without throwing `ConcurrentModificationException`. Whether it observes the newly added key `3` is not guaranteed.

## 4. Difference between fail-fast and fail-safe iterators

| Feature | Fail-fast | “Fail-safe” (informal term) |
|---|---|---|
| Concurrent modification | Attempts to detect certain modifications | Designed to tolerate concurrent updates |
| `ConcurrentModificationException` | May throw it when a modification is detected | Does not throw it for tolerated concurrent updates |
| View of data | Usually traverses the underlying collection | May use a snapshot or weakly consistent traversal |
| Always reflects latest changes? | No | Not necessarily |
| Examples | `ArrayList`, `HashMap` iterators | `CopyOnWriteArrayList`, `ConcurrentHashMap` iterators |

## 5. Common interview questions

### Q1. Does fail-fast mean thread-safe?

No. Fail-fast behavior does not make a collection thread-safe. It is primarily a bug-detection mechanism, and its detection is best-effort.

### Q2. Is ConcurrentModificationException thrown only when multiple threads modify a collection?

No. It can occur even in a single thread when the collection is structurally modified directly while an iterator is active.

### Q3. Does CopyOnWriteArrayList provide a snapshot iterator?

Yes. Its iterator traverses a snapshot of the underlying array as it existed when the iterator was created.

### Q4. Does ConcurrentHashMap use snapshot iterators?

No. Its iterators are weakly consistent rather than snapshot-based. They tolerate concurrent updates and may reflect some updates made during traversal.

### Q5. How can we remove elements while iterating over an ArrayList?

Use `Iterator.remove()` when appropriate, or use `removeIf()`:

```java
List<Integer> numbers =
    new ArrayList<>(List.of(10, 20, 30, 40));

numbers.removeIf(number -> number == 20);

System.out.println(numbers);
```

Output:

```text
[10, 30, 40]
```

### Q6. What is the difference between fail-fast and weakly consistent iteration?

Fail-fast iterators attempt to detect certain unexpected modifications. Weakly consistent iterators allow concurrent updates and may reflect some of them without promising a fixed snapshot.

## 6. Quick revision

- **Fail-fast:** Attempts to detect unexpected structural modifications; may throw `ConcurrentModificationException`.
- **Snapshot-based:** Iterates over a fixed view captured when the iterator is created; example: `CopyOnWriteArrayList`.
- **Weakly consistent:** Tolerates concurrent updates and may reflect some of them; example: `ConcurrentHashMap`.
- Fail-fast behavior is best-effort, not a guarantee of concurrent safety.
- Use the correct collection and iteration strategy for the concurrency requirements.

---

# ConcurrentHashMap in Java: Internal Implementation and Concurrent Collections

## 1. What Is ConcurrentHashMap?

`ConcurrentHashMap` is a thread-safe implementation of the `ConcurrentMap` interface, which extends `Map`. It was introduced in Java 5 to support efficient access to a shared map in multithreaded applications.

Unlike `HashMap`, it supports concurrent reads and updates without requiring callers to synchronize every operation on the entire map.

### Key Features

- **Thread-safe:** Multiple threads can access and update the map concurrently.
- **High concurrency:** Operations on different parts of the map can often proceed simultaneously.
- **Atomic methods:** Supports operations such as `putIfAbsent()`, conditional `remove()`, `replace()`, and `computeIfAbsent()`.
- **Weakly consistent iterators:** Iteration can proceed during concurrent updates without throwing `ConcurrentModificationException` merely because the map changes.
- **No null keys or values:** Attempting to insert a null key or value throws `NullPointerException`.
- **No ordering guarantee:** The map does not guarantee insertion or sorted order.

```java
import java.util.concurrent.ConcurrentHashMap;

public class Main {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map =
                new ConcurrentHashMap<>();

        map.put("one", 1);
        map.put("two", 2);
        map.put("three", 3);

        System.out.println(map.get("one"));          // 1

        map.remove("two");

        System.out.println(map.containsKey("two"));  // false
    }
}
```

**Output:**

```text
1
false
```

## 2. HashMap vs ConcurrentHashMap

| Feature | HashMap | ConcurrentHashMap |
|---|---|---|
| Thread-safe | No | Yes, for individual operations |
| Concurrent access | Requires external coordination for shared mutation | Designed for concurrent access |
| Null keys | One allowed | Not allowed |
| Null values | Allowed | Not allowed |
| Iterators | Typically fail-fast, best-effort | Weakly consistent |
| Ordering | No guarantee | No guarantee |
| Atomic update methods | No concurrent atomicity guarantee | Provides atomic operations |
| Typical use | Single-threaded or externally synchronized access | Shared maps in multithreaded applications |

**Important:** Thread safety of individual operations does not automatically make a sequence of operations atomic. Use atomic map methods or appropriate synchronization when a complete sequence must be indivisible.

## 3. How Does ConcurrentHashMap Work Internally?

The internal implementation changed significantly between Java versions.

### Java 7 and Earlier: Segment-Based Locking

Older implementations divided the map into multiple segments. Each segment managed part of the entries and had its own lock.

Conceptually:

```text
ConcurrentHashMap
       |
       +-- Segment 1 (lock)
       |      +-- Entries
       |
       +-- Segment 2 (lock)
       |      +-- Entries
       |
       +-- Segment 3 (lock)
              +-- Entries
```

Threads operating on different segments could make progress concurrently, reducing contention compared with locking the entire map.

### Java 8 and Later: Bucket-Level Coordination

Java 8 redesigned `ConcurrentHashMap`. It no longer uses the old fixed-segment architecture.

Its implementation uses techniques including:

1. **Bucket array:** Entries are organized by hash into table buckets.
2. **CAS (Compare-And-Set):** Used for certain atomic updates, particularly when initializing or inserting into an empty bucket.
3. **Synchronization on a bucket's first node:** Updates to a non-empty bucket can use a lock associated with that bucket.
4. **Tree bins:** A heavily populated bucket can be represented as a balanced tree to improve lookup performance.
5. **Cooperative resizing:** Multiple threads can help transfer entries when the table grows.
6. **Volatile fields and memory-ordering mechanisms:** Help make updates visible across threads.

Conceptually:

```text
ConcurrentHashMap
       |
       v
    Table Array
       |
       +-- Bucket 0 --> Node --> Node
       |
       +-- Bucket 1 --> Node
       |
       +-- Bucket 2 --> Tree Bin
       |
       +-- Bucket 3 --> Empty
```

### How `put()` Works Conceptually

When a thread calls `put(key, value)`:

1. The map calculates a spread hash from the key's hash code.
2. It uses the hash to locate the relevant bucket.
3. If the bucket is empty, it may use CAS to insert the new node.
4. If the bucket is occupied, it coordinates updates to that bucket, commonly using synchronization on the bucket's first node.
5. If the key already exists, its value is updated.
6. If the table needs to grow, resizing and entry transfer may occur.

The exact implementation is an internal JDK detail and can vary between versions.

### How `get()` Works

A typical `get(key)` operation:

1. Calculates the key's hash.
2. Locates the corresponding bucket.
3. Examines the bucket's nodes or tree.
4. Compares keys using hash and equality checks.
5. Returns the associated value or `null` if no mapping exists.

Reads generally do not acquire the bucket locks used by ordinary updates, allowing efficient concurrent access.

## 4. Why Does ConcurrentHashMap Not Allow Null?

`ConcurrentHashMap` rejects null keys and values in all standard Java versions that provide this class.

Consider:

```java
map.put("user", null);
```

This throws `NullPointerException`.

One important reason is ambiguity during concurrent access. If `get("user")` returns `null`, the caller can interpret it as no mapping for that key. Allowing a mapped null value would make that distinction harder to use safely in concurrent code.

Use an explicit value or a separate state indicator when representing the absence of a value.

## 5. Atomic Operations in ConcurrentHashMap

A thread-safe map does not make a separate `get()` followed by `put()` atomic.

### Problem: A Race Condition

```java
if (!map.containsKey("count")) {
    map.put("count", 1);
}
```

Two threads can both observe that the key is absent before either inserts it.

### Solution: Use `putIfAbsent()`

```java
map.putIfAbsent("count", 1);
```

The check and insertion are performed atomically.

Other useful methods include:

| Method | Purpose |
|---|---|
| `putIfAbsent(key, value)` | Inserts only if no mapping currently exists |
| `remove(key, value)` | Removes only if the key maps to the specified value |
| `replace(key, oldValue, newValue)` | Replaces only if the current value matches |
| `computeIfAbsent(key, function)` | Computes and installs a value when the key has no mapping |
| `compute(key, function)` | Atomically computes an update for a key |
| `merge(key, value, function)` | Adds or combines a value atomically for a key |

Example:

```java
import java.util.concurrent.ConcurrentHashMap;

public class Main {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> counts =
                new ConcurrentHashMap<>();

        counts.putIfAbsent("Java", 1);
        counts.putIfAbsent("Java", 2);

        counts.computeIfPresent(
                "Java",
                (key, value) -> value + 1
        );

        System.out.println(counts.get("Java"));
    }
}
```

**Output:**

```text
2
```

The second `putIfAbsent()` does not overwrite the existing value, and `computeIfPresent()` increments it.

## 6. Can We Modify ConcurrentHashMap While Iterating?

**Yes.** Its iterators are weakly consistent. They can run while other threads update the map and do not throw `ConcurrentModificationException` merely because of those updates.

However, an iterator is **not a snapshot**. It may reflect some changes made during iteration and may not reflect others.

Example:

```java
import java.util.concurrent.ConcurrentHashMap;

public class Main {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map =
                new ConcurrentHashMap<>();

        map.put("one", 1);
        map.put("two", 2);
        map.put("three", 3);

        for (var entry : map.entrySet()) {
            if (entry.getKey().equals("two")) {
                map.remove("two");
            }
        }

        System.out.println(map.containsKey("two"));
    }
}
```

**Output:**

```text
false
```

This is a valid operation. However, because iteration is weakly consistent, the iterator's observations should not be treated as a guaranteed view of the map at one instant.

Use atomic map operations for concurrent updates, and use explicit synchronization or another coordination strategy if the whole iteration-and-update workflow must follow a strict consistency rule.

## 7. What Is a Concurrent Collection?

Concurrent collections are data structures designed to support safe access by multiple threads. Most commonly used implementations are in `java.util.concurrent`.

| Collection | Purpose |
|---|---|
| `ConcurrentHashMap` | Concurrent key-value storage |
| `ConcurrentSkipListMap` | Concurrent sorted map |
| `ConcurrentSkipListSet` | Concurrent sorted set |
| `CopyOnWriteArrayList` | Thread-safe list with snapshot-style iteration |
| `CopyOnWriteArraySet` | Set built around copy-on-write behavior |
| `ConcurrentLinkedQueue` | Non-blocking concurrent FIFO queue |
| `ConcurrentLinkedDeque` | Non-blocking concurrent double-ended queue |
| `LinkedBlockingQueue` | Blocking queue suitable for producer-consumer workflows |
| `ArrayBlockingQueue` | Bounded blocking queue backed by an array |
| `PriorityBlockingQueue` | Blocking queue ordered by priority |

Choose based on the required ordering, blocking behavior, read/write ratio, and performance characteristics. Not every concurrent collection uses locks; for example, `ConcurrentLinkedQueue` is designed around non-blocking algorithms.

## 8. What Is ConcurrentModificationException?

`ConcurrentModificationException` is a runtime exception that commonly occurs when a fail-fast iterator detects a structural modification to a collection outside the iterator's supported removal mechanism.

Despite its name, it does **not** necessarily mean multiple threads are involved. A single thread can trigger it.

### Incorrect: Removing from an ArrayList During Enhanced For

```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> items = new ArrayList<>(
                List.of("Java", "Python", "JavaScript")
        );

        for (String item : items) {
            if (item.equals("Python")) {
                items.remove(item);
            }
        }
    }
}
```

This can throw `ConcurrentModificationException` because the list is modified directly during fail-fast iteration.

### Solution 1: Use Iterator.remove()

```java
var iterator = items.iterator();

while (iterator.hasNext()) {
    if (iterator.next().equals("Python")) {
        iterator.remove();
    }
}
```

### Solution 2: Use removeIf()

```java
items.removeIf(item -> item.equals("Python"));
```

### Solution 3: Use a Suitable Concurrent Collection

For a workload with concurrent access, consider `ConcurrentHashMap`, `CopyOnWriteArrayList`, or another collection designed for the required behavior.

**Important distinction:** Synchronizing access can protect a collection when all relevant accesses follow the same locking protocol. Merely placing one modification inside a `synchronized` block does not make unsynchronized iteration safe. Also, concurrent collections may have different iterator guarantees; do not assume they all behave like snapshot iterators.

## 9. Common Interview Questions

### Q1. Is ConcurrentHashMap completely lock-free?

No. It uses CAS for certain operations and avoids a single global lock for ordinary updates, but it can synchronize updates to occupied buckets and use coordination during resizing.

### Q2. Can two threads update ConcurrentHashMap simultaneously?

Yes. Updates affecting different buckets can often proceed concurrently. Operations that contend for the same bucket may need coordination.

### Q3. Does ConcurrentHashMap maintain insertion order?

No. Use `LinkedHashMap` for insertion order when thread safety is not otherwise required, or select a suitable synchronized or concurrent design if both ordering and concurrency are required.

### Q4. Is ConcurrentHashMap better than HashMap?

Not universally. `HashMap` is generally simpler for single-threaded use or when external synchronization already protects access. `ConcurrentHashMap` is designed for shared concurrent access.

### Q5. Can ConcurrentHashMap contain duplicate keys?

No. Each key has at most one current mapping, just like other `Map` implementations. Multiple keys can map to the same value.

### Q6. Does ConcurrentHashMap throw ConcurrentModificationException?

Its iterators are weakly consistent and do not throw `ConcurrentModificationException` merely because the map is concurrently modified. This does not mean iteration provides a consistent snapshot.

### Q7. What is the difference between ConcurrentHashMap and Hashtable?

`Hashtable` synchronizes its methods, which can create contention across operations on the same map. `ConcurrentHashMap` is designed for greater concurrency and provides atomic compound operations. Both reject null keys and values.

## 10. One-Line Interview Answer

`ConcurrentHashMap` is a thread-safe implementation of `ConcurrentMap` that supports efficient concurrent access using techniques such as CAS, bucket-level synchronization, and cooperative resizing, with weakly consistent iterators and atomic update methods.


---

# Static Synchronization in Java

## 1. What Is Static Synchronization in Java?

**Static synchronization** is the synchronization of a static method or a critical section using the class-level monitor, ensuring that only one thread at a time can execute code protected by that same monitor.

In Java, a `static synchronized` method locks the monitor associated with the class's `Class` object, whereas an instance `synchronized` method locks the monitor of the current object (`this`).

Static synchronization is useful when multiple threads access shared class-level state that must be updated consistently.

## 2. How to Create a Static Synchronized Method

Use the `static` and `synchronized` keywords in the method declaration.

```java
class Counter {
    private static int count = 0;

    public static synchronized void increment() {
        count++;
    }

    public static synchronized int getCount() {
        return count;
    }
}
```

### How Does It Work?

- `static` means the method belongs to the class.
- `synchronized` means the calling thread must acquire the class monitor before executing the method body.
- Only one thread at a time can hold that class monitor.
- The monitor is released when the method exits, including when it exits by throwing an exception.

For example, if multiple threads call `Counter.increment()`, their executions are serialized by the same class monitor.

**Important:** The synchronization works only for code using that same monitor. Unsynchronized access to `count` can still introduce data races.

## 3. Example: Static Synchronization With Multiple Threads

```java
public class StaticSynchronizationDemo {
    private static int count = 0;

    public static synchronized void increment() {
        count++;
    }

    public static int getCount() {
        return count;
    }

    public static void main(String[] args)
            throws InterruptedException {

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 100_000; i++) {
                increment();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 100_000; i++) {
                increment();
            }
        });

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println(getCount());
    }
}
```

**Output:**

```text
200000
```

Both threads update the same static variable. Because `increment()` is static synchronized, the increment operation is protected by the class monitor.

`join()` ensures the main thread waits for both worker threads to finish before printing the result.

## 4. How to Synchronize a Static Variable?

Java does **not** allow the `synchronized` keyword on a variable declaration.

This is invalid:

```java
public static synchronized int count; // Compilation error
```

Instead, synchronize the code that accesses or modifies the shared variable.

### Option 1: Synchronize on the Class Object

```java
class SharedData {
    private static int count = 0;

    public static void add(int value) {
        synchronized (SharedData.class) {
            count += value;
        }
    }

    public static int getCount() {
        synchronized (SharedData.class) {
            return count;
        }
    }
}
```

The synchronized blocks use `SharedData.class`, so they coordinate through the same monitor.

### Option 2: Use a Static Synchronized Method

```java
class SharedData {
    private static int count = 0;

    public static synchronized void add(int value) {
        count += value;
    }
}
```

A static synchronized method automatically acquires the monitor associated with `SharedData.class`.

### Option 3: Use an Atomic Variable

For simple atomic numeric updates, `AtomicInteger` can be a better choice.

```java
import java.util.concurrent.atomic.AtomicInteger;

class SharedCounter {
    private static final AtomicInteger count =
            new AtomicInteger(0);

    public static void increment() {
        count.incrementAndGet();
    }

    public static int getCount() {
        return count.get();
    }
}
```

`AtomicInteger` provides atomic operations on the integer value without requiring an explicit synchronized method for each update.

However, if an operation must maintain a relationship between multiple variables, a larger synchronized section or another suitable concurrency mechanism may still be necessary.

## 5. Static Synchronization vs Instance Synchronization

| Feature | Static synchronized method | Instance synchronized method |
|---|---|---|
| Lock used | Class object's monitor | Current instance's monitor (`this`) |
| Scope | Shared across calls using the same class monitor | Shared across calls on the same object |
| Different instances | Calls still contend for the same class monitor | Calls on different instances can execute concurrently |
| Typical use | Protect shared static state | Protect per-object mutable state |
| Syntax | `static synchronized void method()` | `synchronized void method()` |

### Example: Different Locks

```java
class Example {
    public static synchronized void staticMethod() {
        // Locks Example.class
    }

    public synchronized void instanceMethod() {
        // Locks this instance
    }
}
```

Calling `staticMethod()` does not acquire the same monitor as calling `instanceMethod()`.

Therefore, these two methods can execute concurrently, assuming no other locking dependencies interfere.

## 6. Static Synchronized Method vs Synchronized Block

A synchronized block gives more control over the scope of synchronization.

```java
class Example {
    private static int count = 0;

    public static void increment() {
        // Other work can happen outside the lock.

        synchronized (Example.class) {
            count++;
        }

        // More work can happen outside the lock.
    }
}
```

This block uses the same monitor as a static synchronized method in `Example`.

Use a smaller critical section when it is safe to do so, because reducing lock contention can improve throughput.

Be careful when choosing a lock object. A publicly accessible class object can be locked by other code, potentially causing unexpected contention or deadlocks. A private static final lock object can be preferable when the lock does not need to be shared externally.

## 7. Important Interview Points

### Q1. Does static synchronization lock all objects of a class?

It locks the class's `Class` object, not each individual instance. All calls using that same class monitor coordinate across instances.

### Q2. Can two static synchronized methods of the same class execute simultaneously?

Not while both methods hold the same class monitor. Only one thread at a time can hold that monitor.

### Q3. Can a static synchronized method and an instance synchronized method execute simultaneously?

Yes. They use different monitors: the class object's monitor and the instance object's monitor.

### Q4. Does `synchronized` make a static variable thread-safe automatically?

No. The variable itself is not synchronized. Accesses must consistently use the same synchronization mechanism, or an appropriate atomic or concurrent data structure.

### Q5. Is static synchronization always slower?

Not necessarily. Synchronization introduces coordination and can cause contention, but the impact depends on workload and JVM optimizations. Use synchronization when needed for correctness, then measure performance.

## 8. One-Line Interview Answer

Static synchronization in Java uses the class object's monitor to coordinate access to shared class-level state; a `static synchronized` method locks `ClassName.class`, whereas an instance synchronized method locks the current object.


---

# Which Exceptions Can Be Thrown from the `run()` Method in Java?

## 1. Can the `run()` Method Throw an Exception?

The `run()` method of Java's `Thread` class does not declare any checked exceptions in its method signature.

```java
public void run()
```

Therefore, when overriding `run()`, you cannot add a new checked exception to its `throws` clause if that exception is not permitted by the overridden method's signature.

However, the `run()` method can throw **unchecked exceptions**, including `RuntimeException` subclasses and `Error` subclasses.

Examples include:

- `ArithmeticException`
- `NullPointerException`
- `IllegalArgumentException`
- `AssertionError`
- `ThreadDeath` (a special `Error`)

## 2. Example: Throwing an Unchecked Exception from `run()`

```java
public class ThreadExceptionDemo {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("Thread started");

            int result = 10 / 0;

            System.out.println(result);
        });

        thread.start();

        System.out.println("Main thread continues");
    }
}
```

Possible output:

```text
Thread started
Main thread continues
Exception in thread "Thread-0" java.lang.ArithmeticException: / by zero
```

The exact output order may vary because the main thread and worker thread execute concurrently.

**What happens?**

1. The worker thread starts executing its task.
2. Division by zero throws `ArithmeticException`.
3. The exception propagates up the worker thread's call stack.
4. If it remains uncaught, the thread's uncaught-exception handling mechanism is invoked, and the worker thread terminates.
5. The main thread can continue executing independently.

An uncaught exception in one thread does not ordinarily terminate other threads.

## 3. Can We Throw a Checked Exception from `run()`?

You cannot add a checked exception to the overridden `Thread.run()` method's signature.

This example does not compile:

```java
class MyThread extends Thread {
    @Override
    public void run() throws InterruptedException {
        Thread.sleep(1000);
    }
}
```

The overridden method cannot broaden the checked exceptions permitted by the superclass method.

Instead, catch the checked exception inside `run()`:

```java
class MyThread extends Thread {
    @Override
    public void run() {
        try {
            Thread.sleep(1000);
            System.out.println("Task completed");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println("Thread interrupted");
        }
    }
}
```

Restoring the interrupt status with `Thread.currentThread().interrupt()` is a common approach when the method cannot propagate the interruption to its caller.

## 4. What Is `ThreadDeath`?

`ThreadDeath` is an unchecked `Error` historically associated with the deprecated `Thread.stop()` mechanism.

It is not the normal exception that indicates a thread has completed its work. A thread normally terminates when its `run()` method returns or an uncaught throwable escapes it.

Application code should not use `Thread.stop()` to terminate threads because it can stop a thread while shared data is in an inconsistent state.

Prefer cooperative cancellation using interruption, a cancellation flag, or an appropriate executor API.

## 5. Should We Always Catch Exceptions Inside `run()`?

Not necessarily. Catch exceptions when you can handle them meaningfully, recover, clean up, or report a useful failure. Avoid silently swallowing unexpected exceptions.

For example:

```java
Thread thread = new Thread(() -> {
    try {
        performTask();
    } catch (RuntimeException e) {
        System.err.println("Task failed: " + e.getMessage());
        throw e;
    }
});

thread.setUncaughtExceptionHandler((t, e) ->
        System.err.println(
                "Uncaught exception in " + t.getName()
                        + ": " + e));
```

This example logs the failure locally and rethrows it, allowing the uncaught-exception handler to observe it. In a real application, avoid logging the same exception redundantly at multiple layers.

For tasks submitted to an `ExecutorService`, exceptions are commonly captured by the returned `Future` and can be observed through `Future.get()`, which may throw `ExecutionException`.

## 6. `Thread.run()` vs `Callable.call()`

| Feature | `Thread.run()` / `Runnable.run()` | `Callable.call()` |
|---|---|---|
| Return value | `void` | Returns a value |
| Checked exceptions | Cannot declare arbitrary checked exceptions | Can declare checked exceptions |
| Typical execution | Direct thread execution or submitted `Runnable` task | Usually submitted to an executor |
| Failure observation | Uncaught handler if it escapes a directly running thread | Often observed through `Future.get()` when submitted to an executor |

Example of `Callable`:

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableExceptionDemo {
    public static void main(String[] args)
            throws InterruptedException, ExecutionException {

        ExecutorService executor =
                Executors.newSingleThreadExecutor();

        try {
            Callable<Integer> task = () -> {
                throw new Exception("Task failed");
            };

            Future<Integer> future = executor.submit(task);

            try {
                future.get();
            } catch (ExecutionException e) {
                System.out.println(
                        "Cause: " + e.getCause().getMessage());
            }
        } finally {
            executor.shutdown();
        }
    }
}
```

Output:

```text
Cause: Task failed
```

## 7. Common Interview Questions

### Q1. Can `run()` throw `NullPointerException`?

Yes. `NullPointerException` is an unchecked exception, so it can propagate out of `run()`.

### Q2. What happens when an exception escapes `run()`?

If uncaught, it is passed to the thread's uncaught-exception handling mechanism. The affected thread terminates, while other threads can continue.

### Q3. Can we declare `throws Exception` in an overridden `run()` method?

No. `Thread.run()` does not declare checked exceptions, so an overriding method cannot add `throws Exception`.

### Q4. How do we handle exceptions from tasks submitted to an executor?

Use the returned `Future` and call `get()` to observe task failures through `ExecutionException`, or use an appropriate task-level error-handling strategy.

## 8. One-Line Interview Answer

The `run()` method can propagate unchecked exceptions and errors, but it cannot declare arbitrary checked exceptions when overriding `Thread.run()`; uncaught failures are handled by the thread's uncaught-exception mechanism and terminate that thread.


---

# ThreadLocal, Weak References, volatile, finalize, finally, and Serialization in Java

These are frequently asked Java interview topics covering thread-specific data, memory management, concurrency, exception handling, and object persistence.

## 1. What Is ThreadLocal in Java?

`ThreadLocal<T>` is a Java class that allows each thread to maintain its own independent value for a variable.

If multiple threads access the same `ThreadLocal` object, each thread gets its own value rather than sharing one common value.

### Example

```java
public class ThreadLocalDemo {

    private static final ThreadLocal<Integer> threadLocal =
            ThreadLocal.withInitial(() -> 0);

    public static void main(String[] args)
            throws InterruptedException {

        Thread t1 = new Thread(() -> {
            threadLocal.set(100);
            System.out.println("Thread 1: " + threadLocal.get());
            threadLocal.remove();
        });

        Thread t2 = new Thread(() -> {
            threadLocal.set(200);
            System.out.println("Thread 2: " + threadLocal.get());
            threadLocal.remove();
        });

        t1.start();
        t2.start();

        t1.join();
        t2.join();
    }
}
```

Possible output:

```text
Thread 1: 100
Thread 2: 200
```

The output order may vary, but each thread sees its own value.

### Common Use Cases

- Request-specific context in web applications.
- Correlation IDs for logging and tracing.
- Legacy APIs that require per-thread state.
- Thread-confined formatters or other mutable objects.

**Important:** `ThreadLocal` is not a mechanism for sharing data between threads. It provides separate values for each thread.

### ThreadLocal and Thread Pools

Thread-pool workers are reused, so thread-local values can remain associated with a worker after one task finishes. This can accidentally expose one request's context to a later request processed by the same worker.

Always clean up request-specific values in a `finally` block when appropriate:

```java
private static final ThreadLocal<String> USER =
        new ThreadLocal<>();

public static void processRequest(String user) {
    try {
        USER.set(user);
        // Process the request.
    } finally {
        USER.remove();
    }
}
```

### Does ThreadLocal Implement the Singleton Pattern?

No. `ThreadLocal` is not itself a singleton pattern implementation. A singleton provides one shared instance, while a `ThreadLocal` provides per-thread values associated with a `ThreadLocal` instance.

---

## 2. What Are Weak References in Java?

A **weak reference** allows an object to be referenced without preventing garbage collection of that object when it is no longer strongly reachable.

Java provides the `WeakReference<T>` class in `java.lang.ref`.

### Example

```java
import java.lang.ref.WeakReference;

public class WeakReferenceDemo {
    public static void main(String[] args) {
        Object object = new Object();

        WeakReference<Object> weakReference =
                new WeakReference<>(object);

        System.out.println(weakReference.get() != null);

        object = null;

        // The object may be garbage-collected,
        // but garbage collection is not guaranteed immediately.
        System.gc();

        System.out.println(weakReference.get());
    }
}
```

Before the strong reference is removed, `weakReference.get()` returns the object. Afterward, it may return `null` once garbage collection clears the weak reference.

Do not rely on `System.gc()` to trigger collection at a particular time.

### Types of References

| Reference type | Can prevent garbage collection? | Typical use |
|---|---|---|
| Strong reference | Yes | Ordinary object references |
| Soft reference | May be retained until memory pressure, at the collector's discretion | Memory-sensitive caches |
| Weak reference | No, not once the object is only weakly reachable | Weak associations and some cache designs |
| Phantom reference | Does not keep the object alive; `get()` returns `null` | Advanced post-mortem cleanup tracking using a reference queue |

### WeakHashMap

`WeakHashMap` uses weak references for its keys. Entries may disappear when their keys are no longer strongly reachable elsewhere.

It is useful for certain associations where the map should not keep keys alive, but it is not a general-purpose replacement for every cache.

---

## 3. What Is the volatile Keyword in Java?

`volatile` is a keyword used for fields that need visibility guarantees between threads.

When one thread writes to a volatile variable, subsequent reads of that variable by other threads observe the write according to Java's memory model.

It also establishes relevant happens-before ordering for operations around volatile reads and writes.

### Example: Stopping a Thread

```java
public class VolatileDemo {

    private static volatile boolean running = true;

    public static void main(String[] args)
            throws InterruptedException {

        Thread worker = new Thread(() -> {
            while (running) {
                // Perform work.
            }

            System.out.println("Worker stopped");
        });

        worker.start();

        Thread.sleep(100);
        running = false;

        worker.join();
    }
}
```

The volatile field ensures the worker can observe the updated flag without relying on an unsynchronized ordinary read.

### What volatile Does Not Do

`volatile` does **not** make compound operations atomic.

```java
private static volatile int count = 0;

public static void increment() {
    count++; // Not atomic
}
```

The `count++` operation involves reading, incrementing, and writing. Concurrent increments can be lost.

Use `AtomicInteger`, synchronization, or another suitable concurrency mechanism when atomic updates are required.

### volatile vs synchronized

| Feature | volatile | synchronized |
|---|---|---|
| Visibility | Yes | Yes, through monitor release/acquisition |
| Atomic compound operations | No | Yes, when the complete operation is protected by the same monitor |
| Mutual exclusion | No | Yes |
| Typical use | State flags and safely published values | Protecting shared state and multi-step operations |

---

## 4. What Is finalize() in Java?

`finalize()` was a method inherited from `Object` that the garbage collector could invoke before reclaiming an object.

It was historically used for cleanup, but it was unpredictable and unsafe for reliable resource management.

**Important:** Finalization has been deprecated for removal since Java 18. New code should not depend on it.

### Historical Example — Do Not Use in New Code

```java
@Override
protected void finalize() throws Throwable {
    try {
        System.out.println("Finalizer invoked");
    } finally {
        super.finalize();
    }
}
```

There is no guarantee that this method will run promptly, or even that it will run before the JVM exits.

### Why Is finalize() Discouraged?

- Execution time is unpredictable.
- Finalizers can delay garbage collection.
- Finalization can introduce security and object-resurrection problems.
- Resource cleanup may happen too late or not at all.

### What Should We Use Instead?

Use `try-with-resources` and `AutoCloseable` for resources such as files, sockets, and database connections.

```java
try (var reader =
         new java.io.BufferedReader(
             new java.io.FileReader("input.txt"))) {

    System.out.println(reader.readLine());
}
```

The resource is closed automatically when the try block exits.

---

## 5. What Is finally in Java?

`finally` is a block used with `try` and `catch` for cleanup or other code that should normally run regardless of whether an exception occurs.

### Example

```java
public class FinallyDemo {
    public static void main(String[] args) {
        try {
            int result = 10 / 0;
            System.out.println(result);
        } catch (ArithmeticException e) {
            System.out.println("Exception caught");
        } finally {
            System.out.println("Finally block executed");
        }
    }
}
```

Output:

```text
Exception caught
Finally block executed
```

### Important Points

- `finally` normally runs whether the `try` block completes or throws an exception.
- It commonly supports cleanup, although `try-with-resources` is preferred for `AutoCloseable` resources.
- It may not run if the JVM terminates abruptly, the process is forcibly killed, or the machine loses power.
- Avoid returning or throwing a new exception from `finally`, because this can hide an earlier return value or exception.

### finally vs finalize()

| `finally` | `finalize()` |
|---|---|
| A language construct used with exception handling | A method historically associated with garbage collection |
| Runs as part of normal control flow, subject to abrupt termination | Invocation was not guaranteed |
| Useful for cleanup logic | Deprecated for removal; should not be relied upon |
| Not a garbage-collection mechanism | Related to the old finalization mechanism |

---

## 6. What Is Serialization in Java?

**Serialization** converts an object's state into a byte stream. The bytes can be stored or transmitted and later used to reconstruct an object through deserialization.

Java's traditional object serialization mechanism uses `Serializable`, `ObjectOutputStream`, and `ObjectInputStream`.

### Example

```java
import java.io.Serializable;

public class Employee implements Serializable {
    private static final long serialVersionUID = 1L;

    private int id;
    private String name;

    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return id + " - " + name;
    }
}
```

### Serialize and Deserialize an Object

```java
import java.io.*;

public class SerializationDemo {
    public static void main(String[] args)
            throws IOException, ClassNotFoundException {

        Employee employee = new Employee(101, "Danish");

        try (ObjectOutputStream out =
                 new ObjectOutputStream(
                     new FileOutputStream("employee.ser"))) {

            out.writeObject(employee);
        }

        try (ObjectInputStream in =
                 new ObjectInputStream(
                     new FileInputStream("employee.ser"))) {

            Employee restored =
                    (Employee) in.readObject();

            System.out.println(restored);
        }
    }
}
```

Output:

```text
101 - Danish
```

### Important Serialization Rules

- `Serializable` is a marker interface.
- `serialVersionUID` identifies a version of the serialized class for compatibility checks.
- Static fields are not part of the object's default serialized instance state.
- Fields declared `transient` are excluded from default serialization.
- A non-transient, non-serializable object field can cause `NotSerializableException`.
- Deserialization can execute class-specific behavior and must not be performed on untrusted data without appropriate safeguards.

Traditional Java serialization is not the same as converting an object to JSON. JSON serialization usually requires a JSON library and explicit rules for mapping object state to a data format.

---

## 7. Quick Revision Table

| Concept | Main purpose | Key caution |
|---|---|---|
| `ThreadLocal` | Store independent values per thread | Remove request-specific values when using thread pools |
| Weak reference | Reference an object without keeping it alive | Garbage collection timing is unpredictable |
| `volatile` | Provide visibility and ordering guarantees | Does not make `count++` atomic |
| `finalize()` | Historical garbage-collection cleanup hook | Deprecated for removal; avoid it |
| `finally` | Execute normal cleanup/control-flow code | Not guaranteed after abrupt JVM or process termination |
| Serialization | Convert object state to a byte stream | Deserialization of untrusted data is dangerous |

## 8. One-Line Interview Answer

`ThreadLocal` provides per-thread values, weak references do not keep otherwise unreachable objects alive, `volatile` provides visibility guarantees, `finalize()` is a deprecated cleanup mechanism, `finally` supports exception-related cleanup, and serialization converts object state into a byte stream for storage or transfer.



