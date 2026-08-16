# Region Migration — Agent Prompts

Run in sequence: Prompt 1 first (with docs attached), then paste its output into Prompts 2 and 3.

## Prompt 1 — Architecture extraction

```
You are helping me (an SDE2 candidate) prepare for a technical interview where I need to explain a distributed systems project I own end-to-end: migrating search & recommendation services from a Dublin data center to a new Spain region.

Read all the documentation I've provided and produce a structured technical summary with these exact sections:

1. System components — every service/component involved (proxy fleet, config service, search index, recommendation service, data stores, etc.) and how they connect.
2. Data consistency & staleness prevention — exactly how data is migrated/replicated between Dublin and Spain during the migration window. Dual-write with reconciliation? Async replication with a comparison/diff job? Snapshot + delta sync? How is staleness detected/prevented?
3. Traffic-shift mechanism — how traffic gradually shifts from Dublin to Spain. Percentage-based/canary-style? What decides the pace? Manual or automated?
4. Capacity/autoscaling — how instance counts in Spain scale as traffic shifts. Manual thresholds, autoscaling policies, or predictive scaling? What signals trigger scale-out/scale-in?
5. Rollback mechanism — exactly how rollback works via the proxy fleet. Human-triggered or automatic on breached thresholds (error rate, latency, TPS)? Actual threshold values if documented.
6. Redundancy/failover — how the proxy fleet itself avoids being a single point of failure.
7. Anything ambiguous or undocumented — flag gaps explicitly rather than guessing.

Cite which document/section each answer comes from where possible. Do not fill gaps with generic distributed-systems knowledge — if it's not in the docs, say so.
```

## Prompt 2 — Narrative refinement

```
I'm prepping a technical interview narrative for a project I own: migrating search & recommendation services from Dublin to Spain. Here is my current draft pitch and "four pillars":

[PASTE your elevator pitch + four-pillar table]

Here is the actual documented architecture:

[PASTE Prompt 1's output]

Cross-check my draft against the real architecture:
1. Flag anywhere my draft names a pattern or mechanism that doesn't match what's actually documented — correct it to reflect the real implementation.
2. Rewrite each pillar's "what to have ready" into a first-person, interview-ready answer (3-5 sentences), naming the actual mechanism, not a generic pattern name.
3. Keep the same structure/tone as my draft — don't change anything that's already factually correct.
```

## Prompt 3 — Hard cross-questions

```
Given this documented architecture for a search & recommendation service migration from Dublin to Spain:

[PASTE Prompt 1's output]

Act as a tough, experienced distributed-systems interviewer running a project deep-dive for an SDE2 role. Generate the 15 hardest, most likely follow-up questions covering: failure modes, consistency edge cases, "why not the obvious alternative" questions, scaling limits, cost/latency tradeoffs, and what happens when something breaks mid-migration. For each, draft a strong answer grounded only in the documented architecture — flag anywhere the docs don't give enough to answer confidently.
```
