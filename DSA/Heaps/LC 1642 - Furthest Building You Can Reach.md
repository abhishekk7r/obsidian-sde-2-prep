LC 1642 — Furthest Building You Can Reach

🔑 Ladders cover any height, so they're best saved for the biggest climbs
🔑 But you don't know which climbs are "biggest" until you've seen them — so don't decide upfront

Algorithm:
1. Walk left to right. Skip if next building is same height or shorter.
2. For every climb, spend bricks and record the diff in a max-heap.
3. If bricks go negative, you overspent. Check if you have a ladder.
   - Yes → pop the biggest past brick spend from the heap, refund those bricks, use a ladder there instead. Now the ladder retroactively covers the most expensive climb you've seen.
   - No → you're stuck. Return current index.
4. If you finish the loop, return last index (heights.size() - 1).

Why always spend bricks first?
- Because you don't know if a climb is big or small relative to future climbs.
- By always spending bricks and recording it, you let the heap track your history.
- When you run out, you look back and undo your worst decision with a ladder.

⚠️ Push current diff into heap BEFORE popping — if current diff is the largest, it gets popped back, so the ladder covers the current climb instead. Handles both cases automatically.
⚠️ Don't try to choose bricks vs ladder at encounter time — you can't predict the future, and it creates messy branching.

Complexity: O(n log n) time, O(n) space
Similar: LC 871 Minimum Number of Refueling Stops
