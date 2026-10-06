---
tags: [interview, syllabus, sde2, google, big-tech]
created: 2026-10-07
prep-window: 2026-10-07 to 2026-12-29
interview-window: 2027-01 to 2027-03
---

# Big Tech SDE2 Syllabus (Oct 2026 → Mar 2027)

> [!note] Goal
> - **Oct–Dec:** finish Striver A2Z + LLD + HLD + behavioral stories
> - **Jan–Mar:** interview loops, maintenance mode only
> - Targets: **Google L4**, Uber SDE2, Microsoft SDE2, Atlassian P40, plus other SDE2 loops

Links: [[Daily Protocol]] · [[Active Recall Tracker]] · [[Mistake Log]] · [[Interview Prep Syllabus]] (older 1 hr/day plan)

---

## 1. What the loops actually test

| Company | Coding | LLD / machine coding | HLD | Behavioral |
|---|---|---|---|---|
| **Google L4** | GHA + 1 phone screen + 2–3 onsite coding, 45 min each, no code execution | No standalone round | **Sources disagree**: Hello Interview says none at L4; India guides report 1 round. Prep it anyway | 1 Googleyness round |
| **Uber SDE2** | OA (4 problems, 70–90 min) + phone screen + onsite DSA | 1 machine coding round (45–60 min) | 1 round, Uber-style scenarios | Collaboration / leadership |
| **Microsoft SDE2** | Codility (2 problems) + 2 DSA rounds | 1 LLD round (e.g. parking system) | Usually in HM / AA round | HM round |
| **Atlassian P40** | Karat screen + Data Structures round | Code Design round (e.g. subscription system) | 1 round: MVP first, then scale | Values + Management rounds |

> [!warning] Hiring committee weighs coding most at Google
> - DSA is the deciding factor for Google L4
> - LLD + HLD decide Uber, Microsoft, Atlassian
> - So: DSA every day, design on weekends

### Past failures: all graph / DP family

| Company | Question | Root cause |
|---|---|---|
| Moody's | LC 983 Min Cost Tickets | Interval DP not internalized |
| Visa | — | DSA round |
| Lenskart | Topological sort | Wrong edge direction + Java syntax |
| Amex | **Longest Palindromic Subsequence** | Brute force only; didn't derive memo `f(i,j)` |

> [!danger] Graphs and DP sit at the end of A2Z
> - Linear order = graphs/DP only in Nov–Dec
> - Fix: compress Steps 1–5 into 2 weeks, plus **1 graph/DP re-solve every weekend from Week 1**

---

## 2. Weekly time budget

> [!note] Assumption: about 20 hrs/week alongside work. Adjust the weekly counts if this is off.

| Day | DSA | Design | Recall |
|---|---|---|---|
| Mon–Fri | 1.5 hr (new A2Z problems) | — | 30 min ([[Active Recall Tracker]] due items) |
| Sat | 2 hr | 2.5 hr **LLD** | 30 min |
| Sun | 2 hr | 2.5 hr **HLD** | 30 min (weekly review) |

### Rules for every DSA problem
- Say brute force → optimal → complexity aloud **before** coding
- Solved before? Re-solve from blank, no notes; under 15 min = mark done
- Stuck 25 min → read the approach only, then code it yourself
- Log a one-line **pattern trigger** + pitfall in sheet notes
- Re-test on day 3, day 7, day 14 via [[Active Recall Tracker]]
- Repeated bug → [[Mistake Log]]

> [!tip] Skip rule
> - **Step 1, Step 2, easy Step 3:** skim; code only what you can't solve in 5 min in your head
> - Marked *(opt)* below: do only if the week is on schedule

---

## 3. DSA: Striver A2Z (474 problems, 18 steps)

> [!note] Counts per step are approximate. Tick against the live sheet.

