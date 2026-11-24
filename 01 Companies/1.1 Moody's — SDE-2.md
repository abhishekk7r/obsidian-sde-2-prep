# Moody's — SDE-2

| Date | Round reached | Result |
|---|---|---|
| 20 Aug 2026 | R1 (DSA) | **Rejected at R1** |

## Questions asked
- **Minimum Cost For Tickets** (LC 983, Medium DP)

## How I tackled it
- State definition + 3-branch shape (1-day / 7-day / 30-day pass) correct verbally
- Code wrong: used `dp[i-1]` for all three options
- Correct: 7-day and 30-day → jump back to the **last day outside the window**, not `i-1`

## Outcome
- Failed: **yes**, rejected at R1
- Prep plan de-weighted DSA ("trees-capped, no DP") based on 3 thin accounts
- Interval-covering DP never practiced

> [!danger] Lesson
> "Verbal approach right, code transition wrong." Never zero out a core skill on 2–3 secondhand accounts; company base rates > anecdotes.
