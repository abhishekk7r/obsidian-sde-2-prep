# JPMC SWE II — Question Analysis (R1 Thu 12 Nov 2026)

> [!note] Data
> - [snehasishroy/leetcode-companywise-interview-questions](https://github.com/snehasishroy/leetcode-companywise-interview-questions) `jpmorgan/` — 74 problems all-time, 14 in last 6 months; repo last updated **16 Aug 2026**
> - Frequency % is relative within JPMC, from a small sample — treat as ranking, not probability
> - Cross-checked with 6 SWE II write-ups (Feb–Sep 2026) and [[Failed Interviews]]

___

## Headline

- JPMC DSA = **arrays, strings, hashing, intervals, greedy + heap, sliding window**
- **Graphs: zero** in the 6-month list; trees: one, all-time only
- **DP: light** — but the one DP problem in the 6-month list is **LPS (LC 516), your Amex miss**
- **LRU Cache (LC 146)** is on the all-time list — your August LRU attempt had 5 open bugs
- Coding likely **Java-only** on HackerRank CodePair (one Bangalore offer account)

> [!danger] Overlap with your misses
> LC 516 and LC 146 are both on JPMC's list **and** both are things you've failed or left buggy. Do these two first.

___

## Pattern mix — last 6 months (14 problems)

| Pattern | Problems | Count |
|---|---|---|
| Intervals / sort + greedy | 253, 56, 1710, 2895 | 4 |
| Hashing / prefix sum | 49, 1, 560 | 3 |
| Sliding window / deque | 3, 239 | 2 |
| Heap + greedy | 2182 | 1 |
| Greedy / stack | 678 | 1 |
| String counting | 696 | 1 |
| Matrix | 48 | 1 |
| DP | 516 | 1 |
| Graph / tree | — | 0 |

___

## Tier 1 — must code in Java before Thursday

Recent (6-month list) or overlaps your misses. Sorted by priority.

| # | Problem | Pattern | Why it's here |
|---|---|---|---|
| 1 | [516 Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence) | Interval DP | 6-mo list + **Amex miss** |
| 2 | [146 LRU Cache](https://leetcode.com/problems/lru-cache) | HashMap + DLL | All-time 62.5% + **buggy in Aug** |
| 3 | [253 Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii) | Intervals + heap | 87.5%, also OA pattern |
| 4 | [49 Group Anagrams](https://leetcode.com/problems/group-anagrams) | Hashing | 100% all-time |
| 5 | [3 Longest Substring w/o Repeating](https://leetcode.com/problems/longest-substring-without-repeating-characters) | Sliding window | Asked Feb 2026 R1 |
| 6 | [560 Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k) | Prefix sum + map | 6-mo list |
| 7 | [2182 Construct String With Repeat Limit](https://leetcode.com/problems/construct-string-with-repeat-limit) | Heap + greedy | 100% in 6-mo |
| 8 | [2895 Minimum Processing Time](https://leetcode.com/problems/minimum-processing-time) | Sort + greedy | Most recent (30-day list) |
| 9 | [239 Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum) | Monotonic deque | Hard, 6-mo list |
| 10 | [56 Merge Intervals](https://leetcode.com/problems/merge-intervals) | Intervals | 6-mo list |
| 11 | [678 Valid Parenthesis String](https://leetcode.com/problems/valid-parenthesis-string) | Greedy range | 6-mo list |
| 12 | [48 Rotate Image](https://leetcode.com/problems/rotate-image) | Matrix | 87.5% all-time |

Quick ones (talk through, code only if shaky): [696 Count Binary Substrings](https://leetcode.com/problems/count-binary-substrings), [1 Two Sum](https://leetcode.com/problems/two-sum), [1710 Max Units on a Truck](https://leetcode.com/problems/maximum-units-on-a-truck)

___

## Tier 2 — if time remains

| Problem | Pattern |
|---|---|
| [347 Top K Frequent](https://leetcode.com/problems/top-k-frequent-elements) | Heap / bucket |
| [974 Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k) | Prefix mod + map (560 twin) |
| [42 Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water) | Two pointers |
| [2054 Two Best Non-Overlapping Events](https://leetcode.com/problems/two-best-non-overlapping-events) | Sort + heap / binary search |
| [1208 Get Equal Substrings Within Budget](https://leetcode.com/problems/get-equal-substrings-within-budget) | Sliding window |
| [5 Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring) | Expand around center |
| [295 Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream) | Two heaps |
| [1167 Min Cost to Connect Sticks](https://leetcode.com/problems/minimum-cost-to-connect-sticks) | Min-heap greedy |

___

## What to drop

- Dijkstra, topo sort, grid BFS, Maximal Square, Min Cost Tickets — **not on JPMC's list**; your misses elsewhere, not here
- New DP patterns — only LPS shows up

> [!warning] Re-plan
> The earlier prep doc listed LC 743, 207, 221, 983. Swap them for LC 516, 146, 560, 2182 from Tier 1.

___

## Java API you'll need for Tier 1

| Need | Java |
|---|---|
| Count chars | `int[] cnt = new int[26]` / `map.merge(c, 1, Integer::sum)` |
| Anagram key | `char[] a = s.toCharArray(); Arrays.sort(a); new String(a)` |
| Min-heap on end time | `new PriorityQueue<>((x, y) -> x[1] - y[1])` |
| Max-heap of chars | `new PriorityQueue<>((x, y) -> y - x)` |
| Sort intervals | `Arrays.sort(iv, (a, b) -> Integer.compare(a[0], b[0]))` |
| Monotonic deque | `Deque<Integer> dq = new ArrayDeque<>(); peekLast / pollLast / pollFirst` |
| Prefix map | `map.getOrDefault(sum - k, 0)`; seed `map.put(0, 1)` |
| List → array | `res.toArray(new int[0][])` |

> [!danger] Java traps
> - `x[1] - y[1]` comparator overflows on large values → `Integer.compare`
> - `==` on `Integer` > 127 fails → `.equals()`
> - `String` concat in a loop → `StringBuilder`

___

## Non-DSA mix in the same R1 (from write-ups)

- **PR / code review** of a broken Java class — compile errors, naming, encapsulation, equals/hashCode, concurrency
- Core Java: HashMap internals, equals/hashCode, CHM, streams (when not to use, parallel, findFirst vs findAny)
- Kafka + REST basics (your resume says 3 yrs Kafka)
- Opener: how you use Copilot / Claude Code

> [!tip] Mnemonic
> **LPS, LRU, Rooms, Anagrams** — the four JPMC problems that hit your weak spots or top the list. Code these four in Java first.
