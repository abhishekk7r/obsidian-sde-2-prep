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
