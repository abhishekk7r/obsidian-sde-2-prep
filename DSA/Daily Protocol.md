# Daily Protocol — 1 Hour with Claude

## Start
Say **"daily session"** — Claude reads [[Active Recall Tracker]] and runs everything.

---

## Session (60 min)

| Phase | Time | What happens |
|---|---|---|
| **Recall quiz** | 10 min | Claude picks 1-2 due problems → you explain approach cold (no notes). If you can't in 2 min, confidence stays ≤ 2 |
| **Solve** | 35 min | New problem or re-solve. Claude plays interviewer — hints only. Full re-implementation only for tricky problems; for known ones, explain approach + flag the gotcha |
| **Debrief** | 10 min | Claude checks [[Mistake Log]] for recurrence, updates tracker, shows tomorrow's queue |
| **Pattern flash** | 5 min | 2-3 random patterns: "when, template, gotcha?" — 30 sec each |

---

## Solve modes

| You know the problem well | You don't |
|---|---|
| Explain approach + key insight + gotcha in 2 min | Full solve from scratch, timed |
| Claude probes the tricky part only | Claude gives hints like an interviewer |
| Confidence 4-5 if clean | Scored normally |

---

## Commands

| Say | Claude does |
|---|---|
| `daily session` | Full 60-min session |
| `quick recall` | Recall quiz only (10 min) |
| `weekly review` | Tracker analysis + priority reset |
| `interview in [N] days` | Emergency sprint plan |
| `what's due today` | Show review queue |
