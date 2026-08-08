
- Ensures **only one thread** executes a critical section at a time.
- Prevents **race conditions** when multiple threads modify shared data.
- Without `synchronized`, two threads can execute `count++` simultaneously.