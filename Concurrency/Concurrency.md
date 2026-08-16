
## 1. Process vs Thread

| | Process | Thread |
|---|---|---|
| Memory | Own isolated address space (heap, code, data, FDs) | Own stack, PC, registers — **shares** heap/code/FDs with sibling threads |
| Communication | IPC (pipes/sockets/shared mem) — slow, serialized | Shared heap — fast, direct |
| Fault isolation | Yes — one process crashing doesn't affect others | No — bug in one thread can corrupt shared state for all |
| Context switch cost | High — address space switch, page table swap, TLB flush | Low — just registers/PC/stack pointer, same address space |

> [!tip] Why thread switch is cheaper
> No MMU/TLB touch needed — same address space. Process switch requires a full page table reload + TLB flush.

### Uncaught exceptions
- Uncaught exception → default handler prints trace → **only that thread dies**, others unaffected
- JVM exits only when the **last non-daemon thread** finishes/dies
- Daemon threads never keep JVM alive — irrelevant to lifecycle whether they die by exception or normally

> [!danger] Trap
> Daemon vs non-daemon does NOT change *how* the exception is handled — same handler either way. Only difference: does the JVM care that this thread is gone.

### Thread lifecycle — `Thread.State`
**NEW → RUNNABLE → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED**

| State | Meaning |
|---|---|
| NEW | Created, not started |
| RUNNABLE | Running or ready to run (Java doesn't distinguish) |
| BLOCKED | Waiting to *acquire* a monitor lock |
| WAITING | Gave up lock voluntarily — `wait()`, `join()`, `park()` — indefinite |
| TIMED_WAITING | Same, bounded — `sleep()`, `wait(ms)`, `join(ms)` |
| TERMINATED | Finished |

- Thread IDs **can be reused** after termination (JVM impl detail, not spec-guaranteed)

### `wait()` vs `sleep()`

| | `wait()` | `sleep()` |
|---|---|---|
| Belongs to | `Object` | `Thread` (static) |
| Releases lock held? | **Yes** | **No** |
| Must be in `synchronized`? | Yes (else `IllegalMonitorStateException`) | No |
| Wakes via | `notify()`/`notifyAll()` or timeout | Timer expiry or interrupt |
| Purpose | Inter-thread coordination | Pause execution only |

> [!danger] Trap
> `sleep()` inside `synchronized` still **holds the lock** the whole time. `wait()` exists to avoid exactly this.

**Mnemonic:** *sleep holds, wait yields.*

### `counter++` race
Not atomic — **Read → Modify → Write**. Two threads reading the same stale value → lost update.

---

## 2. Race Conditions & Critical Sections

- **Critical section** — code accessing shared mutable state; protect via mutual exclusion or immutability
- **Race condition** — correctness depends on unpredictable thread interleaving (broad)
- **Data race** — same memory location, ≥1 write, no synchronization between accesses (specific, JMM-undefined-behavior)

> [!tip] Distinction
> Every data race is a race condition. Not every race condition is a data race — can happen even *with* synchronization if separate synchronized calls are combined non-atomically.

| | Race condition | Deadlock |
|---|---|---|
| Symptom | Wrong/inconsistent data, intermittent | Program hangs, no progress |
| Reproducibility | Flaky, timing-dependent | Often reproducible under load |
| Cause | Missing/incorrect synchronization | Circular wait on locks |
| Fix | Proper synchronization | Lock ordering, `tryLock` with timeout |

**Mnemonic:** *Race = wrong answer. Deadlock = no answer.*

### Check-then-act (TOCTOU)
- `if (!map.containsKey(k)) map.put(k, v);` — two threads both pass the check → lost/duplicate insert
- Fix: `putIfAbsent()`, `computeIfAbsent()` — atomic compound methods

### `volatile` ≠ atomicity
- `volatile` = visibility + ordering only — **not atomicity**
- `i++` on volatile int still races (RMW). Use `AtomicInteger` instead.

### Broken double-checked locking
```java
if (instance == null) {
    synchronized(this) {
        if (instance == null) {      // inner check required
            instance = new Singleton();
        }
    }
}
```
- Missing inner check → multiple threads both initialize
- Flag/reference **must be `volatile`** — else reordering can expose a half-constructed object

### Other notes
- Read-only shared data → never races (no writer, no race)
- Synchronized methods ≠ thread-safe class — a **sequence** of two synchronized calls (e.g. `contains` then `add`) isn't atomic as a unit (e.g. `Vector`, `Hashtable`)

---

## 3. `synchronized` Keyword

- Guards **code paths**, not objects — an unsynchronized method on the same object is not blocked by another thread holding the lock elsewhere
- Backed by every object's implicit **monitor** — one thread holds it at a time, others → BLOCKED

> [!danger] Trap
> `synchronized` protects code, not the object. Two threads can be inside a synchronized method and a plain method **simultaneously** on the same object.

| | Locks on | Scope |
|---|---|---|
| Instance `synchronized` / `synchronized(this)` | the instance | per-object |
| Static `synchronized` / `synchronized(ClassName.class)` | the `Class` object | shared across **all** instances |

> [!danger] Trap
> Instance-sync and static-sync methods on the same class **don't block each other** — different locks. Mixing `this` and `.class` to protect the *same* data is a bug.

### Reentrancy
- `synchronized` **is reentrant** — same thread re-entering its own held monitor never self-blocks (hold count tracked internally)
- Nested synchronized calls, same thread, same object → no self-deadlock

### `synchronized` vs `ReentrantLock`

| | `synchronized` | `ReentrantLock` |
|---|---|---|
| Reentrant | Yes | Yes |
| Scope | Block-scoped only | Can span method boundaries |
| `tryLock()` / timeout | No | Yes |
| Fairness policy | No | Yes — `new ReentrantLock(true)`, FIFO |
| Multiple wait-queues | No — one wait-set per object | Yes — `newCondition()`, multiple independent queues |
| Interruptible wait | No | Yes — `lockInterruptibly()` |
| Unlock guarantee | Automatic (even on exception) | **Manual** — must `unlock()` in `finally` |

### Locking on the wrong object
- Never synchronize on `String` literals or cached boxed `Integer` (-128..127) — JVM-wide interned/shared → risk of accidental shared lock with unrelated code
- Use a private dedicated lock: `private final Object lock = new Object();`

**Mnemonic:** *`synchronized` locks code, not objects — and never locks a thread out of itself.*

---

## 4. Locks — ReentrantLock / tryLock / Fairness

### Correct usage pattern
```java
lock.lock();          // acquire — OUTSIDE try (if lock() throws, never acquired, don't try to release)
try {
    // critical section
} finally {
    lock.unlock();     // release — INSIDE finally, guarantees release even on exception
}
```
- Unlike `synchronized`, release is **manual** — forgetting `finally` leaks the lock forever
- `unlock()` without holding the lock → `IllegalMonitorStateException`

### `tryLock()`
- Returns `boolean` — `true` if acquired, `false` if not (immediately, or `tryLock(timeout, unit)` after bounded wait)
- Does **not** spin/busy-wait internally when given a timeout — blocks up to that bound, then gives up

### Breaking deadlock with `tryLock()`
- Classic setup: Thread A holds Lock1 wants Lock2; Thread B holds Lock2 wants Lock1 → circular wait
- Fix: use `tryLock(timeout)` for the second lock. On failure, **release the lock already held**, back off, retry
- Breaks circular-wait condition — no thread blocks forever holding one lock while waiting on another

### `ReentrantLock` vs `synchronized` — scope
- `synchronized` — strictly block/method-scoped, lock()/unlock() implicit and lexically paired
- `ReentrantLock` — **no scope restriction**: can `lock()` in one method, `unlock()` in another, as long as same `Lock` object threaded through
- Enables patterns `synchronized` can't express — e.g. **hand-over-hand locking** (lock coupling) in linked structures: acquire next node's lock before releasing current

### `ReentrantReadWriteLock`

| Read-heavy workload | Write-heavy workload |
|---|---|
| Big win — multiple readers hold read lock concurrently | No benefit — writes serialize anyway, extra overhead for nothing |

> [!danger] Trap
> Read lock = **shared** (many readers at once). Write lock = **exclusive** (blocks readers too). It's not "read then write sequentially" — it's a concurrent-access-mode split.

### Fairness — precise meaning
- `new ReentrantLock(true)` → FIFO-ish: longest-waiting thread served next
- **Not** a hard guarantee that no thread ever waits longer than another — reduces starvation risk, doesn't eliminate all variance

### Checking lock state
- `lock.isHeldByCurrentThread()` — check without attempting to acquire
- `lock.getHoldCount()` — how many times current thread has reentered

**Mnemonic:** *Lock outside try, unlock inside finally — or you leak it forever.*

---

## 5. wait() / notify() / notifyAll()

- Both `wait()` and `notify()`/`notifyAll()` require holding the **same object's monitor** (inside `synchronized(obj)`) — else `IllegalMonitorStateException`
- `notify()` wakes exactly **one** waiting thread (unspecified which) — `notifyAll()` wakes all, needed when multiple *kinds* of waiters exist (e.g. producers vs consumers) so the right kind actually gets a turn

### Always use `while`, never `if`, around `wait()`
```java
synchronized void consume() {
    while (queue.isEmpty()) {     // NOT if
        wait();
    }
    Object item = queue.remove(); // safe — re-verified right before use
}
```
Reasons:
1. **Spurious wakeups** — JVM spec permits waking without any `notify()` at all
2. `notifyAll()` wakes everyone, but only some may have their condition actually satisfied
3. **Stale state race** — between wake and lock re-acquisition, another thread may have already consumed the resource

> [!danger] Trap — concrete failure with `if`
> Two consumers both find queue empty → both `wait()`. Producer adds ONE item, calls `notifyAll()`. Consumer A wakes first, takes the item. Consumer B wakes next — with `if`, B skips re-checking and calls `remove()` on now-empty queue → crash/bad state. `while` forces B to recheck and go back to `wait()`.

### Edge cases

| Scenario | Behavior |
|---|---|
| `notify()` with no thread waiting | Signal is **lost** — not queued for a future `wait()` |
| `Thread.sleep()` polling loop vs `wait()`/`notify()` | Polling = busy-wait, wastes CPU, adds latency (reacts only on next tick). `wait()` = zero CPU while parked, reacts instantly |
| `notify()` called, then more work before unlocking | **Not a deadlock** — just delayed handoff; woken thread blocks re-acquiring lock until notifier releases |
| Order threads re-acquire lock after `notifyAll()` | **Not guaranteed FIFO** — unspecified, JVM-scheduler dependent |
| Thread interrupted while in `wait()` | `wait()` throws `InterruptedException` immediately (after re-acquiring lock) — standard way to cancel a waiting thread |

**Mnemonic:** *Check with `while`, not `if` — the world can change while you slept.*

---

## 6. `volatile`

![[volatile-visibility.svg]]

- Guarantees **visibility** (every read sees the latest write, no stale CPU-cache copies) and **ordering** (happens-before — no reordering across a volatile write/read boundary)
- Does **NOT** guarantee atomicity for compound operations

> [!danger] Trap — classic stop-flag bug
> ```java
> private boolean running = true;   // NOT volatile
> public void run() { while (running) { /* work */ } }
> public void shutdown() { running = false; }
> ```
> JIT may cache `running` in a register inside the loop — worker thread may **never** see the update, loops forever. Fix: `volatile boolean running`.

### What volatile does and doesn't cover

| Scenario | Safe with volatile alone? |
|---|---|
| Single read/write of the field | Yes — atomic, no tearing |
| Compound op (`i++`, `x = !x`) | **No** — still Read-Modify-Write, still races |
| Mutations on a referenced object (`volatile List`, then `list.add()`) | **No** — reference is fresh, object mutations aren't covered |
| `long`/`double` read/write | Yes — `volatile` specifically fixes **word tearing** (64-bit split into two 32-bit ops on some JVMs without it) |

### `volatile` vs `synchronized`
- `synchronized` also establishes happens-before → gives visibility too, without needing `volatile`, as long as *all* access to that field goes through the synchronized block/method consistently
- `volatile` = visibility + ordering only, no mutual exclusion
- `synchronized` = visibility + ordering + mutual exclusion, but costs more (blocking)

### Classic correct use cases
- Single flag, one writer, many readers, no compound logic (stop-flag pattern above)
- Double-checked locking singleton — instance reference must be `volatile` (visibility + ordering, prevents exposing a half-constructed object)

### Does `volatile` make the whole class thread-safe?
- No — scoped to that one field only. Other fields and any compound/multi-step logic remain unprotected.

**Mnemonic:** *volatile = fresh reads, not safe writes.*

---
# 7. Deadlock, Livelock & Starvation

![[deadlock-livelock-starvation.svg]]

- **Deadlock** — threads **blocked** forever, each holding a resource the other needs
- **Livelock** — threads **active**, keep changing state in response to each other, never progress
- **Starvation** — a thread perpetually denied CPU/resource access due to scheduling unfairness (no blocking, no looping — just bad luck)

> [!tip] Mnemonic
> Deadlock = frozen. Livelock = spinning. Starvation = ignored.

## Coffman conditions (all 4 required for deadlock)

| Condition | Meaning |
|---|---|
| Mutual exclusion | Resource held by only one thread at a time |
| Hold and wait | Thread holds one resource while waiting for another |
| No preemption | Resource can't be forcibly taken, only voluntarily released |
| Circular wait | Cycle of threads each waiting on the next |

> [!danger] Classic deadlock trap
> `transfer(A, B)` and `transfer(B, A)` both doing `synchronized(from) { synchronized(to) { ... } }` called concurrently in opposite directions → circular wait → deadlock, even though the code "looks fine" in isolation.

## Fix: lock ordering

![[lock-ordering-fix.svg]]

- Always acquire locks in a **fixed, consistent order** (e.g. by account ID/hashcode), regardless of operation direction
- Structurally kills circular wait — no thread ever holds one lock while blocked on the other's held lock
- Alternative: `tryLock()` + timeout + backoff — works, but more complex and can itself livelock without jitter

## Deadlock vs livelock vs starvation

| | State | Root cause | Fix |
|---|---|---|---|
| Deadlock | Blocked | Circular wait on held locks | Lock ordering, `tryLock` + timeout |
| Livelock | Active, no progress | Threads keep reacting to each other's state changes | Randomized backoff (jitter) |
| Starvation | Some threads never scheduled | Unfair scheduling/lock granting | Fair locks (`ReentrantLock(true)`) |

## Self-deadlock

> [!warning] A single thread CAN deadlock itself
> - `Thread.currentThread().join()` — thread waits on itself to finish, which can never happen
> - Recursive acquisition of a **non-reentrant** lock — thread already holds it, tries again, blocks waiting on its own release
> - `synchronized` and `ReentrantLock` are both **reentrant** by design specifically to prevent this — hold-count increments instead of blocking. This is why people wrongly assume self-deadlock is impossible.

## Priority vs fairness — don't mix these up

| | Thread priority | Lock fairness |
|---|---|---|
| Controls | Scheduling hint — which thread gets CPU time | Which waiting thread gets a contended lock next |
| Mechanism | `Thread.setPriority()` — advisory, platform-dependent, often ignored | `new ReentrantLock(true)` — FIFO wait queue |
| Solves | Nothing reliably guaranteed | Starvation on that lock |
| Cost | N/A | Throughput hit — no barging, more bookkeeping |

> [!tip] Mnemonic
> Priority is a suggestion to the scheduler. Fairness is a promise from the lock.

## Does `ConcurrentHashMap` prevent deadlock?

> [!warning] No
> `ConcurrentHashMap` only protects its own internal state via fine-grained locking/CAS. It does **not** throw on concurrent modification the way a fail-fast `HashMap` iterator does (that's `ConcurrentModificationException`, unrelated). If your own code takes multiple external locks — including `synchronized` on a `ConcurrentHashMap` instance combined with other locks — in inconsistent order, deadlock is still fully possible.
# 8. Thread Pools & ExecutorService

![[jvm-os-thread-mapping.svg]]

- Every `java.lang.Thread` maps **1:1 to a native OS thread** on HotSpot — JVM asks OS for a real kernel thread, OS scheduler time-slices across CPU cores
- Thread creation is expensive precisely because it's an **OS-level cost**, not just JVM
- No hard JVM cap on thread count, but bounded by: stack memory per thread (~512KB–1MB, tunable via `-Xss`), OS thread limits (`ulimit -u`), and context-switch overhead long before either wall is hit

> [!tip] Mnemonic
> Classic `Thread` = always 1:1 with the OS. (Virtual threads break this — covered separately.)

## `Executors` factory methods

| Method | Threads | Queue | Use case |
|---|---|---|---|
| `newFixedThreadPool(n)` | Fixed at n | Unbounded `LinkedBlockingQueue` | Steady predictable load; risk: unbounded queue growth → OOM under sustained backlog |
| `newCachedThreadPool()` | Unbounded, on demand | `SynchronousQueue` (direct hand-off) | Many short-lived bursty tasks; risk: unbounded thread creation |
| `newSingleThreadExecutor()` | Exactly 1 | Unbounded | Strict sequential/ordered execution |
| `newScheduledThreadPool(n)` | Fixed at n | — | Delayed/periodic tasks |

![[cached-thread-pool-dynamics.svg]]

> [!warning] Interview trap
> Most `Executors.new*` factories use unbounded queues or unbounded thread creation — production code often constructs `ThreadPoolExecutor` directly with explicit bounds instead.

## `ThreadPoolExecutor` core parameters

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

## Rejection policies

| Policy | Behavior |
|---|---|
| `AbortPolicy` (default) | Throws `RejectedExecutionException` |
| `CallerRunsPolicy` | Submitting thread runs the task itself — natural backpressure |
| `DiscardPolicy` | Silently drops the task |
| `DiscardOldestPolicy` | Drops oldest queued task, retries submit |

> [!tip] Why `CallerRunsPolicy` is a common production default
> Converts overload into backpressure — slows the producer instead of failing/dropping. Downside: blocks the caller thread doing pool work, degrading its own throughput if it happens often.

## `shutdown()` vs `shutdownNow()`

| | `shutdown()` | `shutdownNow()` |
|---|---|---|
| Behavior | Graceful — stops new tasks, lets queued + running finish | Aggressive — interrupts running tasks, returns unstarted queued tasks |
| Guarantee | All in-flight work completes | None — task must cooperate with interrupt |

> [!warning] `awaitTermination(timeout, unit)` after `shutdown()`
> Blocks the caller up to `timeout` waiting for termination. Returns `true` if finished in time, `false` if tasks are still running — and in the `false` case, **nothing is forcibly stopped**; tasks keep running in the background. Common idiom: call `shutdownNow()` as a fallback when `awaitTermination` returns `false`.

> [!tip] Mnemonic
> `shutdown()` finishes the queue. `shutdownNow()` tries to cut the line and interrupt whoever's talking.
# 9. Virtual Threads & Java 17-21 Features

![[virtual-vs-platform-threads.svg]]

- Solves: platform threads are 1:1 with OS threads (expensive, capped at low thousands) — virtual threads let you write plain **blocking, sequential-style code** at massive scale (millions) without going reactive/async
- Virtual threads are **M:N** — many share a small pool of carrier platform threads
- On a blocking call, JVM **unmounts** the virtual thread from its carrier, freeing the carrier for other work; remounts (possibly on a different carrier) once unblocked
- Stack lives on the heap, grows/shrinks dynamically — not a fixed OS stack

## Platform vs virtual threads

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

## Structured concurrency (JEP 453, preview)

- Treats a group of subtasks in separate threads as **one unit of work with one lifetime** — if one fails, siblings are auto-cancelled, nothing leaks past the parent's scope

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var user = scope.fork(() -> fetchUser());
    var order = scope.fork(() -> fetchOrder());
    scope.join().throwIfFailed();
}
```

## Java 17-21 grab-bag

| Feature | Version | One-liner |
|---|---|---|
| Record patterns | 21 | Deconstruct records in `instanceof`/`switch`: `if (obj instanceof Point(int x, int y))` |
| Sequenced Collections | 21 | `SequencedCollection`/`SequencedSet`/`SequencedMap` — uniform `getFirst()`, `getLast()`, `reversed()` |
| Pattern matching for switch, Records, Sealed interfaces | 17/21 | Already covered in Java/Java Core.md |
# 10. Atomic Classes & CAS

- **CAS (compare-and-swap)** — a single atomic CPU instruction: read current value, compare to expected, swap in new value if they match, else no-op. All three steps happen as one indivisible hardware operation — no thread can interleave between read and write
- This is what "lock-free" means: no mutex, no thread ever blocked/parked — a failed CAS just means the thread loops and retries with fresh data

```java
// Conceptually what incrementAndGet() does
int current;
int next;
do {
    current = get();
    next = current + 1;
} while (!compareAndSet(current, next)); // retries on failure, never throws/blocks
```

> [!tip] Mnemonic
> Lock = blocked and parked by the OS. CAS failure = still runnable, just loops and retries.

## `volatile` vs `AtomicInteger`

| | `volatile int` | `AtomicInteger` |
|---|---|---|
| Guarantees | Visibility only — every read sees latest write | Visibility **and** atomicity of read-modify-write |
| Safe `i++`? | No — three separate steps can interleave | Yes — CAS loop makes the whole operation atomic |

## `compareAndSet(expected, newValue)`

- Returns `true` on success, `false` if current value didn't match `expected`
- Manual retry pattern:
```java
int current;
do {
    current = atomicVar.get();
} while (!atomicVar.compareAndSet(current, current + 1));
```

## CAS under very high contention

> [!warning] CAS is not always faster than locking
> At **extreme contention**, many threads repeatedly CAS-fail on the same cache line — wasted CPU cycles plus heavy cache-line bouncing across cores. A blocked thread on a lock, by contrast, is parked by the OS and burns no CPU while waiting. At low/moderate contention CAS clearly wins (no block/wake overhead); at extreme contention, locks can win.

## `LongAdder` — the fix for hot counters

- A single `AtomicLong` under high-contention counting becomes a bottleneck — every thread CASes the *same* location
- `LongAdder` internally stripes the counter across **an array of separate cells**; each thread updates a different cell (thread-local hash), spreading contention
- `sum()` adds all cells for the total
- **Tradeoff:** reads are more expensive/approximate mid-update — use for write-heavy, read-rarely counters (metrics, request counts), not when you need a precise value on every read (use `AtomicLong` there instead)

## The ABA problem

> [!danger] CAS can be fooled by A → B → A
> CAS only checks "is the value still what I expect" — it can't detect a value that changed and changed back before the CAS ran. Classic case: lock-free stack where the top node is popped, other nodes come and go, then a node with the same reference is pushed back — CAS sees "unchanged" and proceeds incorrectly.
> **Fix: `AtomicStampedReference`** — CASes on a `(value, stamp)` pair; the stamp increments on every update, so even a return-to-`A` is detected via the changed stamp. (`AtomicMarkableReference` is a lighter boolean-mark variant.)

## CAS across multiple variables

> [!warning] CAS protects exactly one memory location — never several independent ones at once
> If `balance` and `transactionCount` must update together atomically, CAS alone can't do it directly. Options:
> - Combine both into a single **immutable object** and CAS the *reference* (`AtomicReference<State>`), replacing the whole state at once
> - Fall back to `synchronized`/`ReentrantLock` around both updates

> [!tip] Mnemonic
> One CAS, one location. Multiple variables together → wrap in one object, or use a lock.
