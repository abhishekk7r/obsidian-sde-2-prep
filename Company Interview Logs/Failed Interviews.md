# Failed Interviews — Post-mortem

> [!note] Purpose
> Re-read before every interview. Each entry: **round reached → exact question → why it failed → fix**. Last updated 2026-11-09.

Related: [[Interview Questions]] · [[Mistake Log]] · [[JPMC SWE II Analysis]]

___

## Summary

| Company / Role | When | Out at | Question that sank it | Failure type |
|---|---|---|---|---|
| Moody's SDE-2 | 20 Aug 2026 | R1 | **Minimum Cost For Tickets** (LC 983, DP) | DSA — DP transition wrong |
| Lenskart Backend Engineer | 22 Aug 2026 | R1 | **Dependency resolution / topological sort** (Kahn's) | DSA — incorrect live code |
| Angel One SDE-2 | early Sep 2026 | DSA round | not recorded | DSA |
| Oracle Health ASE2 (req 340965) | Sep 2026 | after all 4 rounds cleared | none — rejected during comp talk after IC3 ask | Not a skill failure |
| NatWest Java SE (B7) | Sep 2026 | after tech assessment | not recorded | Unknown |
| Amex SE II | 26 Oct 2026 | after R3 (director) | **Longest Palindromic Subsequence** (LC 516) in R1 — brute force only | DSA — couldn't recall optimal (cleared R1 anyway) |
| Visa | Aug 2026 | — | see Lenskart & Visa block in [[Interview Questions]] | "Not a developer" pushback on RFE project |

___

## Moody's SDE-2 — rejected at R1

- **Question:** Minimum Cost For Tickets (LC 983, Medium DP)
- **What happened**
	- State definition + 3-branch shape (1-day / 7-day / 30-day pass) correct verbally
	- Code wrong: used `dp[i-1]` for all three options
	- Correct: 7-day and 30-day → jump back to **last day outside the window**, not `i-1`
- **Why it failed**
	- Prep plan de-weighted DSA ("trees-capped, no DP") based on 3 thin accounts
	- Interval-covering DP never practiced
- **Fix:** never zero out a core skill on 2–3 secondhand accounts; company base rates > anecdotes

> [!danger] Pattern
> "Verbal approach right, code transition wrong." Same gap as Mistake Log #2 — reasoning and code drift apart.

___

## Lenskart Backend Engineer — rejected at R1 (walk-in drive)

- **Question:** dependency resolution → topological sort (Kahn's algorithm)
- **What happened:** coded an incorrect solution live
- **Why it failed**
	- Prep drilled hashing + sliding window; graphs not reached in time
	- Topo sort existed only as a template, never coded under time pressure
- **Fix:** indegree map + queue + count-check for cycle; code it cold until it's under 10 min

> [!warning] Trap
> Kahn's: push all indegree-0 nodes first; result size < n → cycle. Missing the cycle check is the usual bug.

___

## Angel One SDE-2 — rejected (DSA-only round)

- Prepped in a 2-day window
- Specific question not logged — **add it if you remember**

___

## Amex SE II — rejected after R3

- **R1 DSA:** Longest Palindromic Subsequence (LC 516) — brute force only, optimal not recalled
- Cleared R1 + R2 (Java, async, dev end-to-end) and R3 (director)
- Rejected same day as R3; suspected layoffs, not performance
- **Recruiter considering him for another role** (as of 29 Oct)

> [!tip] Fix for LPS
> `dp[i][j]` = LPS in `s[i..j]`. Equal ends → `2 + dp[i+1][j-1]`; else `max(dp[i+1][j], dp[i][j-1])`. Fill **i from n-1 down**, j from i up. Or: LCS(s, reverse(s)).

___

## Oracle Health ASE2 — rejected after offer stage

- Cleared all 4 rounds (vending machine LLD, tree DSA, HM scenarios, problem-solving coding)
- Final round feedback: first candidate in 2 weeks to find that solution
- Rejected ~1 week into comp talk, after asking for IC3
- **Not a skill gap** — level ask + expected layoffs
- Now re-engaging for Senior ASE (req 337914)

___

## Recurring "not a developer" pushback (Oracle, Visa)

- RFE (Dublin → Spain migration) heard as DevOps/migration work
- **Fix:** lead with code you wrote: proxy routing/fallback, data reconciliation, traffic-shift + capacity algorithm, rollback automation
- Map each to a pattern: Strangler Fig, CAP trade-off, canary, fail-fast

___

## Cross-interview patterns

| Pattern | Evidence | Fix |
|---|---|---|
| **R1 DSA is the gate** | Moody's, Lenskart, Angel One all lost at DSA | DSA daily, not "if time remains" |
| **DP + graph templates not retrievable** | LC 983, topo sort, LC 516 | Code cold, timed; "solved once" ≠ ready |
| **Verbal ≠ code** | Moody's transition bug, Mistake Log #2 | Write the recurrence/invariant as a comment before coding |
| **Language friction** | C++ daily, Java in interview | Know which language is allowed before the round |
| **Project framing** | Oracle, Visa pushback | Lead with code + patterns, not migration steps |
| **Design rounds go fine** | Oracle 4/4, Amex R2–R3, Kotak R1 | Keep, don't over-invest |

> [!tip] Mnemonic
> **Gate, Template, Comment, Language, Code-first** — DSA is the gate, templates cold, comment before code, confirm language, pitch the code not the migration.
