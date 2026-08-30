# LLD — Common Learnings

> Cross-question this file before/during any new LLD problem. If a mistake below repeats, that's the signal — not a one-off.

---

## Things I did well
- Defend design decisions with reasoning, not guesses (e.g. rate locked at entry, one-directional refs)
- Self-catch gaps mid-flow when actively questioning my own draft
- Fix scope contradictions fast once flagged, no hedging
- Iterate on a broken draft instead of settling

## Concepts that are missing
- **Entity filter**: holds state / enforces rule → entity; else → field on something else
- **Orchestrator identification**: one class drives the workflow; others are subordinates with narrow jobs
- **Pattern justification**: a pattern needs a real varying-algorithm problem (Strategy ≠ a value lookup); don't force Singleton/Strategy/etc. onto plain cardinality or config
- **Interfaces = behavior contracts only** — no stored fields on the interface itself
- **Stateless service classes**: gate/manager-style classes shouldn't hold a specific request's data (ticket/vehicle/spot) as instance fields
- **Directionality of references**: default one-directional; only go bidirectional if there's an actual reverse-lookup use case
- **Return types that carry real objects**, not booleans, when the caller needs the object (e.g. `assignSpot` should return the `ParkingSpot`, not `bool`)
- **Enums over raw strings** for closed sets (vehicle type, status)

## Problems in my understanding
- **Requirement drift during class design** — a requirement gets locked in Section 1, then silently unmet two class-design rounds later (e.g. `Ticket` missing `entryTime`/`hourlyRate` despite requirement 2 stating it explicitly). Need to recheck each class against the requirements list, not just "does this look reasonable."
- **Locked decisions not carried forward** — agreeing a class shouldn't own something (e.g. `ParkingSpotManager` shouldn't know about rates), then giving it that field anyway in the next draft. Treat every prior lock as a constraint to check new code against.
- **Claiming a fix before verifying it landed** — said "fixed" while the file still showed the old, unfixed version. Verify before asserting.
- **Assumptions stated only when asked, not upfront** — e.g. single-threaded/single-gate should be said proactively, not pulled out via a follow-up question.
- **Shifting from independent reasoning to asking for the fix directly** under difficulty — fine for first-pass learning, but this is what "interviewer-assisted" looks like on a scorecard; the goal is closing gaps ("missing method," "wrong return type," "field violates a locked separation") without the corrected code being handed over.

---

## Log
- 2026-08-30 — Parking Lot (Requirements → Class Design)
