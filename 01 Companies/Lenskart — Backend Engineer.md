# Lenskart — Backend Engineer

| Date | Round reached | Result |
|---|---|---|
| 22 Aug 2026 (walk-in drive) | R1 | **Rejected at R1** |

## Questions asked
- **DSA:** dependency resolution → **topological sort** (Kahn's algorithm)

> [!note] Questions logged jointly with Visa
> These were written down as one "Lenskart & Visa" list, so the split between the two companies isn't recorded.

- **Java**
	- CompletableFuture: what it is, production issues you'd face
	- Java 21 features (heavy on this), records
	- Atomic classes and variables, ReentrantLock, synchronized, volatile vs atomic
	- ConcurrentHashMap, thread pool, ThreadLocal
	- JVM memory, garbage collection (tuning)
	- equals vs hashCode, interface vs abstract class
	- Spring Boot lifecycle, concurrency questions
- **Spring Boot**
	- `@Transactional`, `@Controller` vs `@RestController`, common annotations
	- Exception handling without try-catch (`@RestControllerAdvice`)
	- N+1 problem, IoC / DI
- **Production debugging**
	- [[Production Debugging — OOM after an old commit]]
	- How to trace and RCA: logging, metrics, recent deployment, transient errors
- **DSA:** Max flip to make domino numbers equal to target · Longest Consecutive Sequence
- **System design:** notification system · vending machine

## How I tackled it
- Coded an incorrect topological sort live
- Prep had drilled hashing + sliding window; graphs not reached in time
- Topo sort existed only as a template, never coded under time pressure

## Outcome
- Failed: **yes**, rejected at R1

> [!warning] Lesson
> Kahn's: indegree map + queue, push all indegree-0 nodes first; result size < n → cycle. Missing the cycle check is the usual bug. Code it cold until it's under 10 min.