| Week | Dates | A2Z steps | ~Problems | Weekend graph/DP re-solve |
|---|---|---|---|---|
| W1 | Oct 7–13 | Step 1 Basics (skim), Step 2 Sorting (merge, quick, insertion), Step 3 Arrays easy → hard | ~70, most skimmed | LC 983 Min Cost Tickets |
| W2 | Oct 14–20 | Step 4 Binary Search (1D, on answers, 2D), Step 5 Strings basic/medium | ~45 | Course Schedule II (edge direction) |
| W3 | Oct 21–27 | Step 6 Linked List (1D, DLL, medium, hard), Step 7 Recursion (subsequences, hard) | ~55 | Longest Palindromic Subsequence |
| W4 | Oct 28–Nov 3 | Step 8 Bit Manipulation (core only, rest *(opt)*), Step 9 Stack & Queue (monotonic, implementation) | ~45 | Coin Change + Word Break |
| W5 | Nov 4–10 | Step 10 Sliding Window & Two Pointer, Step 11 Heaps, Step 12 Greedy | ~45 | Rotting Oranges + Clone Graph |
| W6 | Nov 11–17 | Step 13 Binary Trees (traversals, views, LCA, construction, hard) | ~40 | House Robber I/II/III |
| W7 | Nov 18–24 | Step 14 BST, Step 15 Graphs part 1 (BFS/DFS, grid, cycle, bipartite, topo sort) | ~35 | LCS + Edit Distance |
| W8 | Nov 25–Dec 1 | Step 15 Graphs part 2 (Dijkstra, Bellman-Ford, Floyd, MST, DSU, bridges *(opt)*) | ~30 | Interval DP: Burst Balloons |
| W9 | Dec 2–8 | Step 16 DP part 1 (1D, 2D grid, subsequences / knapsack) | ~30 | — (DP is the main track) |
| W10 | Dec 9–15 | Step 16 DP part 2 (strings, stocks, LIS, partition / MCM), Step 17 Tries | ~35 | — |
| W11 | Dec 16–22 | Step 18 Strings hard (KMP, Z, Rabin-Karp *(opt)*), then **unlabeled mixed sets**: Google-tagged mediums, 2 per day, no topic shown | ~25 | Re-solve every ⚠ in Mistake Log |
| W12 | Dec 23–29 | **Mocks**: 4 timed (45 min, 1–2 problems, talk aloud), redo every miss | — | — |
| Buffer | Dec 30–Jan 5 | Catch up on any unfinished step | — | — |

### Highest-yield topics for Google (prioritise if behind)
- **Graphs:** BFS/DFS on grids, topo sort, Dijkstra, DSU
- **DP:** 1D, grid, LCS family, interval DP, knapsack
- **Arrays/strings:** sliding window, two pointers, prefix sums, intervals
- **Trees:** LCA, path sums, serialize/deserialize
- **Binary search on the answer** (Koko, Aggressive Cows, Split Array)

> [!warning] A2Z gap
> - The sheet labels every problem with its topic; interviews don't
> - W11 unlabeled sets + W12 mocks exist to close this

---

## 4. LLD (Saturdays)

Framework (Hello Interview): **Requirements → Entities & Relationships → Class Design → Implementation → Extensibility**
Language: Java. Folder: `LLD/` · patterns notes in `Design Patterns/`

| Week | Topic | Deliverable |
|---|---|---|
| W1 | OOP, SOLID, composition vs inheritance | Notes refresh |
| W2 | Patterns: Strategy, Factory, Observer, Singleton, Builder | One example each |
| W3 | Patterns: State, Decorator, Command, Chain of Responsibility | One example each |
| W4 | [[Parking Lot]] redo (cold, 60 min) + [[Design Elevator]] redo | Timed |
| W5 | Movie Ticket Booking (seat locking, concurrency) | Full design + code |
| W6 | Rate Limiter (token bucket, thread-safe) + LRU / LFU Cache | Code + concurrency |
| W7 | Splitwise + Amazon Locker | Full design |
| W8 | Logging Service + Pub-Sub / message queue | Full design |
| W9 | File System + Inventory Management | Full design |
| W10 | Subscription system (Atlassian) + Task Scheduler | Full design |
| W11 | Connect Four / Tic-Tac-Toe + Snake & Ladder (game-style machine coding) | 90-min timed builds |
| W12 | [[Vending Machine]] redo (State pattern) + 1 mock machine coding | Timed |

> [!tip] Machine coding bar (Uber, Flipkart style)
> - Working, runnable code in 60–90 min
> - Clean interfaces, in-memory storage, a demo `main`
> - Extensibility talk at the end

---

## 5. HLD (Sundays)

Framework (Hello Interview): **Requirements (functional, non-functional) → Core Entities → API → High-Level Design → Deep Dives**

### Concepts and technologies (W1–W4)
| Week | Concepts | Technologies |
|---|---|---|
| W1 | Networking, API design (REST, gRPC, pagination), numbers to know | API Gateway |
| W2 | Data modeling, DB indexing, SQL vs NoSQL | PostgreSQL, DynamoDB, Cassandra |
| W3 | Caching, sharding, consistent hashing, CAP | Redis |
| W4 | Patterns: real-time updates, contention, scaling reads/writes, large blobs, long-running tasks | Kafka, Elasticsearch, ZooKeeper |

