
## 1. IoC / DI & Bean Lifecycle

- **IoC (Inversion of Control)** — object creation/wiring handed to the container instead of the class doing `new` itself
- **DI** — the mechanism IoC uses: container injects dependencies (constructor / setter / field)

> [!tip] Constructor injection is preferred
> Enables immutability (`final` fields), fails fast at startup if a dependency is missing, makes circular dependencies visible immediately instead of at runtime.

### Bean lifecycle — 8 phases

| #   | Phase                                                 | Detail                                                                                                                                |
| --- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Instantiation                                         | Constructor called                                                                                                                    |
| 2   | Populate properties                                   | Dependencies injected (setter/field)                                                                                                  |
| 3   | Aware callbacks                                       | `BeanNameAware`, `BeanFactoryAware`, `ApplicationContextAware` etc. — bean gets container-level info if it implements these           |
| 4   | `BeanPostProcessor.postProcessBeforeInitialization()` | Runs for **every** bean, before init callbacks                                                                                        |
| 5   | Init callbacks                                        | `@PostConstruct` → `InitializingBean.afterPropertiesSet()` → custom `init-method`, in that order                                      |
| 6   | `BeanPostProcessor.postProcessAfterInitialization()`  | AOP proxies (e.g. `@Transactional` proxy) typically created **here** — this is why the object your code holds may not be the raw bean |
| 7   | Bean ready                                            | Available for use via the container                                                                                                   |
| 8   | Destruction                                           | `@PreDestroy` → `DisposableBean.destroy()` → custom `destroy-method`, on container shutdown                                           |

> [!danger] Trap — singleton-lifetime misconception
> A singleton bean is constructed **once** at container startup (unless lazy) and lives for the container's entire lifetime — it does not get recreated per request/use. Corrected misconception: assuming the container "re-touches" a singleton on each injection point re-triggers some part of the lifecycle — it doesn't; steps 1-7 happen exactly once.

**Mnemonic:** *Build → Wire → Aware → PostProcess-Before → Init → PostProcess-After → Ready → Destroy.*

---

## 2. `@Transactional`

- Implemented via a **proxy** (JDK dynamic proxy if the bean implements an interface, CGLIB subclass proxy otherwise) — the proxy wraps the real method call with begin/commit/rollback logic
- The proxy is created in the bean lifecycle's `postProcessAfterInitialization` step (see above)

> [!danger] Self-invocation trap
> Calling a `@Transactional` method **from another method in the same class** (`this.someTransactionalMethod()`) bypasses the proxy entirely — the call goes directly to the raw object, so no transaction is started. Classic interview gotcha. Fix: inject a self-reference bean, or move the transactional method to a separate collaborator bean, or use AspectJ compile-time weaving instead of proxy-based AOP.

### Rollback rules
- Default: rolls back on **unchecked exceptions** (`RuntimeException` and its subclasses) and `Error`
- Does **NOT** roll back on checked exceptions by default
- Override with `@Transactional(rollbackFor = SomeCheckedException.class)` or `noRollbackFor = ...`

### Propagation

| Propagation | Behavior |
|---|---|
| `REQUIRED` (default) | Join existing transaction if present, else create new |
| `REQUIRES_NEW` | Always suspend any existing transaction, start a fresh independent one |
| `NESTED` | Runs within a savepoint of the outer transaction — inner rollback doesn't necessarily kill the outer |
| `SUPPORTS` | Join if a transaction exists, else run non-transactionally |
| `MANDATORY` | Must run inside an existing transaction — throws if none |
| `NEVER` / `NOT_SUPPORTED` | Must **not** run in a transaction / suspends any existing one |

> [!warning] Production connection-pool gotcha
> A long-running or accidentally-nested `@Transactional` method holds a DB connection **checked out from the pool** for its entire duration. Under load, transactions held too long (e.g. wrapping slow external calls inside a `@Transactional` method) can exhaust the connection pool and cascade into unrelated request failures — not just a correctness issue, a capacity issue.

**Mnemonic:** *Proxy wraps the call — call yourself directly and the wrapper never sees it.*

---

## 3. Bean Scopes

| Scope | Lifetime | Container manages full lifecycle (incl. destroy)? |
|---|---|---|
| `singleton` (default) | One instance per container | Yes |
| `prototype` | New instance every time it's requested | **No** — container hands it off and forgets; `@PreDestroy`/`DisposableBean` callbacks are **never invoked** |
| `request` | One instance per HTTP request | Yes, scoped to request |
| `session` | One instance per HTTP session | Yes, scoped to session |

> [!danger] Trap — prototype's missing destroy callback
> Since the container doesn't track prototype instances after handing them out, any cleanup logic in `@PreDestroy` is silently skipped. If a prototype bean holds a resource that needs explicit release, the caller is responsible for it manually — the container will not help.

### Edge-case battle-test — injecting a narrower-scoped bean into a singleton

**Q5 — prototype bean, field injection, autowired into a singleton.**
Does the singleton get a fresh prototype instance every time it uses the field?
**Answer:** No. Field injection happens once, at the singleton's own construction/wiring time (lifecycle phase 2) — the container resolves and injects exactly one prototype instance then, and the singleton holds that same instance for its entire lifetime. There's no re-fetch from the container on each use; scope mismatch alone doesn't create per-use freshness. (Fix, if per-use freshness is actually needed: `ObjectFactory<T>`/`Provider<T>`, method injection, or a scoped proxy.)

**Q1 — request-scoped bean, constructor injection into a singleton controller, no scoped proxy.**
**Answer:** Fails **eagerly at application startup**, not at request time. Constructor injection forces the container to resolve the dependency immediately when constructing the singleton — but no HTTP request context exists yet at startup, so there's no request-scoped instance to hand over. Fails with `Scope 'request' is not active for the current thread`. Fix: `@Scope(proxyMode = ScopedProxyMode.TARGET_CLASS)` on the request-scoped bean — injects a proxy that resolves the real instance lazily, per-request, the first time a method is actually called on it.

> [!tip] Why field injection (Q2, in progress) differs from constructor injection (Q1) here
> Field injection doesn't force *eager* resolution the same way constructor injection does — worth tracing through carefully rather than assuming the same failure mode as Q1.

**Mnemonic:** *Narrower scope into a singleton = wired once, frozen forever — unless a scoped proxy defers the fetch.*
