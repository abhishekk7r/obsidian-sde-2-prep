# Graph Patterns — BFS / DFS

## Pattern 1: Multi-source BFS (Level-order)
**When:** shortest distance/time from multiple starting points simultaneously
**Template:**
1. Push all sources into queue
2. `while queue not empty` → `size = q.size()` → process full level
3. Increment counter after each level
4. Stop on target condition or queue empty

**Problems (in practice order):**

| Problem | Twist | Status |
|---|---|---|
| [[Walls and Gates]] | Fill distance from all gates | ✅ |
| 01 Matrix (LC 542) | Distance of each cell to nearest 0 | ⬜ |
| [[Rotting Oranges]] | Time-based, return -1 if unreachable | ✅ |
| As Far from Land (LC 1162) | Last cell visited = answer | ⬜ |
| Map of Highest Peak (LC 1765) | Assign heights from water cells | ⬜ |
| [[Shortest Bridge]] | DFS to find + multi-source BFS to expand | ✅ |

**Evaluation checklist (for next solve):**
- [ ] Did you process per-level (`size = q.size()`) not per-cell?
- [ ] Did you mark cells visited **at push time**, not at pop time?
- [ ] Did you handle the "impossible" case (return -1)?

---

## Pattern 2: DFS + Memoization on Grid
**When:** longest/shortest path with overlapping subproblems, no cycles (or naturally acyclic via constraint like "strictly increasing")
**Template:**
1. `dp[i][j]` = answer for cell `(i, j)`
2. DFS returns `int`, not `void`
3. Base case: if `dp[i][j]` already set, return it
4. Recurse into valid neighbors, aggregate with `max` (or `min`)
5. Cache result, return it
6. Outer loop: try every cell, track global answer

**Problems (in practice order):**

| Problem | Twist | Status |
|---|---|---|
| [[Longest Increasing Path In A Matrix]] | Strictly increasing = natural cycle prevention | ✅ |

**Evaluation checklist (for next solve):**
- [ ] Is DFS returning a value, not void?
- [ ] Are you using `max` (not `+=`) to aggregate?
- [ ] Is `ans = max(ans, dfs(...))` — tracking global max correctly?
- [ ] Is the memo check at the top of DFS?

---

## Pattern 3: Multi-source Boundary DFS/BFS
**When:** mark reachable cells from boundary, then classify interior
**Template:**
1. Start DFS/BFS from all relevant boundary cells
2. Mark reachable cells
3. Anything unmarked is the answer

**Problems:**

| Problem | Twist | Status |
|---|---|---|
| [[Pacific Atlantic Water Flow]] | Two passes (Pacific + Atlantic), intersect results | ✅ |
| Surrounded Regions (LC 130) | Border O's are safe, flip the rest | ✅ |
