# Atomics & Concurrent Collections

## Atomic Classes & CAS

![[cas-retry-loop.svg]]

- **CAS** — atomic CPU instruction: read → compare to expected → swap if match, else no-op. Never blocks; failure just retries
- "Lock-free" = thread stays runnable on failure, just loops — never parked like a blocked lock-waiter

**`volatile` vs `AtomicInteger`:**

| | `volatile int` | `AtomicInteger` |
|---|---|---|
| Guarantees | Visibility only | Visibility + atomicity |
| Safe `i++`? | No | Yes (CAS loop) |

- `compareAndSet(expected, new)` → returns `true`/`false`; manual retry = `while(!cas(current, current+1)) current = get();`

> [!warning] CAS ≠ always faster than locking
> Extreme contention → repeated failed retries burn CPU + bounce cache lines. Locks park failed threads (no CPU cost while waiting). Low/moderate contention → CAS wins clearly.

![[longadder-striping.svg]]

- `LongAdder` stripes a hot counter across cells, one per thread bucket — `sum()` aggregates on read
- Use for write-heavy/read-rarely counters (metrics); use `AtomicLong` when you need a precise value on every read

> [!danger] ABA problem
> CAS can't detect A → B → A — looks unchanged, but structure moved underneath. **Fix:** `AtomicStampedReference` — CASes on `(value, stamp)` pair, stamp increments on every update so a return-to-A is still caught. (`AtomicMarkableReference` = lighter boolean-mark variant.)

> [!warning] CAS = one memory location, never several at once
> Need `balance` + `transactionCount` atomic together? Wrap both in one immutable object and CAS the *reference*, or fall back to a lock.

> [!tip] Mnemonic
> One CAS, one location. Lock-free ≠ always faster. Stripe hot counters.

---

## Concurrent Collections

![[chm-vs-synchronizedmap-locking.svg]]

- **CHM:** thread safety via bucket-level locking, not one global lock
- **Java 8 rewrite:** pre-8 = fixed 16 `Segment`s (own lock each, capped concurrency). Java 8 = per-bin locking + CAS on empty bins + treeification (>8 collisions → red-black tree, O(log n) worst case)
- **Iteration:** weakly consistent — never throws CME (unlike `HashMap`'s fail-fast iterator)

> [!warning] `size()` is an estimate, not exact
> Sums per-bucket counts that can still be changing mid-call — no lock, no copy taken.

> [!tip] `putIfAbsent()` is atomic
> Only one thread ever inserts for a given key. Losers get the existing value back — never overwrite.

![[copy-on-write-mechanism.svg]]

- **`CopyOnWriteArrayList`:** every write copies the whole array, then atomically swaps the reference
- Reads are lock-free (just read current reference); iterators are bound to the array snapshot at creation — never see later writes, never throw CME

| | Cost |
|---|---|
| Reads | Fast, lock-free |
| Writes | O(n) — full copy every time |

> [!tip] Mnemonic
> Copy-On-Write: writers pay so readers never have to. Good for read-heavy/write-rare; bad for write-heavy.

#### vs `Collections.synchronizedX(...)`

| | `synchronizedMap`/`synchronizedList` | CHM / COW |
|---|---|---|
| Locking | Whole collection, every op | Fine-grained or none |
| Consistency | Strong | Weakly consistent / snapshot |
| Best for | Need strict read+write consistency together | High-concurrency reads or scale-out writes |
