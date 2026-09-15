# Interview Prep Syllabus

## Current Target
- **Google L4** Software Engineer
- **Uber SDE2** (L4)
- **Timeline:** 12–16 weeks of daily practice (1 hr/day before office)

---

## System Architecture

| Component | Tool | Purpose |
|---|---|---|
| Pattern curriculum | [[Pattern Curriculum]] (Obsidian) | 14 patterns, ~95 problems, ordered by company relevance |
| Spaced repetition | [[Active Recall Tracker]] (Obsidian) | Tracks every problem: confidence, interval, due date |
| Session protocol | [[Daily Protocol]] (Obsidian) | How each 1-hr session works with Claude |
| Problem notes | Google Sheets | Solution writeups (single cell per problem) |
| Revision notes | Obsidian (`DSA/` subfolders) | Pattern templates, detailed approach notes |
| Diagrams | Excalidraw + Deco pen | DP tables, graph traversals, tree traces |
| Mistake tracking | [[Mistake Log]] (Obsidian) | Recurring bugs Claude flags during practice |
| Daily driver | Claude Pro | Picks problems, runs recall quiz, simulates interviewer, updates tracker |

---

## Historical Interview Results

| Company | Round reached | Question | Root cause |
|---|---|---|---|
| Moody's | R1 DSA | Minimum Cost For Tickets (LC 983, interval DP) | Interval-covering pattern not internalized |
| Visa | R1 DSA | — | Rejected at DSA round |
| Lenskart | R1 DSA | Dependency resolution (topological sort) | Wrong edge direction + collection API syntax errors |
| Oracle | R4 Final (passed) | Family tree ancestor traversal | Clean code + modularity focus |
| Kotak | Bar Raiser | Longest Common Prefix + Shipment LLD + distributed systems | Breadth round — LLD execution not smooth |

> [!danger] The pattern
> 3 of 4 DSA rejections trace to **graph/DP-family topics** that were flagged but never battle-tested under pressure.

---

## Phase Plan

### Phase 1 — Foundation (Weeks 1–4)
**Focus:** Tier 1 patterns — Graphs, DP, Sliding Window, Trees
**Goal:** 3+ problems per pattern at confidence 3+
**Daily mix:** 1 recall quiz (review) + 1 new problem

### Phase 2 — Breadth (Weeks 5–8)
**Focus:** Tier 2 patterns — Binary Search, Stack, Two Pointers, Heap, Backtracking
**Goal:** 2+ problems per pattern at confidence 3+
**Daily mix:** 1-2 recall quizzes (growing review queue) + 1 new problem

### Phase 3 — Depth + Hardening (Weeks 9–12)
**Focus:** Hard problems across all patterns + Tier 3 patterns
**Goal:** All Tier 1 patterns at "interview ready" (≥3 problems at confidence 4+)
**Daily mix:** 2 recall quizzes + 1 hard problem or re-solve

### Phase 4 — Mock + Maintenance (Weeks 13–16)
**Focus:** Full mock interviews (45 min, 2 problems, timed)
**Goal:** Consistent clean solves under pressure
**Daily mix:** Mock interview or recall-heavy review session

---

## Strong Areas (leverage, don't re-study)
- LeetCode 1700+, Top 5%
- A-Band (Top 10%), two consecutive years, TCS competitive rounds
- LLD / Design Patterns — Strategy, Factory, Observer, Singleton, Builder, Decorator, Proxy
- Region migration project narrative — 4 pillars for system design defense
- Java collections internals (strong, documented in [[Collections]])

---

## References
- [[Pattern Curriculum]] — the full problem set
- [[Active Recall Tracker]] — spaced repetition state
- [[Daily Protocol]] — how to run a session
- [[Mistake Log]] — recurring bugs
- [[Patterns — Graph BFS-DFS]] — existing graph pattern notes
