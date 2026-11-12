
## `synchronized` Keyword

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

## `volatile`

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

## Locks — ReentrantLock / tryLock / Fairness

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
