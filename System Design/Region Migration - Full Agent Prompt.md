# Region Migration — Full Agent Prompt (single copy-paste)

Attach all internal documentation about the search & recommendation Dublin→Spain migration, then paste this in full.

```
You are helping me (an SDE2 candidate) prepare for a technical interview where I need to explain a distributed systems project I own end-to-end: migrating search & recommendation services from a Dublin data center to a new Spain region.

STEP 1 — Extract the real architecture from the attached documentation. Produce a structured summary with these exact sections, citing which document/section each answer comes from where possible. Do not fill gaps with generic distributed-systems knowledge — if something isn't in the docs, say so explicitly:

1. System components — every service/component involved (proxy fleet, config service, search index, recommendation service, data stores, etc.) and how they connect.
2. Data consistency & staleness prevention — exactly how data is migrated/replicated between Dublin and Spain during the migration window. Dual-write with reconciliation? Async replication with a comparison/diff job? Snapshot + delta sync? How is staleness detected/prevented?
3. Traffic-shift mechanism — how traffic gradually shifts from Dublin to Spain. Percentage-based/canary-style? What decides the pace? Manual or automated?
4. Capacity/autoscaling — how instance counts in Spain scale as traffic shifts. Manual thresholds, autoscaling policies, or predictive scaling? What signals trigger scale-out/scale-in?
5. Rollback mechanism — exactly how rollback works via the proxy fleet. Human-triggered or automatic on breached thresholds (error rate, latency, TPS)? Actual threshold values if documented.
6. Redundancy/failover — how the proxy fleet itself avoids being a single point of failure.
7. Anything ambiguous or undocumented — flag gaps explicitly rather than guessing.

STEP 2 — Using only what you extracted in Step 1, rewrite each of the six areas above into a first-person, interview-ready answer (3-5 sentences each), naming the actual mechanism, not a generic pattern name. These should sound like me explaining my own design decisions and the tradeoffs I made, not a description of the system.

STEP 3 — Act as a tough, experienced distributed-systems interviewer running a project deep-dive for an SDE2 role. Based only on what you extracted in Step 1, generate the 15 hardest, most likely follow-up questions — covering failure modes, consistency edge cases, "why not the obvious alternative" questions, scaling limits, cost/latency tradeoffs, and what happens when something breaks mid-migration. For each, draft a strong answer grounded only in the documented architecture — flag anywhere the docs don't give enough to answer confidently.

Output all three steps clearly labeled and separated.
```
