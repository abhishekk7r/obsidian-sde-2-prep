
- Ensures **only one thread** executes a critical section at a time.
- Prevents **race conditions** when multiple threads modify shared data.
- Without `synchronized`, two threads can execute `count++` simultaneously.

#### How Synchronization works in java?

Every Java object has an **Intrinsic Lock (Monitor)**.

```
When a thread enters a synchronized block:
- Acquires the monitor lock.
- Executes the code.
- Releases the lock.
- Other threads wait until the lock is released.
```

