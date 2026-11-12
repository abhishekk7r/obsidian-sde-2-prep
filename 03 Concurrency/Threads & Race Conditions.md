# Threads & Race Conditions

## Process vs Thread

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

## Race Conditions & Critical Sections

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
