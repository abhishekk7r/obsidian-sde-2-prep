LC 1642 — Furthest Building You Can Reach
🔑 For every climb, always spend bricks and push diff into max-heap. If bricks go negative and ladders remain, pop max from heap, refund those bricks, use a ladder there instead — ladder automatically lands on the biggest climb seen so far.
🔑 Push current diff BEFORE popping — if current is the largest, it gets popped back, so ladder covers current climb. Handles both cases.
⚠️ Don't decide bricks vs ladder at encounter time — you can't predict future climbs, and it creates branching hell.
O(n log n) time | Similar: LC 871 Minimum Refueling Stops
