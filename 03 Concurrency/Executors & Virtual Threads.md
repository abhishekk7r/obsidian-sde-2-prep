# Executors & Virtual Threads

## Thread Pools & ExecutorService

![[jvm-os-thread-mapping.svg]]

- Every `java.lang.Thread` maps **1:1 to a native OS thread** on HotSpot — JVM asks OS for a real kernel thread, OS scheduler time-slices across CPU cores
- Thread creation is expensive precisely because it's an **OS-level cost**, not just JVM
- No hard JVM cap on thread count, but bounded by: stack memory per thread (~512KB–1MB, tunable via `-Xss`), OS thread limits (`ulimit -u`), and context-switch overhead long before either wall is hit

> [!tip] Mnemonic
> Classic `Thread` = always 1:1 with the OS. (Virtual threads break this — covered separately.)

### `Executors` factory methods

| Method | Threads | Queue | Use case |
|---|---|---|---|
| `newFixedThreadPool(n)` | Fixed at n | Unbounded `LinkedBlockingQueue` | Steady predictable load; risk: unbounded queue growth → OOM under sustained backlog |
| `newCachedThreadPool()` | Unbounded, on demand | `SynchronousQueue` (direct hand-off) | Many short-lived bursty tasks; risk: unbounded thread creation |
| `newSingleThreadExecutor()` | Exactly 1 | Unbounded | Strict sequential/ordered execution |
| `newScheduledThreadPool(n)` | Fixed at n | — | Delayed/periodic tasks |

![[cached-thread-pool-dynamics.svg]]

> [!warning] Interview trap
> Most `Executors.new*` factories use unbounded queues or unbounded thread creation — production code often constructs `ThreadPoolExecutor` directly with explicit bounds instead.

### `ThreadPoolExecutor` core parameters

| Parameter | Role |
|---|---|
| `corePoolSize` | Threads kept alive even when idle (unless `allowCoreThreadTimeOut(true)`) |
| `maximumPoolSize` | Hard ceiling on total threads |
| `keepAliveTime` | How long a thread beyond core sits idle before termination |
| `workQueue` | Where tasks wait when core threads are busy |

**Submission order (memorize this exactly):**
1. Threads < `corePoolSize` → **spawn new thread**, even if other core threads are idle
2. Core threads busy → **queue the task**
3. Queue full **and** threads < `maximumPoolSize` → **spawn thread beyond core**, up to max
4. Queue full **and** at max → **reject**

> [!danger] Common mistake
> The pool only grows past `corePoolSize` **after the queue is full** — not immediately on the next task. Trace this order exactly when walking through a scenario.

### Rejection policies

| Policy | Behavior |
|---|---|
| `AbortPolicy` (default) | Throws `RejectedExecutionException` |
| `CallerRunsPolicy` | Submitting thread runs the task itself — natural backpressure |
| `DiscardPolicy` | Silently drops the task |
| `DiscardOldestPolicy` | Drops oldest queued task, retries submit |

> [!tip] Why `CallerRunsPolicy` is a common production default
> Converts overload into backpressure — slows the producer instead of failing/dropping. Downside: blocks the caller thread doing pool work, degrading its own throughput if it happens often.

### `shutdown()` vs `shutdownNow()`

| | `shutdown()` | `shutdownNow()` |
|---|---|---|
| Behavior | Graceful — stops new tasks, lets queued + running finish | Aggressive — interrupts running tasks, returns unstarted queued tasks |
| Guarantee | All in-flight work completes | None — task must cooperate with interrupt |

> [!warning] `awaitTermination(timeout, unit)` after `shutdown()`
> Blocks the caller up to `timeout` waiting for termination. Returns `true` if finished in time, `false` if tasks are still running — and in the `false` case, **nothing is forcibly stopped**; tasks keep running in the background. Common idiom: call `shutdownNow()` as a fallback when `awaitTermination` returns `false`.

> [!tip] Mnemonic
> `shutdown()` finishes the queue. `shutdownNow()` tries to cut the line and interrupt whoever's talking.

---

## Virtual Threads & Java 17-21 Features

![[virtual-vs-platform-threads.svg]]

- Solves: platform threads are 1:1 with OS threads (expensive, capped at low thousands) — virtual threads let you write plain **blocking, sequential-style code** at massive scale (millions) without going reactive/async
- Virtual threads are **M:N** — many share a small pool of carrier platform threads
- On a blocking call, JVM **unmounts** the virtual thread from its carrier, freeing the carrier for other work; remounts (possibly on a different carrier) once unblocked
- Stack lives on the heap, grows/shrinks dynamically — not a fixed OS stack

### Platform vs virtual threads

| | Platform thread | Virtual thread |
|---|---|---|
| OS mapping | 1:1 | M:N via carrier threads |
| Stack | Fixed OS stack (~1MB) | Small, heap-based, dynamic |
| Cost to create | Expensive | Cheap — millions feasible |
| On blocking I/O | Carrier OS thread blocks too | Unmounts, carrier freed |
| Creation | `new Thread()` | `Thread.ofVirtual().start(...)`, `Executors.newVirtualThreadPerTaskExecutor()` |

```java
Thread vt = Thread.ofVirtual().start(() -> doWork());

try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> doWork());
}
```

> [!danger] Pinning
> A virtual thread **cannot unmount** inside a `synchronized` block/method or during certain native calls (Java 21) — stays pinned to its carrier, blocking it like a platform thread. Prefer `ReentrantLock` over `synchronized` in virtual-thread-heavy code.

> [!tip] Mnemonic
> Unmount on block, pin on `synchronized`.

### Structured concurrency (JEP 453, preview)

- Treats a group of subtasks in separate threads as **one unit of work with one lifetime** — if one fails, siblings are auto-cancelled, nothing leaks past the parent's scope

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var user = scope.fork(() -> fetchUser());
    var order = scope.fork(() -> fetchOrder());
    scope.join().throwIfFailed();
}
```

### Java 17-21 grab-bag

| Feature | Version | One-liner |
|---|---|---|
| Record patterns | 21 | Deconstruct records in `instanceof`/`switch`: `if (obj instanceof Point(int x, int y))` |
| Sequenced Collections | 21 | `SequencedCollection`/`SequencedSet`/`SequencedMap` — uniform `getFirst()`, `getLast()`, `reversed()` |
| Pattern matching for switch, Records, Sealed interfaces | 17/21 | Already covered in Java/Java Core.md |
