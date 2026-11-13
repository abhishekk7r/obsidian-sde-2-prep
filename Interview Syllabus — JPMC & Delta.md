---
tags: [syllabus, jpmc, delta]
updated: 2026-11-13
---

# Interview Syllabus — JPMC SWE II & Delta Sr SDE

## 1. The two loops

| | Delta Air Lines | JPMorganChase |
|---|---|---|
| Role | Senior SDE, Software Backend | Software Engineer II, Java/AWS (Req 210804303) |
| When | **Sun 15 Nov, 3:15–4:00 PM IST** | **Mon 16 Nov, 4–6 PM IST** |
| Format | 1 round, 45 min, Zoom, **recorded**, run by **BarRaiser** (3rd-party interviewer), screen share for coding | 2 × 60 min on Zoom, no OA |
| Interviewers | BarRaiser expert | 4 PM **Ankit Agarwal** (Associate, SWE III, ex-Nykaa payments) · 5 PM **Saurabh Kumar** (VP, Digital Banking) |
| Focus | **Java coding · Java internals & Spring Boot · AWS services** | R1: PR review + easy–medium DSA · R2: HLD + project + resume depth + behavioural |

> [!warning] Language
> Code in **Java** for both. Use `var` where allowed. C++ only as an emergency fallback, after asking.

---

## 2. What gets asked (from ~31 JPMC write-ups)

| Topic | Frequency | Examples |
|---|---|---|
| **PR / code review** | ~16 of 31 | Spring Boot PR, static/volatile misuse, `double` for money, swallowed exceptions, SQL injection |
| **DSA, easy–medium** | Nearly every loop | Circular dials, circular array, duplicates, sort chars by frequency, min-difference pairs, largest square, sliding window |
| **Concurrency on the same code** | ~11 | synchronized, volatile, ConcurrentHashMap, race in the reviewed snippet |
| **Core Java** | ~8 | HashMap internals, equals/hashCode, String immutability, streams, Java 17/21 |
| **HLD** | Most loops | **Rate limiter (3×)**, parking lot, Drive-like storage, monolith → microservices, flight search |
| **Resume depth (VP)** | Where rejected | Optimistic vs pessimistic locking, Kafka (brokers, DLQ, throughput), WebFlux, virtual threads |
| **Spring Boot** | ~6 | `@SpringBootApplication`, `@Transactional`, microservices |
| **SQL** | ~4 | Joins, GROUP BY / HAVING, nested query |
| **Behavioural** | Every loop | Why JPMC, conflict, feedback, a hard task owned |

> [!note] Never seen in SWE II write-ups
> Hard DP, segment trees, tries, bit tricks, full-code LLD.

---

## 3. DSA — new problems (JPMC-reported or gap-fillers)

| Problem | Pattern |
|---|---|
| [Open the Lock](https://leetcode.com/problems/open-the-lock/) | BFS on states (circular dials) |
| [Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/) | Monotonic stack, circular |
| [Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/) | Kadane, circular |
| [Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array/) | Negative marking / cyclic sort |
| [Sort Characters by Frequency](https://leetcode.com/problems/sort-characters-by-frequency/) | Count + bucket / heap |
| [Minimum Absolute Difference](https://leetcode.com/problems/minimum-absolute-difference/) | Sort + neighbours |
| [Maximal Square](https://leetcode.com/problems/maximal-square/) | 2D DP |
| [Count Substrings Without Repeating Character](https://leetcode.com/problems/count-substrings-without-repeating-character/) | Sliding window |
| [Divide Intervals Into Minimum Number of Groups](https://leetcode.com/problems/divide-intervals-into-minimum-number-of-groups/) | Sweep line / heap |
| [Car Pooling](https://leetcode.com/problems/car-pooling/) | Difference array |
| [Minimum String Length After Removing Substrings](https://leetcode.com/problems/minimum-string-length-after-removing-substrings/) | Stack deletion |
| [Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Dijkstra |
| [Coin Change](https://leetcode.com/problems/coin-change/) | 1D DP |
| [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) | 1D DP |
| [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/) | 2D DP (lost Amex R1) |

**Revise from the sheet (recode in Java):** Merge Intervals · Group Anagrams · Longest Consecutive · Top K Frequent · Max Consecutive Ones III · Min Window Substring · Longest Substring w/o Repeat · Koko · Daily Temperatures · Decode String · Asteroid Collision · LRU Cache · Course Schedule I+II · Walls and Gates · Shortest Bridge · House Robber I–III · Min Cost for Tickets · LCS. Rest of the sheet: verbal only.

> [!tip] Order
> Revise a sheet problem, then the new one with the same pattern: intervals → arrays → hashing → stack → sliding window → graph → DP.

---

## 4. Java depth (Delta focus + JPMC VP)

- **Code review checklist** → [[2.6 Code Review, SOLID & Clean Code]]
- **HashMap / ConcurrentHashMap internals** → [[2.2 HashMap & equals-hashCode]], [[3.4 Atomics & Concurrent Collections]]
- **synchronized / volatile internals, JMM** → [[3.2 synchronized, volatile & Locks]], [[3.3 Lock Internals & Java Memory Model]]
- **Executors, CompletableFuture, virtual threads** → [[3.7 Executors & Virtual Threads]], [[3.8 CompletableFuture, Future & ForkJoin]]
- **JVM memory & GC** → [[2.5 JVM Memory & Garbage Collection]]
- **Spring:** IoC/beans, `@Transactional` traps, JPA N+1
- **Optimistic vs pessimistic locking**, isolation levels
- **Kafka basics** (on resume)

## 5. AWS (Delta focus)

- **Lambda:** cold starts, concurrency, memory/timeout
- **SQS + DLQ:** visibility timeout, maxReceiveCount, idempotent consumers
- **DynamoDB:** partition/sort key, hot partitions, vs RDS
- **Resilience:** retries + backoff + jitter, circuit breaker

## 6. HLD (JPMC R2) — Hello Interview format

1. **Rate limiter** — token bucket, Redis + Lua, Redis-down fail-open/closed, 429
2. Parking lot or notification system
3. Payment / idempotency if time

## 7. Project & behavioural

- **Region migration (RFE)** told as code you wrote → [[8.1 Region Migration (RFE)]]
- 5 STAR stories → [[8.2 Story Bank]]
- Why JPMC / why Delta / why leaving Amazon after a year

> [!tip] Mnemonic
> **Java first, DSA daily, rate limiter ready, project as code.**
