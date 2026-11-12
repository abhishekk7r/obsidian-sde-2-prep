# Thread Coordination

## wait() / notify() / notifyAll()

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

## Semaphore / CountDownLatch / CyclicBarrier

![[semaphore-latch-barrier.svg]]

- **Semaphore** — N permits, up to N threads hold concurrently; (N+1)th blocks. Not exclusive like a lock — any thread can `release()`, not just the acquirer
- **`CountDownLatch`** — one-time gate: count decrements via `countDown()`, `await()` blocks until zero. **Never resets** — single-use
- **`CyclicBarrier`** — fixed group of N threads wait for each other at `await()`; trips together (optional barrier action), **auto-resets** for reuse

```java
// Semaphore — cap concurrent calls to a rate-limited API
Semaphore limiter = new Semaphore(5);
limiter.acquire();
try { callApi(); } finally { limiter.release(); }

// CountDownLatch — main waits for N services to finish init
CountDownLatch initLatch = new CountDownLatch(3);
// each service: initLatch.countDown();  main: initLatch.await();

// CyclicBarrier — 4 threads sync after each round of parallel computation
CyclicBarrier barrier = new CyclicBarrier(4, () -> System.out.println("round done"));
// each worker: doRound(); barrier.await();
```

| | Real-life analogy | Who counts/waits | Reusable? |
|---|---|---|---|
| Semaphore | Parking garage, N spots | Any thread acquires/releases | N/A — ongoing |
| `CountDownLatch` | Rocket launch — 4 systems report ready, control waits | Workers count down, **different** thread(s) wait | No — one-shot |
| `CyclicBarrier` | Hiking group regroups at each checkpoint | **Same** threads both trip and wait | Yes — auto-resets |

> [!tip] Mnemonic
> Latch: different threads count down vs wait, opens once. Barrier: same threads wait for each other, resets every round.
