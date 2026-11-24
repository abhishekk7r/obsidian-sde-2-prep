# IoC, DI & Beans

## IoC / DI & Bean Lifecycle

- **IoC** — object creation/wiring handed to the container
- **DI** — mechanism used: constructor / setter / field injection
- Prefer **constructor injection** — immutable `final` fields, fails fast at startup, circular deps visible immediately

![[bean-lifecycle-flow.svg]]

> [!danger] Trap
> Singleton is built **once**, at startup — steps 1-7 never repeat for the same bean.

**Mnemonic:** *Build → Wire → Aware → PostProcess-Before → Init → PostProcess-After → Ready → Destroy.*

---

## Bean Scopes

| Scope                 | Lifetime             | Full lifecycle incl. destroy?      |
| --------------------- | -------------------- | ---------------------------------- |
| `singleton` (default) | One per container    | Yes                                |
| `prototype`           | New every request    | **No** — `@PreDestroy` never fires |
| `request`             | One per HTTP request | Yes                                |
| `session`             | One per HTTP session | Yes                                |

![[bean-scope-injection-traps.svg]]

> [!tip] Q1 vs Q2
> Constructor injection (Q1) forces **eager** resolution → fails at startup. Field injection (Q2, in progress) doesn't force the same eager resolution — trace it through rather than assuming the identical failure mode.

**Mnemonic:** *Narrower scope into a singleton = wired once, frozen forever — unless a scoped proxy defers the fetch.*

---

## Stereotype Annotations

- `@Component`, `@Service`, `@Repository` all register a bean identically — same scanning mechanism, functionally interchangeable for bean registration
- `@Service`/`@Component` differ only in semantic intent + AOP pointcut targeting

> [!tip] The one real behavioral difference
> `@Repository` triggers `PersistenceExceptionTranslationPostProcessor`, which catches platform-specific exceptions (JPA `PersistenceException`, raw `SQLException`, Hibernate exceptions) and translates them into Spring's unified unchecked `DataAccessException` hierarchy. Swap persistence tech later, calling code still just catches `DataAccessException`.

**Mnemonic:** *`@Repository` = scanning + exception translation. The others = scanning + a label.*
