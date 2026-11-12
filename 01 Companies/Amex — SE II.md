# Amex — SE II

| Date | Round reached | Result |
|---|---|---|
| 26 Oct 2026 | R3 (director) | **Rejected after R3** |

## Questions asked
- **R1 DSA:** Longest Palindromic Subsequence (LC 516)
- **R2:** Java, async, development end-to-end
- **R3:** director round

## How I tackled it
- R1: brute force only, optimal not recalled; cleared R1 anyway
- Cleared R2 and R3

## Outcome
- Failed: **rejected the same day as R3**; suspected layoffs, not performance
- Recruiter considering him for another role (as of 29 Oct)

> [!tip] LPS fix
> `dp[i][j]` = LPS in `s[i..j]`. Equal ends → `2 + dp[i+1][j-1]`; else `max(dp[i+1][j], dp[i][j-1])`. Fill **i from n-1 down**, j from i up. Or: LCS(s, reverse(s)).
