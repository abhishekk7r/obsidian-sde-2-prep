# Pattern Curriculum — Google L4 / Uber SDE2

> [!note] Source
> Built from 2025–2026 interview experiences at Google L4 and Uber SDE2 (LeetCode Discuss, Glassdoor, Roundz, HelloInterview). Patterns ordered by frequency × difficulty gap.

---

## Tier 1 — High frequency, confirmed in live interviews

### P01: Graphs — BFS / DFS / Multi-source
**Google:** ✅ confirmed (dependency graph, matrix arrangement)
**Uber:** ✅ confirmed (Number of Islands II, Bus Routes)
**Your status:** 6 problems solved, pattern notes exist, but live failures at Lenskart (topo sort)
**Problems (practice order):**

| # | Problem | LC# | Difficulty | Twist |
|---|---|---|---|---|
| 1 | Walls and Gates | 286 | M | Multi-source BFS baseline |
| 2 | Rotting Oranges | 994 | M | Multi-source BFS + impossible case |
| 3 | Number of Islands | 200 | M | Connected components DFS |
| 4 | Clone Graph | 133 | M | DFS + hashmap for visited |
| 5 | Pacific Atlantic Water Flow | 417 | M | Boundary DFS, intersect |
| 6 | Course Schedule | 207 | M | Cycle detection (Kahn's) |
| 7 | Course Schedule II | 210 | M | Topological sort (Kahn's) — **your Lenskart failure pattern** |
| 8 | Alien Dictionary | 269 | H | Topo sort from constraints — **asked at Uber 2026** |
| 9 | Shortest Bridge | 934 | M | DFS find + BFS expand |
| 10 | Word Ladder | 127 | H | BFS on implicit graph |
| 11 | Shortest Path in Binary Matrix | 1091 | M | BFS shortest path on grid |
| 12 | Longest Increasing Path in Matrix | 329 | H | DFS + memo on grid |

---

### P02: Dynamic Programming — 1D / 2D / Interval
**Google:** ✅ confirmed (hard DP in onsite, candidate failed for brute-force only)
**Uber:** ✅ confirmed (DP on trees, game theory DP, Optimal BST variation)
**Your status:** live failure at Moody's (interval DP), flagged as weak
**Problems (practice order):**

| # | Problem | LC# | Difficulty | Sub-pattern |
|---|---|---|---|---|
| 1 | Climbing Stairs | 70 | E | 1D baseline |
| 2 | House Robber | 198 | M | 1D with skip constraint |
| 3 | Coin Change | 322 | M | Unbounded knapsack |
| 4 | Longest Increasing Subsequence | 300 | M | 1D with binary search optimization |
| 5 | Minimum Cost For Tickets | 983 | M | Interval-covering DP — **your Moody's failure** |
| 6 | Unique Paths | 62 | M | 2D grid DP baseline |
| 7 | Longest Common Subsequence | 1143 | M | 2D classic |
| 8 | Edit Distance | 72 | M | 2D string DP |
| 9 | Partition Equal Subset Sum | 416 | M | 0/1 knapsack |
| 10 | Longest Palindromic Subsequence | 516 | M | Interval DP |
| 11 | Burst Balloons | 312 | H | Interval DP — range thinking |
| 12 | Stone Game | 877 | M | Game theory DP — **Uber pattern** |
| 13 | Optimal Strategy for a Game | — | M | Minimax DP — **asked at Uber SDE2** |
| 14 | Maximum Profit in Job Scheduling | 1235 | H | Interval scheduling + DP + binary search |

---

### P03: Sliding Window
**Google:** ✅ confirmed (anagram finding, frequency counting)
**Uber:** ✅ confirmed (frequent pattern)
**Your status:** 1 problem in vault, weak coverage
**Problems (practice order):**

| # | Problem | LC# | Difficulty | Twist |
|---|---|---|---|---|
| 1 | Max Consecutive Ones III | 1004 | M | Variable window with flip budget |
| 2 | Longest Substring Without Repeating | 3 | M | Variable window + hashset |
| 3 | Minimum Window Substring | 76 | H | Variable window + frequency map |
| 4 | Find All Anagrams in a String | 438 | M | Fixed window + frequency — **Google pattern** |
| 5 | Sliding Window Maximum | 239 | H | Monotonic deque |
| 6 | Longest Repeating Character Replacement | 424 | M | Window with max-frequency trick |
| 7 | Permutation in String | 567 | M | Fixed window frequency match |

---

### P04: Trees — DFS / BFS / BST
**Google:** ✅ confirmed (tree path problems)
**Uber:** ✅ confirmed (DP on trees — heavy)
**Your status:** 1 problem in vault (Family Hierarchy)
**Problems (practice order):**

| # | Problem | LC# | Difficulty | Twist |
|---|---|---|---|---|
| 1 | Invert Binary Tree | 226 | E | DFS baseline |
| 2 | Maximum Depth | 104 | E | DFS baseline |
| 3 | Binary Tree Level Order Traversal | 102 | M | BFS baseline |
| 4 | Validate BST | 98 | M | Inorder / range check |
| 5 | Lowest Common Ancestor | 236 | M | Postorder DFS |
| 6 | Binary Tree Right Side View | 199 | M | BFS last-in-level |
| 7 | Diameter of Binary Tree | 543 | E | Postorder + global max |
| 8 | Serialize and Deserialize | 297 | H | Preorder + queue |
| 9 | Path Sum III | 437 | M | Prefix sum on tree |
| 10 | House Robber III | 337 | M | DP on tree — **Uber pattern** |
| 11 | Binary Tree Maximum Path Sum | 124 | H | Postorder + split-vs-extend |

---

## Tier 2 — Medium frequency, pattern mastery required

### P05: Binary Search
| # | Problem | LC# | Difficulty | Twist |
|---|---|---|---|---|
| 1 | Binary Search | 704 | E | Template baseline |
| 2 | Search in Rotated Sorted Array | 33 | M | Modified BS |
| 3 | Find Minimum in Rotated Array | 153 | M | BS on condition |
| 4 | Koko Eating Bananas | 875 | M | BS on answer |
| 5 | Median of Two Sorted Arrays | 4 | H | BS on partition |
| 6 | Split Array Largest Sum | 410 | H | BS on answer + greedy check |

### P06: Stack — Monotonic
**Google:** ✅ confirmed (visibility / next-greater-element variant)
| # | Problem | LC# | Difficulty | Twist |
|---|---|---|---|---|
| 1 | Next Greater Element I | 496 | E | Monotonic stack baseline |
| 2 | Daily Temperatures | 739 | M | Next warmer day |
| 3 | Largest Rectangle in Histogram | 84 | H | Classic monotonic stack |
| 4 | Trapping Rain Water | 42 | H | Two-pointer or stack |
| 5 | Car Fleet | 853 | M | Sort + stack |

### P07: Two Pointers
**Google:** ✅ confirmed (sorting + two pointers, greedy matching)
| # | Problem | LC# | Difficulty | Twist |
|---|---|---|---|---|
| 1 | Two Sum II (sorted) | 167 | M | Classic |
| 2 | 3Sum | 15 | M | Sort + skip duplicates |
| 3 | Container With Most Water | 11 | M | Shrink from wider side |
| 4 | Trapping Rain Water | 42 | H | Left-right max tracking |

### P08: Heap / Priority Queue
**Uber:** ✅ confirmed (frequent)
| # | Problem | LC# | Difficulty | Twist |
|---|---|---|---|---|
| 1 | Kth Largest Element | 215 | M | Min-heap of size k |
| 2 | Top K Frequent Elements | 347 | M | Heap or bucket sort |
| 3 | Merge K Sorted Lists | 23 | H | Min-heap merge |
| 4 | Find Median from Data Stream | 295 | H | Two heaps |
| 5 | Task Scheduler | 621 | M | Greedy + heap |

### P09: Backtracking
| # | Problem | LC# | Difficulty | Twist |
|---|---|---|---|---|
| 1 | Subsets | 78 | M | Include/exclude template |
| 2 | Permutations | 46 | M | Swap or used-array |
| 3 | Combination Sum | 39 | M | Unbounded with start index |
| 4 | Word Search | 79 | M | Grid backtracking |
| 5 | N-Queens | 51 | H | Column + diagonal tracking |
| 6 | Palindrome Partitioning | 131 | M | Partition + isPalindrome |

---

## Tier 3 — Lower frequency, high-impact when they appear

### P10: Greedy
| # | Problem | LC# | Difficulty |
|---|---|---|---|
| 1 | Jump Game | 55 | M |
| 2 | Jump Game II | 45 | M |
| 3 | Gas Station | 134 | M |
| 4 | Interval Scheduling / Non-overlapping Intervals | 435 | M |

### P11: Trie
| # | Problem | LC# | Difficulty |
|---|---|---|---|
| 1 | Implement Trie | 208 | M |
| 2 | Word Search II | 212 | H |
| 3 | Design Add and Search Words | 211 | M |

### P12: Union-Find
| # | Problem | LC# | Difficulty |
|---|---|---|---|
| 1 | Number of Connected Components | 323 | M |
| 2 | Redundant Connection | 684 | M |
| 3 | Accounts Merge | 721 | M |
| 4 | Number of Islands II | 305 | H |

### P13: Intervals
| # | Problem | LC# | Difficulty |
|---|---|---|---|
| 1 | Merge Intervals | 56 | M |
| 2 | Insert Interval | 57 | M |
| 3 | Meeting Rooms II | 253 | M |

### P14: Advanced Graph (Dijkstra, Bellman-Ford)
| # | Problem | LC# | Difficulty |
|---|---|---|---|
| 1 | Network Delay Time | 743 | M |
| 2 | Cheapest Flights Within K Stops | 787 | M |
| 3 | Path with Maximum Probability | 1514 | M |

---

## Total: 14 patterns, ~95 problems
**Target:** internalize all 14 patterns + solve 60-70 of these problems deeply
**Timeline at 1 hr/day:** ~12-16 weeks to full coverage with spaced repetition baked in
