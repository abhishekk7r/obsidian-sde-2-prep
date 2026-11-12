# Transactions & Spring Data JPA

## `@Transactional`

- Proxy-based AOP — JDK dynamic proxy (interface) or CGLIB subclass (no interface)
- Proxy is created at lifecycle step 6 (`postProcessAfterInitialization`)

![[transactional-proxy-self-invocation.svg]]

| Rollback rule | |
|---|---|
| Unchecked (`RuntimeException`, `Error`) | Rolls back by default |
| Checked exceptions | Does **not** roll back by default — use `rollbackFor` |

| Propagation               | Behavior                                       |
| ------------------------- | ---------------------------------------------- |
| `REQUIRED` (default)      | Join existing, else create new                 |
| `REQUIRES_NEW`            | Suspend existing, start independent one        |
| `NESTED`                  | Savepoint within outer transaction             |
| `SUPPORTS`                | Join if exists, else run non-transactionally   |
| `MANDATORY`               | Must already be in a transaction — else throws |
| `NEVER` / `NOT_SUPPORTED` | Must not / suspends existing                   |

> [!warning] Production trap
> Long-running `@Transactional` methods hold a pooled DB connection for their full duration — wrap a slow external call inside one and you can exhaust the connection pool under load.

**Mnemonic:** *Proxy wraps the call — call yourself directly and the wrapper never sees it.*

---

## Spring Data JPA

- `interface OrderRepository extends JpaRepository<Order, Long>` — CRUD with zero implementation, Spring generates a runtime proxy

### Derived query methods
- `findByCustomerEmailAndStatus(...)` — method name is parsed as a query: strip `findBy`/`deleteBy`/etc., split on `And`/`Or` respecting camelCase, match each token against entity fields (including nested traversal, e.g. `Order.customer.email`)
- Built into JPQL and validated **at `ApplicationContext` startup** (proxy creation), not on first call

### Validation timing — the pattern to remember

| Query type | Validated when |
|---|---|
| Derived method name | Startup (proxy creation) |
| `@Query("JPQL...")` | Startup — handed to `entityManager.createQuery()` during proxy build |
| `@Query(nativeQuery = true)` | **First execution only** — raw SQL isn't parsed against the JPA metamodel |

> [!tip] Recurring theme today
> Same fail-fast philosophy as eager constructor injection and classpath-gated auto-configuration — Spring prefers crashing loudly at boot over misbehaving at runtime, wherever it can.

**Mnemonic:** *JPQL — checked at boot. Native SQL — checked at first call. Typo a derived method name → app won't even start.*
