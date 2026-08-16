# Region Migration — Interview Prep Prompt (single file)

Attach all internal documentation about the search & recommendation Dublin → Spain migration, then paste this in full into the agent.

```
You are helping me (an SDE2 candidate) prepare for a technical interview where I need to explain a distributed systems project I own end-to-end: migrating search & recommendation services from a Dublin data center (which hit its physical expansion limit) to a new Spain region, including full ownership of the traffic-shifting strategy.

Here is my current draft pitch and framework to use as a starting point — correct it wherever the real documentation contradicts it:

DRAFT PITCH:
"I own the migration of our search and recommendation services from a Dublin data center that hit its physical expansion limit, to a new Spain region — including the full traffic-shifting strategy. The core engineering constraint was that client devices can't be redirected quickly: config pushes to update device-side endpoints take 6+ hours to propagate, so a naive cutover meant that any problem in the new region could mean hours of degraded service with no fast way back. I designed a proxy indirection layer in front of both regions so rollback became a routing change at the proxy — seconds instead of hours — built the reconciliation logic to guarantee we never served stale data during the migration window, and designed the gradual, percentage-based traffic-shift and capacity model so we scaled the new region without overprovisioning."

DRAFT FOUR PILLARS (pattern name → what needs to be accurate):
1. Proxy fleet indirection → Strangler Fig migration pattern — why device-config rollback was rejected, tradeoff of one extra hop of latency for bounded fast recovery
2. Data migration / staleness prevention → CAP tradeoff / consistency model — exact mechanism unknown, needs real answer
3. Traffic-shift & capacity → Progressive delivery / canary rollout — exact shift-pace and scaling logic unknown, needs real answer
4. Rollback & monitoring → Fast-fail / automated recovery — exact trigger mechanism unknown, needs real answer

Do the following in order, using ONLY the attached documentation for factual details — flag anything the docs don't cover clearly rather than guessing:

STEP 1 — ARCHITECTURE EXTRACTION
Produce a structured technical summary with these sections, citing the source doc/section for each:
1. System components — every service/component involved (proxy fleet, config service, search index, recommendation service, data stores, etc.) and how they connect
2. Data consistency & staleness prevention — exact replication/reconciliation mechanism between Dublin and Spain, and how staleness is detected/prevented
3. Traffic-shift mechanism — how the shift percentage and pace are decided, manual or automated
4. Capacity/autoscaling — how instance counts scale as traffic shifts, what signals trigger scale-out/scale-in
5. Rollback mechanism — human-triggered or automatic on breached thresholds (error rate, latency, TPS), actual threshold values if documented
6. Redundancy/failover — how the proxy fleet avoids being a single point of failure
7. Gaps — anything ambiguous or undocumented, flagged explicitly

STEP 2 — CORRECTED NARRATIVE
Using Step 1's findings, rewrite the draft pitch and all four pillars to be factually accurate to the real architecture, first person, interview-ready. Flag and fix any place the draft used a pattern name or mechanism that doesn't match reality.

STEP 3 — SLIDE FRAMING
Turn the corrected narrative into a slide-ready structure: one slide's worth of bullets per pillar (title + 3-5 short bullets, no full sentences), plus one opening "problem statement" slide and one closing "results/impact" slide (flag if impact metrics aren't in the docs so I can get real numbers).

STEP 4 — HARD CROSS-QUESTIONS
Acting as a tough, experienced distributed-systems interviewer running a project deep-dive for an SDE2 role, generate the 15 hardest, most likely follow-up questions — covering failure modes, consistency edge cases, "why not the obvious alternative" questions, scaling limits, cost/latency tradeoffs, and what happens when something breaks mid-migration. Draft a strong answer for each grounded only in the documented architecture, flagging anywhere the docs don't give enough to answer confidently.

Output all four steps clearly labeled and separated.
```
