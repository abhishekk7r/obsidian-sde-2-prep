## 1. IoC / DI & Bean Lifecycle

- **IoC** — object creation/wiring handed to the container
- **DI** — mechanism used: constructor / setter / field injection
- Prefer **constructor injection** — immutable `final` fields, fails fast at startup, circular deps visible immediately

![[bean-lifecycle-flow.svg]]

> [!danger] Trap
> Singleton is built **once**, at startup — steps 1-7 never repeat for the same bean.

**Mnemonic:** *Build → Wire → Aware → PostProcess-Before → Init → PostProcess-After → Ready → Destroy.*

---

## 2. `@Transactional`

- Proxy-based AOP — JDK dynamic proxy (interface) or CGLIB subclass (no interface)
- Proxy is created at lifecycle step 6 (`postProcessAfterInitialization`)

![[transactional-proxy-self-invocation.svg]]

| Rollback rule | |
|---|---|
| Unchecked (`RuntimeException`, `Error`) | Rolls back by default |
| Checked exceptions | Does **not** roll back by default — use `rollbackFor` |

| Propagation | Behavior |
|---|---|
| `REQUIRED` (default) | Join existing, else create new |
| `REQUIRES_NEW` | Suspend existing, start independent one |
| `NESTED` | Savepoint within outer transaction |
| `SUPPORTS` | Join if exists, else run non-transactionally |
| `MANDATORY` | Must already be in a transaction — else throws |
| `NEVER` / `NOT_SUPPORTED` | Must not / suspends existing |

> [!warning] Production trap
> Long-running `@Transactional` methods hold a pooled DB connection for their full duration — wrap a slow external call inside one and you can exhaust the connection pool under load.

**Mnemonic:** *Proxy wraps the call — call yourself directly and the wrapper never sees it.*

---

## 3. Bean Scopes

| Scope | Lifetime | Full lifecycle incl. destroy? |
|---|---|---|
| `singleton` (default) | One per container | Yes |
| `prototype` | New every request | **No** — `@PreDestroy` never fires |
| `request` | One per HTTP request | Yes |
| `session` | One per HTTP session | Yes |

![[bean-scope-injection-traps.svg]]

> [!tip] Q1 vs Q2
> Constructor injection (Q1) forces **eager** resolution → fails at startup. Field injection (Q2, in progress) doesn't force the same eager resolution — trace it through rather than assuming the identical failure mode.

**Mnemonic:** *Narrower scope into a singleton = wired once, frozen forever — unless a scoped proxy defers the fetch.*
