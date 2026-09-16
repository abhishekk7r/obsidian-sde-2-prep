LC 1642 — Furthest Building You Can Reach (Heap + Greedy)

🔑 Pattern: "Lazy greedy correction with max-heap" — make a default choice now, fix retroactively when you run out
🔑 Ladders are most valuable on the largest climbs → let the heap decide placement, not you upfront
🔑 Core loop: always spend bricks, push diff to max-heap. If bricks < 0 and ladders > 0, pop max, refund bricks, use ladder. Else stuck.
🔑 Push current diff BEFORE popping — handles case where current diff is the largest automatically
⚠️ Don't try to decide "bricks or ladder?" at encounter time — leads to a branching nightmare
⚠️ Don't sort all diffs upfront — you don't know how far you'll reach
⚠️ Return index i when stuck (bricks < 0, no ladders), return heights.size()-1 if loop completes
⚠️ Only process positive diffs (next building is taller)
Complexity: O(n log n) time, O(n) space
Similar: LC 871 Minimum Number of Refueling Stops (same skeleton)
