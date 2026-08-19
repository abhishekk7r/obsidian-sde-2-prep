---
tags: [design-patterns, index]
---

# Design Patterns — Index

Status tracker for the design-patterns revision set (Moody's SDE2). One file per category.

| Category | File | Patterns | Status |
|---|---|---|---|
| Creational | [[Creational]] | Singleton, Factory Method, Builder | ✅ |
| Structural | [[Structural]] | Adapter, Decorator, Facade, **Proxy** | ✅ |
| Behavioral | [[Behavioral]] | Strategy, Observer, Chain of Responsibility, Template Method | ✅ |
| Distributed | [[Distributed Patterns]] | **Saga**, Circuit Breaker, Retry, Bulkhead, CQRS, Event Sourcing, Repository, Pub-Sub, API Gateway, Service Discovery, Sidecar | ✅ |

## Project-mapped patterns (name these explicitly in the interview)

| My project element | Pattern | Note |
|---|---|---|
| OPF proxy fleet in front of old/new region | **Proxy** (GoF) + **Strangler Fig** (arch) | Access control + indirection during migration |
| Gradual traffic-shift / canary rollout | Progressive delivery / **Canary** | Not GoF — pattern-adjacent |
| SQS + DLQ | **Publish-Subscribe** + dead-letter idiom | Ties to Retry / Circuit-Breaker thinking |
| Rollback + monitoring chain | **Circuit Breaker** style fail-fast | "Detect bad state → revert traffic" |
| Cross-service consistency (no stale data) | **Saga** / **CAP** tradeoff | Compensating actions, not 2PC |

> [!tip] Interview framing
> Never recite a pattern abstractly. Every answer = **"the problem was X, so I used pattern Y, which works by Z"** — then tie to the region-migration project where it fits.