### Problems (W5–W12, 2 per Sunday)
| Week | Problems |
|---|---|
| W5 | Bitly (URL shortener), Rate Limiter (distributed) |
| W6 | [[Notification System]] redo, [[Design Top K Heavy Hitters]] redo |
| W7 | WhatsApp (chat), FB News Feed |
| W8 | Ticketmaster (contention), Uber (geo + matching) |
| W9 | Dropbox (blobs), YouTube (video upload + streaming) |
| W10 | Web Crawler, Ad Click Aggregator (stream processing) |
| W11 | Payment System (idempotency), Job Scheduler |
| W12 | Google Docs (collaboration), Distributed Cache, 1 mock |

> [!tip] Your edge
> - Use real Amazon experience in deep dives: SQS + DLQ retries, DynamoDB key design, Kafka consumer groups, region migration
> - Interviewers reward "I've run this in production" trade-offs

---

## 6. Behavioral (December, 2 hrs/week from W9)

- **Google Googleyness:** ambiguity, feedback, challenging the status quo, user focus, integrity, collaboration
- **Atlassian:** values round + management round (your individual impact)
- **Uber:** collaboration, metrics-backed impact

### Story bank (STAR, 8 stories, 2 min each)
- [ ] Region migration (Dublin → Spain): ambiguity + ownership
- [ ] Agentic CI/CD / AI tooling: challenging the status quo
- [ ] Production incident or debugging under pressure
- [ ] Disagreement with a senior / pushback with data
- [ ] Mistake you made and fixed
- [ ] Mentoring or helping a teammate
- [ ] TCS BFSI work: user focus + delivery (Top 10% rating)
- [ ] Tight deadline / trade-off between scope and quality

---

## 7. Java and backend (light, as needed)
Only for Microsoft / fintech / product-company tech rounds:
- Collections internals (HashMap resize vs treeify, ArrayList), `equals` / `hashCode`
- Concurrency: `ExecutorService`, `CompletableFuture`, locks, `ConcurrentHashMap`, `start()` vs `run()`
- Spring Boot: DI, bean lifecycle, transactions, exception handling
- Folders: `Java/`, `Concurrency/`, `Spring Boot/`

---

## 8. Jan–Mar: interview phase

| When | Action |
|---|---|
| **Mid-Dec** | Ask for referrals; Google first (its process runs about 6–10 weeks) |
| Early Jan | Apply to Uber, Microsoft, Atlassian, others; space onsites 1–2 weeks apart |
| Every day | 2 problems (1 new mixed, 1 recall re-solve), 30 min |
| Every week | 1 mock (DSA or design), 1 HLD redo |
| 1 week before each loop | That company's tagged problems + past-round questions + its values |
| After each loop | Log every question asked + what went wrong in `Company Interview Logs/` |

---

## 9. Progress

| Week | DSA steps done | LLD | HLD | Recall due cleared | Notes |
|---|---|---|---|---|---|
| W1 | | | | | |
| W2 | | | | | |
| W3 | | | | | |
| W4 | | | | | |
| W5 | | | | | |
| W6 | | | | | |
| W7 | | | | | |
| W8 | | | | | |
| W9 | | | | | |
| W10 | | | | | |
| W11 | | | | | |
| W12 | | | | | |

> [!danger] If you fall behind
> - Cut *(opt)* items first, then easy re-solves
> - Never cut: graphs, DP, weekend design, recall
> - Two weeks behind by end of Nov → move mocks into Jan week 1, not later

**Mnemonic:** *Weekdays solve, weekends design, every day recall.*

## Sources
- [Hello Interview: Google L4 guide](https://www.hellointerview.com/guides/google/l4)
- [Jobrise: Google India L3/L4 prep](https://www.jobrise.io/en/blog/google-interview-preparation-india/)
- [PracHub: is Striver A2Z enough (474 problems, 18 sections)](https://prachub.com/resources/is-striver-s-a2z-dsa-sheet-enough-for-coding-interviews-in-2026)
- [Hello Interview: system design](https://www.hellointerview.com/learn/system-design/in-a-hurry/introduction)
- [Hello Interview: low-level design](https://www.hellointerview.com/learn/low-level-design/in-a-hurry/introduction)
- [TechPrep: Uber interview process](https://www.techprep.app/blog/uber-interview-process)
- [LeetCode: Microsoft India SDE2 process](https://leetcode.com/discuss/interview-experience/1780051/microsoft-complete-interview-process-india-sde2se2)
- [LeetCode: Atlassian P40 offer](https://leetcode.com/discuss/interview-experience/5757617/)
