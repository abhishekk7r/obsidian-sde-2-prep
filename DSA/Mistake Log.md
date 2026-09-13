# Mistake Log

> [!note]
> Recurring mistakes to watch for. Claude will flag these during practice if they reappear.

## Recurring Bugs

### 1. `+=` instead of `max` for path aggregation
- **Where:** Longest Increasing Path in a Matrix
- **What happened:** summed all neighbor path lengths instead of taking the longest
- **Root cause:** confused "how many paths contribute" with "which path is longest"
- **Watch for:** any problem asking for **longest/shortest** path — aggregation is always `max`/`min`, never `+=`
- **Count:** 1

### 2. `void` DFS when return value is needed
- **Where:** Longest Increasing Path in a Matrix
- **What happened:** described "return while backtracking" correctly in words, then wrote `void dfs(...)`
- **Root cause:** gap between verbal reasoning and code translation
- **Watch for:** if your explanation mentions "return the result up," the function signature must return something
- **Count:** 1

### 3. Wrong variable in `max` tracking
- **Where:** Longest Increasing Path in a Matrix
- **What happened:** `ans = max(dp[i][j], dfs(...))` instead of `max(ans, dfs(...))`
- **Root cause:** careless variable reference — compared cell value against DFS result instead of running max against global best
- **Watch for:** global max tracking — first arg to `max` must be the accumulator (`ans`), not a local value
- **Count:** 1

### 4. Forgetting to mark visited at push time in BFS
- **Where:** Shortest Bridge
- **What happened:** pushed water cells into queue without marking them as `2` → duplicates, broken level count
- **Root cause:** in DFS you mark at entry; in BFS you must mark at push — different timing
- **Watch for:** every BFS — **push and mark together, always**
- **Count:** 1

### 5. Per-cell increment instead of per-level
- **Where:** Rotting Oranges (initial attempt)
- **What happened:** incremented `minutes` every time a single cell rotted a neighbor, not once per BFS wave
- **Root cause:** missing the `size = q.size()` level-processing loop
- **Watch for:** any "minimum time/steps" BFS — counter goes up once per full layer, not per cell
- **Count:** 1

### 6. Incorrect nested loop break
- **Where:** Shortest Bridge
- **What happened:** double `break` without a flag — broke unconditionally regardless of whether target was found
- **Root cause:** C++ `break` only exits one loop level
- **Watch for:** any time you search a 2D grid for a starting cell — use a `found` flag
- **Count:** 1
