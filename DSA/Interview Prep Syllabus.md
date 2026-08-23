# Interview Prep Syllabus

## Status
- Only active process: **Oracle SDE2/IC2**, interview Fri Aug 28 — DSA round + coding-based LLD round
- Moody's, Visa, Lenskart all ended — all three at the **DSA round**, none reached round 2
- Plan: pause further interviewing after the Oracle outcome (offer or rejection)

---
## Company-wise issues faced

| Company | Round reached | Question | Root cause |
|---|---|---|---|
| Moody's | R1 DSA | Minimum Cost For Tickets (LC 983, interval DP) | Used `arr[i-1]` for all three pass options; correct is `arr[last-index-outside-window]` for the 7-day/15-day passes — interval-covering pattern not internalized |
| Visa | R1 DSA | — | Rejected at DSA round, no further detail captured |
| Lenskart | R1 DSA | Dependency resolution / topological sort (Kahn's algorithm) | Live logic bug — built the edge in the wrong direction and decremented the wrong node's indegree; also multiple Java collection-API syntax errors (uninitialized collections, array vs `ArrayList`/`Map` syntax mixed up) |
| Oracle | Upcoming Aug 28 | DSA + coding-based LLD | Remaining live process |

> [!danger] The pattern
> All three rejections trace back to graph/DP-family topics that were already flagged as "template only, not battle-tested" in the DSA baseline review — before either was tested live.

---
## Weak areas
- **Graphs** — Dijkstra, DSU, Floyd-Warshall, BFS/DFS, topological sort — template-only; confirmed live failure (Lenskart)
- **Interval-covering DP** — the "ticket problem" shape — confirmed live failure (Moody's)
- **Backtracking** — template only, not yet tested live
- **Trie, KMP, Bellman-Ford** — template only, untested

> [!warning] Dormant ≠ retrievable
> The Notion Leetcode DB has ~55 solved problems (Apr–Dec 2025) including Graph (7) and real DP coverage, but they're months dormant. A past solve doesn't mean it's retrievable under interview pressure — treat dormant topics as untested until reactivated live.

---
## Strong areas
- LeetCode 1700+, **Top 5%**
- **A-Band (Top 10%)**, two consecutive years, TCS competitive rounds
- [[README|LLD / Design Patterns]] — Strategy, Factory, Observer, Singleton, Builder, Decorator, Proxy — theory sweep done, mapped to real project
- [[Region Migration|Region migration project narrative]] — 4 pillars ready for system-design defense:
  - Proxy fleet indirection → Strangler Fig
  - Data staleness prevention → CAP tradeoff
  - Traffic-shift & capacity → progressive delivery/canary
  - Rollback & monitoring → fast-fail/automated recovery

---
## Oracle syllabus (Aug 23–28)
Revised post-Lenskart: graphs elevated to their own dedicated block (was only folded into a mixed drill before), interval-DP bumped from Sat (that day got consumed entirely by the Lenskart interview).

| Day | Focus |
|---|---|
| Sun 23 | **Interval-DP** (bumped) + **Graphs** (BFS/DFS, topological sort, Union-Find) — dedicated, elevated priority |
| Mon 24 | Backtracking |
| Tue 25 | LLD day — LRU, LFU, URL Shortener |
| Wed 26 | Mixed DSA drill + remaining gap sweep |
| Thu 27 | Light revision (cold HLD pass likely cut for time) |
| Fri 28 | **Oracle interview** |

> [!tip] Mnemonic
> Graphs & Gaps first, LLD next, drill before you rest.
