# Active Recall Tracker

> [!note] How this works
> Claude reads this file at the start of every daily session to pick what you practice.
> **Confidence:** 1 = can't recall at all → 5 = can solve cold in interview time
> **Next review:** based on spaced repetition — if you nail it, interval doubles; if you fail, resets to 1 day
> **Intervals:** 1d → 3d → 7d → 14d → 30d → 60d (graduated)

---

## Spaced Repetition Rules

```
Confidence after review → Next interval:
  1 (blank / wrong approach) → review tomorrow
  2 (recalled pattern, fumbled execution) → review in 2 days
  3 (solved with minor hints) → review in 4 days
  4 (solved clean, minor hesitation) → current interval × 2
  5 (solved cold, interview-ready) → current interval × 2.5
  
  Max interval: 60 days
  If confidence drops below previous → reset to half the current interval
```

---

## Problem Log

> [!tip] Format
> Each row = one problem. Claude updates this after each session.
> `Interval` = current spacing in days. `Due` = next review date.

| Problem | LC# | Pattern | Solves | Last Solved | Confidence | Interval | Due | Notes |
|---|---|---|---|---|---|---|---|---|
| Walls and Gates | 286 | Graph BFS | 1+ | pre-tracker | 3 | 3 | 2026-09-16 | Existing in vault |
| Rotting Oranges | 994 | Graph BFS | 1+ | pre-tracker | 3 | 3 | 2026-09-16 | Existing in vault |
| Pacific Atlantic Water Flow | 417 | Graph DFS | 1+ | pre-tracker | 3 | 3 | 2026-09-16 | Existing in vault |
| Shortest Bridge | 934 | Graph BFS+DFS | 1+ | pre-tracker | 3 | 3 | 2026-09-16 | Existing in vault |
| Longest Increasing Path | 329 | Graph DFS+Memo | 1+ | pre-tracker | 3 | 3 | 2026-09-16 | Existing in vault |
| Max Consecutive Ones III | 1004 | Sliding Window | 1+ | pre-tracker | 3 | 3 | 2026-09-16 | Existing in vault |

---

## Session Log

> [!note] Format
> Claude appends a row here after every daily session.

| Date | Type | Problem | Result | Confidence | Mistakes | Time |
|---|---|---|---|---|---|---|
| — | — | — | — | — | — | — |

---

## Pattern Mastery Scorecard

> [!tip]
> Claude updates this periodically. A pattern is "interview-ready" when ≥3 problems in it are at confidence 4+.

| Pattern | Problems Tracked | Avg Confidence | Interview Ready? |
|---|---|---|---|
| Graph BFS/DFS | 5 | 3.0 | ❌ |
| DP (1D/2D/Interval) | 0 | — | ❌ |
| Sliding Window | 1 | 3.0 | ❌ |
| Trees | 0 | — | ❌ |
| Binary Search | 0 | — | ❌ |
| Monotonic Stack | 0 | — | ❌ |
| Two Pointers | 0 | — | ❌ |
| Heap | 0 | — | ❌ |
| Backtracking | 0 | — | ❌ |
| Greedy | 0 | — | ❌ |
| Trie | 0 | — | ❌ |
| Union-Find | 0 | — | ❌ |
| Intervals | 0 | — | ❌ |
| Advanced Graph | 0 | — | ❌ |
