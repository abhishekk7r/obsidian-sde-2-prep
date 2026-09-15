# Mistake Log

> Quick-scan before interviews. Each row = one mistake. Read top-to-bottom in 2 min.

| # | Mistake | Root Cause | Fix | Where | Count |
|---|---|---|---|---|---|
| 1 | `+=` instead of `max` for path aggregation | Confused "count paths" with "longest path" | longest/shortest → always `max`/`min`, never `+=` | Longest Increasing Path | 1 |
| 2 | `void` DFS when return value needed | Gap between verbal reasoning and code | If explanation says "return up" → function must return | Longest Increasing Path | 1 |
| 3 | Wrong variable in `max` tracking | `max(dp[i][j], dfs(...))` instead of `max(ans, dfs(...))` | Global max → first arg is accumulator (`ans`), not local value | Longest Increasing Path | 1 |
| 4 | Forgot to mark visited at push time in BFS | DFS marks at entry; BFS must mark at push | **Push and mark together, always** | Shortest Bridge | 1 |
| 5 | Per-cell increment instead of per-level in BFS | Missing `size = q.size()` level loop | "Minimum time/steps" BFS → counter goes up once per full layer | Rotting Oranges | 1 |
| 6 | Nested loop `break` doesn't exit outer loop | C++ `break` exits one level only | Use a `found` flag for 2D grid search | Shortest Bridge | 1 |
