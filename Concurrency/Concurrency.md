# Concurrency

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
