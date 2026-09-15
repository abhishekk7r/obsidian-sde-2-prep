# Daily Protocol — 1 Hour DSA Session with Claude

> [!note] How to start
> Open Claude. Say: **"daily session"** or **"DSA practice"**
> Claude will read your [[Active Recall Tracker]] and run the session.

---

## Session Structure (60 minutes)

### Phase 1 — Active Recall Quiz (10 min)
**What Claude does:**
- Reads [[Active Recall Tracker]], finds problems that are **due today or overdue**
- Picks 1-2 due problems
- Shows you **only the problem statement** (no hints, no pattern name)
- You must articulate within 2 minutes:
  1. What pattern does this map to?
  2. What is the key insight?
  3. Walk through the approach step by step
  4. Time and space complexity
- Claude scores your recall (1-5) and updates the tracker

> [!warning] The rule
> If you can't articulate the approach in 2 minutes without looking at notes, it doesn't count as "known." Confidence stays ≤ 2.

### Phase 2 — Solve (35 min)
**Two modes (Claude picks based on your tracker):**

**Mode A — New Problem:**
- Claude picks the next unsolved problem from [[Pattern Curriculum]] based on priority:
  1. Tier 1 patterns with < 3 problems tracked
  2. Patterns where avg confidence < 3
  3. Next problem in sequence within a pattern
- Claude gives you the problem statement only
- You solve it (pseudocode, as per your preference)
- Claude simulates SDE2 interviewer — hints only, no answers unless you're stuck > 15 min
- After solving, Claude gives evaluation + adds to tracker

**Mode B — Re-solve (spaced repetition):**
- For problems with confidence 1-2 that keep failing recall
- Full re-solve from scratch, timed
- Must solve within interview time (20-25 min for medium, 30-35 for hard)

### Phase 3 — Debrief & Update (10 min)
- Claude identifies mistakes → checks against [[Mistake Log]] for recurrence
- Adds new mistakes to Mistake Log if pattern is new
- Updates [[Active Recall Tracker]] with today's results
- Tells you tomorrow's expected queue

### Phase 4 — Pattern Flash (5 min)
- Claude picks 2-3 random patterns from Tier 1
- For each: "When do you use [pattern]? What's the template? What's the gotcha?"
- You answer in 30 seconds each — this builds retrieval speed

---

## Weekly Review (Sunday, 15 min extra)

Say: **"weekly review"**

Claude will:
1. Read the full tracker and summarize:
   - How many problems are at confidence 4+?
   - Which patterns are still red?
   - What's your weakest area right now?
2. Adjust priorities for the coming week
3. Flag any problems that have been stuck at confidence 1-2 for > 2 weeks (candidate for re-learning)
4. Update the Pattern Mastery Scorecard

---

## Emergency Mode — Interview in < 7 days

Say: **"interview prep mode — [company] in [N] days"**

Claude will:
1. Read your tracker + the company's pattern profile from [[Pattern Curriculum]]
2. Build a day-by-day plan focusing on:
   - High-frequency patterns for that company
   - Your weakest areas within those patterns
   - Re-solve of previously failed interview problems
3. Switch to 2-problem sessions (review + new) instead of quiz + solve
4. Add company-specific problems if available

---

## Key Principles

> [!danger] Non-negotiable
> 1. **No reading notes during recall.** The whole point is testing retrieval, not recognition.
> 2. **Pseudocode, not runnable code.** Speed of thought > syntax.
> 3. **If you can't explain it in 2 min, you don't know it.** Be honest with yourself.
> 4. **Consistency > intensity.** 1 hour daily for 12 weeks beats 5 hours on weekends.

> [!tip] When to use Excalidraw
> For any problem involving:
> - Graph traversal (draw the grid/graph state changes)
> - DP table filling (draw the table, trace the recurrence)
> - Tree traversal (draw the tree, trace DFS/BFS order)
> - Backtracking (draw the decision tree)
> Claude will tell you when to switch to your Deco pen.

---

## Command Reference

| You say | Claude does |
|---|---|
| `daily session` | Full 60-min protocol |
| `quick recall` | Phase 1 only (10 min) |
| `solve [problem name]` | Jump to Phase 2 with a specific problem |
| `weekly review` | Weekly tracker analysis |
| `interview prep mode — [company] in [N] days` | Emergency sprint plan |
| `update tracker` | Manual tracker update (if session was offline) |
| `what's due today` | Shows today's review queue without starting session |
| `add problem [name] [LC#] [pattern]` | Manually add a problem to tracker |
