# Cover

What's left to cover, in priority order. Tick a line once the note is written **and** revised.

> [!note] How every note gets written
> - Enough to re-learn the topic in **one pass**, 2–3 months later, just before an interview
> - Per topic: the core idea → how it works → traps → likely questions → one-line mnemonic
> - Fragment bullets, tables for contrasts, callouts for traps. No essays, no textbook dumps
> - If a note runs past ~150 lines, it's too long: cut, don't add

---

## Tier 1: missing, almost certain to come up

- [ ] **[[Code Review, SOLID & Clean Code]]**
	- Review order: correctness → thread safety → exceptions → security → design → money → naming and tests
	- SOLID with one bad-code example each
	- Q: review this PR · which SOLID principle does this break · why `BigDecimal` for money
- [ ] **[[Lock Internals & Java Memory Model]]**
	- `synchronized`: monitorenter/monitorexit, object header (mark word), lock inflation
	- `volatile`: memory barriers, happens-before rules
	- `ReentrantLock` internals: AQS, CAS + wait queue
	- Q: how does synchronized work internally · how is volatile implemented · what is happens-before
- [ ] **[[CompletableFuture, Future & ForkJoin]]**
	- `Future` vs `CompletableFuture` · `thenApply` / `thenCompose` / `thenCombine` / `allOf`
	- Exception handling: `exceptionally`, `handle`, `whenComplete`
	- Common pool vs custom executor · ForkJoin work stealing
	- Q: production issues (blocking `get()`, lost exceptions, pool starvation)
- [ ] **[[Kafka]]**
	- Broker, topic, partition, replication, ISR
	- Ordering guarantee · consumer groups · rebalancing · offsets and commits
	- `acks`, idempotent producer, at-least-once vs exactly-once
	- DLQ / retry topics · throughput levers · Kafka vs SQS
- [ ] **[[Transactions, Isolation & Locking]]**
	- ACID · 4 isolation levels × anomalies (dirty, non-repeatable, phantom)
	- Optimistic (`@Version`) vs pessimistic (`SELECT ... FOR UPDATE`)
	- Q: how do you prevent a double debit
- [ ] **[[JPA Performance (N+1, Fetching, Caching)]]**
	- N+1 and fixes: `JOIN FETCH`, `@EntityGraph`, batch size
	- Lazy vs eager · `LazyInitializationException` · entity states
	- First-level vs second-level cache
- [ ] **[[Rate Limiter]]** (Hello Interview format)
	- Token bucket vs fixed / sliding window
	- Redis + Lua, distributed counters, sharding by key
	- Redis down: fail-open vs fail-closed · 429 + `Retry-After`
- [ ] **[[Caching & Redis]]**
	- Cache-aside, read-through, write-through, write-back, write-around
	- Eviction (LRU, TTL) · invalidation · cache stampede · Redis data types
- [ ] **[[JVM Memory & Garbage Collection]]**
	- Heap, stack, metaspace · young / old generation
	- G1 vs ZGC · Java memory leaks · OOM types · heap dump reading
- [ ] **[[Region Migration (RFE)]]**
	- 5-minute walkthrough
	- The 4 pieces I wrote, with numbers: proxy routing / fallback, data reconciliation, traffic-shift + capacity, rollback + monitoring
- [ ] **[[Story Bank]]**
	- Conflict · feedback · performance fix · owned alone · choice between two options
	- Why JPMC · why leaving Amazon after a year

---

## Tier 2: exists, but too shallow

- [ ] [[Atomics & Concurrent Collections]]: ConcurrentHashMap: why no nulls, `computeIfAbsent` atomicity + recursion trap, how `size()` works
- [ ] [[Executors & Virtual Threads]]: pool sizing (CPU-bound vs I/O-bound)
- [ ] [[ThreadLocal & BlockingQueue]]: ThreadLocal leak in thread pools · producer-consumer with `BlockingQueue`
- [ ] [[IoC, DI & Beans]]: circular dependencies · `@Configuration` vs `@Component` · Spring Boot startup flow
- [ ] [[REST Controllers & Exception Handling]] + [[API Design & Idempotency]]: status codes · idempotent methods · idempotency key · pagination · versioning
- [ ] [[SQL]]: Nth-highest salary · dept with no employees · manager self-join · duplicates · `EXISTS` vs `IN`
- [ ] [[Indexing]]: composite index + leftmost prefix · when an index isn't used · `EXPLAIN` · B+ tree vs hash
- [ ] [[Lambda]]: cold starts · reserved vs provisioned concurrency · memory / CPU / timeout limits · SQS-trigger retries
- [ ] [[Notification System]]: fill the empty template

---

## Tier 3: only if time remains

- [ ] Spring Security + JWT
- [ ] AOP, filters vs interceptors, `@Async`
- [ ] API Gateway, ALB, ECS
- [ ] Sharding, replication, CAP, consistent hashing
- [ ] Payment / balance service HLD
