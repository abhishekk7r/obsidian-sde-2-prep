Pattern: DFS + Memoization on grid — every cell caches longest increasing path starting from it
dp[i][j] = 1 + max(dfs(all valid neighbors where neighbor value > current value))
DFS must return int, not void — parent cell needs child results to compute its own answer
Base case: if dp[i][j] != 0, return it immediately (memo hit — don't recompute)
At minimum dp[i][j] = 1 (the cell itself counts as a path of length 1)
🔑 No visited array needed — "strictly increasing" constraint prevents cycles naturally; you can never revisit a smaller cell from a larger one
🔑 Starting cell doesn't matter — memoization means even a "bad" starting cell's result gets cached and reused when a better path passes through it later
Outer loop: try every cell as a starting point, track global max across all calls
⚠️ Aggregation is max, NOT += — two neighbor paths of length 3 and 5 give 1 + 5 = 6, not 1 + 3 + 5 = 9; "longest" = max, "count" = +, never confuse these
⚠️ Global max tracking: ans = max(ans, dfs(...)) — first arg must be the accumulator, not dp[i][j]; using dp[i][j] loses the running best and only keeps the last cell's result
⚠️ Write the recurrence in one line BEFORE coding: dp[i][j] = 1 + max(dfs(neighbors)) — this forces return type, aggregation op, and base case decisions upfront

Complexity: O(nm) time (each cell computed once, memo prevents revisits), O(nm) space (dp grid + recursion stack worst case)