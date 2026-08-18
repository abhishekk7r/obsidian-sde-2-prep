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


---

## 4. Stereotype Annotations

- `@Component`, `@Service`, `@Repository` all register a bean identically — same scanning mechanism, functionally interchangeable for bean registration
- `@Service`/`@Component` differ only in semantic intent + AOP pointcut targeting

> [!tip] The one real behavioral difference
> `@Repository` triggers `PersistenceExceptionTranslationPostProcessor`, which catches platform-specific exceptions (JPA `PersistenceException`, raw `SQLException`, Hibernate exceptions) and translates them into Spring's unified unchecked `DataAccessException` hierarchy. Swap persistence tech later, calling code still just catches `DataAccessException`.

**Mnemonic:** *`@Repository` = scanning + exception translation. The others = scanning + a label.*

---

## 5. Auto-Configuration

![[autoconfiguration-conditional-flow.svg]]

- All auto-config classes listed in `META-INF/spring/...AutoConfiguration.imports`, loaded and gated by `@Conditional*` annotations
- `@ConditionalOnClass` — is the required library actually on the classpath
- `@ConditionalOnMissingBean` — has the user already defined this bean themselves (back off if so)
- `@ConditionalOnProperty` — is a specific `application.properties` key set
- `@ConditionalOnWebApplication` — servlet-based app vs plain

**Mnemonic:** *Auto-config = a pile of `@Bean` methods gated by "is the library here?" and "did you not already do this yourself?"*

---

## 6. REST Controller Annotations

- `@RestController` = `@Controller` + `@ResponseBody` — return values serialize straight to the response body instead of resolving a view name

| | `@PathVariable` | `@RequestParam` |
|---|---|---|
| Source | Part of the URL structure itself | Query string, tacked on |
| Omitted by client | No match → **404**, request never routes | `required=true` by default → **400**, never reaches method |
| Optional? | Not really — it's the route pattern | Only if `required=false` or `defaultValue(...)` set |

> [!danger] Trap
> Neither one just "becomes null" when missing. Both fail the request *before your method runs* unless explicitly declared optional.

**Wrapper vs primitive when actually optional (`required = false`):**

| Type | Value when omitted | Why |
|---|---|---|
| `Boolean` (wrapper) | `null` | Object reference, can hold null |
| `boolean` (primitive) | `false` (type's zero-value) | Primitives can't hold null — Spring falls back to default |

> [!danger] Trap
> `if (includeItems)` on a null `Boolean` → `NullPointerException` on auto-unboxing. Same check on primitive `boolean` can never NPE.

---

## 7. Exception Handling — `@ControllerAdvice`

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
            .map(f -> f.getField() + ": " + f.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return ResponseEntity.badRequest().body(new ErrorResponse(message));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        return ResponseEntity.status(500).body(new ErrorResponse("Unexpected error"));
    }
}
```

- `@RestControllerAdvice` = `@ControllerAdvice` + `@ResponseBody`, applies globally across every controller
- `@Valid @RequestBody` field validation failure → **`MethodArgumentNotValidException`** (not the same as a missing `@RequestParam`, which throws `MissingServletRequestParameterException`)

### Handler resolution rule
- Spring walks the thrown exception's class hierarchy and picks the **closest matching** `@ExceptionHandler` in the advice class — not declaration order
- An unlisted exception type (no exact/ancestor match anywhere in the advice class) **skips the advice entirely** — falls through to Spring Boot's default `BasicErrorController` → generic 500

> [!danger] Trap — narrow advice is a coverage gap, not a safety net
> A `@ControllerAdvice` with only specific handlers (`MethodArgumentNotValidException`, `EntityNotFoundException`, etc.) does **not** catch everything thrown in a controller. Anything not explicitly matched bypasses the class completely. Production code keeps a generic `@ExceptionHandler(Exception.class)` specifically as a deliberate catch-all.

**Mnemonic:** *Handlers are opt-in per exception type — unmatched types don't fall through your class, they fall through it entirely.*

---

## 8. Spring Data JPA

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

---

## 9. Application Properties / Profiles

- `application.properties` = base config, always loaded regardless of active profile
- `application-{profile}.properties` = overlay, loaded on top when that profile is active

### Merge rule — overlay by key, not full replacement

| Key exists in | Result |
|---|---|
| Both base and profile file | Profile-specific value wins |
| Base only | Base value still applies |
| Profile file only | That value is added on top |

**Mnemonic:** *Base file is the floor, profile file is the patch — only keys the patch actually mentions get overridden.*
