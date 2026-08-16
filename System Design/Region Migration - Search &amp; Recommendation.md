# Region Migration — Search & Recommendation

![[region-migration-architecture.svg]]

- Owns full migration of search & recommendation services from Dublin (hit expansion limit) to a new Spain region, **including the full traffic-shift strategy**
- Core constraint: devices don't call the backend directly — a config service pushes new endpoints to devices, propagation takes ~6 hours, so device-side repoint can't be used for fast rollback

## Elevator pitch (memorize this)

> I own the migration of our search and recommendation services from a Dublin data center that hit its physical expansion limit, to a new Spain region — including the full traffic-shifting strategy. The core engineering constraint was that client devices can't be redirected quickly: config pushes to update device-side endpoints take 6+ hours to propagate, so a naive cutover meant that any problem in the new region could mean hours of degraded service with no fast way back. I designed a proxy indirection layer in front of both regions so rollback became a routing change at the proxy — seconds instead of hours — built the reconciliation logic to guarantee we never served stale data during the migration window, and designed the gradual, percentage-based traffic-shift and capacity model so we scaled the new region without overprovisioning.

> [!tip] Framing rule
> Every sentence is a design decision + reason, never a task log ("I migrated servers"). That's what separates this from reading as DevOps work.

## Four pillars — deep-dive framework

| Pillar | Pattern to name | What to have ready |
|---|---|---|
| Proxy fleet indirection | **Strangler Fig migration pattern** | Why device-config rollback was rejected (too slow, unbounded blast radius); tradeoff — one extra hop of latency for bounded, fast recovery |
| Data migration / staleness prevention | **CAP tradeoff / consistency model** | `[FILL IN: exact mechanism — dual-write + reconciliation? async replication + comparison job? freeze-and-snapshot? how staleness was verified — checksums, shadow reads, diffing?]` |
| Traffic-shift & capacity | **Progressive delivery / canary rollout** | `[FILL IN: how shift % and pace were decided; manual thresholds or autoscaling tied to real-time load?]` |
| Rollback & monitoring | **Fast-fail / automated recovery** | `[FILL IN: was rollback human-triggered or automatic on breached thresholds — error rate, latency, TPS? this is the strongest "not just DevOps" signal]` |

> [!warning] Interviewer follow-up prep
> For each pillar, expect "why not [the obvious alternative]?" — e.g. "why not a hard cutover?", "why not synchronous replication?" Naming the rejected alternative + why is what reads as engineering judgment.

## Open problems (name these if asked what's still hard)

- Handling high TPS during dual-region operation
- Right-sizing new-region instance counts as traffic shifts, avoiding overprovisioning cost
- Reducing added latency from the proxy hop
- Avoiding the proxy fleet itself becoming a single point of failure — redundant instances, multi-AZ, health-check failover

## Domain note

- TCS background is BFSI (claims/insurance) — Moody's Analytics is risk/credit-analytics focused, not claims/insurance, so the domain overlap is loose. Lead with engineering depth, not the domain match.
