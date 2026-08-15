# Concurrency

## 1. Process vs Thread

- **Process** — independent execution unit; own isolated address space (heap, code, data segments, own FDs)
- **Thread** — unit of execution *within* a process; own stack, program counter, registers — **shares** heap, code segment, open file/socket handles with sibling threads
- Threads communicate via shared heap → fast, but no fault isolation (one thread's bug can corrupt shared state for all)
- Processes are isolated → safer, but IPC (pipes/sockets/shared mem segments) is slower — serialization + OS-mediated

> [!tip] Why thread switch < process switch (cost)
> Thread switch: save/restore registers, PC, stack pointer only — same address space, no MMU/TLB touch.
> Process switch: all of the above **+ address space switch** — page table swap + TLB flush.

### Uncaught exceptions
- Any thread's uncaught exception → default `UncaughtExceptionHandler` prints trace, **only that thread dies** — others unaffected
- JVM exits only when **last non-daemon thread** finishes/dies (main thread is usually this)
- Daemon threads never keep JVM alive — irrelevant to JVM lifecycle whether they die by exception or normally

> [!danger] Trap
> Daemon vs non-daemon does NOT change *how* the exception is handled — same default handler either way. Only difference: does the JVM care that this thread is gone.

### Thread lifecycle — `Thread.State`
**NEW → RUNNABLE → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED**

| State | Meaning |
|---|---|
| NEW | Created, not started |
| RUNNABLE | Running or ready to run (Java doesn't distinguish) |
| BLOCKED | Waiting to *acquire* a monitor lock (stuck at `synchronized` entry) |
| WAITING | Voluntarily gave up lock — `wait()`, `join()`, `LockSupport.park()` — waits indefinitely for notify/completion |
| TIMED_WAITING | Same as WAITING but bounded — `sleep()`, `wait(ms)`, `join(ms)` |
| TERMINATED | Finished execution |

- Thread IDs **can be reused** after a thread terminates (JVM impl detail, not spec-guaranteed)

### `wait()` vs `sleep()`

| | `wait()` | `sleep()` |
|---|---|---|
| Belongs to | `Object` | `Thread` (static) |
| Releases lock held? | **Yes** | **No** |
| Must be in `synchronized`? | Yes (else `IllegalMonitorStateException`) | No |
| Wakes via | `notify()`/`notifyAll()` or timeout | Timer expiry or interrupt |
| Purpose | Inter-thread coordination | Pause execution, no coordination |

> [!danger] Trap
> `sleep()` inside a `synchronized` block still **holds the lock** the whole time — blocks other threads for nothing. `wait()` exists specifically to avoid this.

**Mnemonic:** *"sleep holds, wait yields."*

### `counter++` race condition
Not atomic — it's **Read → Modify (increment) → Write**. Two threads can both read the same stale value, increment locally, and the second write clobbers the first → lost update.

---

## 2. Race Conditions & Critical Sections

- **Critical section** — code accessing **shared mutable state**; protect via mutual exclusion (`synchronized`, locks, atomics) or by removing shared mutability (immutability)
- **Race condition** — correctness depends on unpredictable thread interleaving/timing. Broader concept.
- **Data race** — specific: same memory location, ≥1 write, no synchronization/happens-before order between accesses. Undefined behavior per JMM.

> [!tip] Distinction
> Every data race is a race condition. Not every race condition is a data race — can happen even with synchronization if the *logic* itself is order-dependent (e.g. two separately-synchronized calls used together).

### Check-then-act (TOCTOU)
- Classic race: `if (!map.containsKey(k)) map.put(k, v);` — two threads can both pass the check before either puts → lost/duplicate insert
- Fix: atomic compound methods — `putIfAbsent()`, `computeIfAbsent()`

### `volatile` ≠ atomicity
- `volatile` = visibility (fresh reads) + ordering (no reordering around it) — **NOT atomicity**
- `i++` on a volatile int is still Read→Modify→Write, still races. Use `AtomicInteger` for atomic increments.

### Broken double-checked locking (lazy singleton)
```java
if (instance == null) {
    synchronized(this) {
        if (instance == null) {      // inner check required — else two threads both run init
            instance = new Singleton();
        }
    }
}
```
- Missing inner check → multiple threads pass outer check, both initialize
- Flag/reference **must be `volatile`** — else instruction reordering can expose a half-constructed object to another thread (classic pre-fix JMM gotcha)

### Read-only data
- No writers → no race, regardless of thread count

### Synchronized methods ≠ thread-safe class
- Each method atomic individually; a **sequence** of two synchronized calls is not atomic as a unit
- Example: `Vector`/`Hashtable` — thread-safe per-method, but `if (!v.contains(x)) v.add(x)` still races

### Race condition vs Deadlock

| | Race condition | Deadlock |
|---|---|---|
| Symptom | Wrong/inconsistent data, intermittent | Program hangs, no progress |
| Reproducibility | Flaky, timing-dependent | Often reproducible under load |
| Cause | Missing/incorrect synchronization | Circular wait on locks |
| Fix | Proper synchronization | Lock ordering, `tryLock` with timeout |

**Mnemonic:** *Race = wrong answer. Deadlock = no answer.*

---

## 3. `synchronized` Keyword

- Guards **code paths**, not objects — an unsynchronized method on the same object is NOT blocked by another thread holding the lock via a synchronized method/block
- Backed by every object's implicit **monitor** (intrinsic lock) — only one thread holds it at a time, others go to **BLOCKED**

> [!danger] Trap
> `synchronized` protects code, not the object itself. Two threads can be inside a synchronized method and a plain method **simultaneously** on the same object — no blocking, since the plain method never checks the lock.

### Instance vs static synchronized

| | Locks on | Scope |
|---|---|---|
| `synchronized` instance method / `synchronized(this)` | the instance (`this`) | per-object — different instances don't block each other |
| `synchronized` static method / `synchronized(ClassName.class)` | the `Class` object | one per class — blocks across **all** instances |

> [!danger] Trap
> Instance-synchronized and static-synchronized methods on the same class **do not block each other** — different locks. Mixing `this` and `.class` locks to protect the *same* shared data is a bug.

### Reentrancy
- `synchronized` **is reentrant** — a thread already holding an object's monitor can re-enter other synchronized blocks/methods on the same object freely (JVM tracks a hold count)
- Nested synchronized calls, same thread, same object → **no self-deadlock**
- `ReentrantLock` is also reentrant (matches `synchronized` behavior) — its name refers to that guarantee, not something `synchronized` lacks. Extra features: `tryLock()`, timeouts, **fairness policy** (`new ReentrantLock(true)` — FIFO, longest-waiting thread first, prevents starvation), multiple `Condition` objects (multiple independent wait-queues vs. `synchronized`'s single wait-set), **interruptible** lock acquisition (`lockInterruptibly()` — can abort a blocked wait; `synchronized` cannot)

### Exception safety
- Lock is **always released** on exit from a synchronized block — normal or exceptional. JVM-guaranteed, acts like an implicit `finally`.

### Locking on the wrong object
- Never synchronize on `String` literals or cached boxed `Integer` (-128..127) — both are JVM-wide shared/interned objects, so you risk sharing a lock with unrelated code
- Always use a **private, dedicated lock object**: `private final Object lock = new Object();`

**Mnemonic:** *`synchronized` locks code, not objects — and it never locks the same thread out of itself.*

---
