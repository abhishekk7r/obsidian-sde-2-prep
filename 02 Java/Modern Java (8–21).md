# Modern Java (8–21)

## Streams API — map / filter / reduce / collect

- Stream = pipeline over a source, **not** a data structure — lazy, single-use
- **Intermediate ops** (`map`, `filter`, `sorted`, `distinct`, `flatMap`) → return a Stream, don't execute anything
- **Terminal ops** (`collect`, `forEach`, `reduce`, `count`, `findFirst`) → trigger execution, consume the stream

| Stateless (map, filter) | Stateful (sorted, distinct) |
|---|---|
| Processes + emits each element immediately | Must buffer **entire** stream before emitting anything |
| `filter().findFirst()` can short-circuit | `sorted().findFirst()` still processes everything |

**map vs flatMap**
```java
// map: List<List<String>> -> Stream<Stream<String>>   (wrong shape, still nested)
nested.stream().map(List::stream);
// flatMap: List<List<String>> -> Stream<String>        (flattened)
nested.stream().flatMap(List::stream).collect(Collectors.toList());
```

**reduce() — 2-arg vs 3-arg**
```java
int sum = list.stream().reduce(0, Integer::sum);                    // 2-arg: identity, accumulator
int len = list.parallelStream().reduce(0,
    (acc, s) -> acc + s.length(),   // accumulator
    (a, b) -> a + b);               // combiner — merges partial results from different threads
```
- 3-arg needed when: parallel stream (combiner merges thread-local partials), or accumulator's output type ≠ input type

**Primitive streams** — `IntStream`/`LongStream`/`DoubleStream` avoid autoboxing, add `sum()`/`average()`/`summaryStatistics()`
```java
int total = nested.stream().flatMap(List::stream).mapToInt(Integer::intValue).sum();
```

**Common Collectors**

| Collector | Does |
|---|---|
| `toList()` / `toSet()` / `toMap()` | Standard collections |
| `joining(delim)` | Concatenate strings |
| `groupingBy(classifier)` | `Map<K, List<T>>` — like SQL GROUP BY |
| `groupingBy(classifier, downstream)` | e.g. `groupingBy(Employee::getDept, averagingDouble(Employee::getSalary))` |
| `partitioningBy(predicate)` | Always `Map<Boolean, List<T>>` |
| `counting()` | Group sizes, usually downstream of groupingBy |

> [!warning] Trap — stream reuse
> A stream is a single-use wrapper over a spliterator, not a container. Once a terminal op runs, the spliterator is exhausted — calling another op throws `IllegalStateException`. Same reason `sorted()` can't emit early: it has no "rewind," only forward traversal.

> [!danger] Trap — reduce() needs associativity
> ```java
> list.stream().reduce(0, (a, b) -> a - b); // fine sequential, WRONG if parallelized
> ```
> Non-associative accumulator/combiner (e.g. subtraction) gives different results depending on how the stream is split — parallel streams may group/combine in any order.

> [!warning] Trap — parallelStream() with shared mutable state
> ```java
> List<String> results = new ArrayList<>();
> list.parallelStream().forEach(results::add); // race condition — not thread-safe
> ```
> Lambdas passed to a parallel stream must be stateless, non-interfering, and (for reduction) associative.

> [!tip] Mnemonic
> Stateless = pass-through. Stateful = buffer-then-burst. Reduce needs order not to matter.

---

## var — Local Type Inference (Java 10+)

```java
var list = new ArrayList<String>(); // inferred as ArrayList<String>
```

- **Statically typed at compile time** — not dynamic typing, just less typing
- Only for **local variables with an initializer** — not fields, method params, return types, or bare `var x;`
- Can't infer from `null` alone (`var x = null;` → compile error)

---

## Switch Expressions & Pattern Matching (Java 14 / 21)

**Switch expression** (Java 14) — returns a value, arrow syntax, exhaustive:
```java
int numLetters = switch (day) {
    case MON, FRI, SUN -> 6;
    case TUE -> 3;
    default -> {
        yield 0;
    }
};
```

**Pattern matching for switch** (Java 21) — matches on type, deconstructs records directly:
```java
String describe(Payment p) {
    return switch (p) {
        case CardPayment c -> "Card: " + c.cardNumber();
        case UpiPayment u  -> "UPI: " + u.upiId();
    };
}
```

> [!tip] Why this pairs with sealed interfaces
> Because `Payment` is `sealed` with only `CardPayment`/`UpiPayment` permitted, the compiler knows the switch is **exhaustive** — no `default` needed. Add a new permitted type later → compiler forces you to handle it here. This combo (sealed + records + pattern-matching switch) is the modern Java idiom for closed variant modeling.

---

## Records

```java
record User(String name, int age) {}
```

Auto-generated: `private final` fields, canonical constructor, accessors (`name()`, not `getName()`), `equals()`/`hashCode()`/`toString()` — all based on component values.

|Property|Rule|
|---|---|
|Class modifiers|Implicitly `final` — cannot be extended, cannot extend another class|
|Interfaces|Can implement any number|
|Fields|Cannot add extra instance fields|
|Generics|Allowed — `record Pair<T,U>(T a, U b) {}`|
|Static members/methods|Allowed|

**Compact constructor** (validation/normalization, compiler still assigns fields):
```java
record User(String name, int age) {
    public User {
        if (age < 0) throw new IllegalArgumentException();
    }
}
```

> [!warning] Shallow immutability only
> `record User(String name, List<String> roles) {}` — the `roles` **reference** can't be reassigned, but the underlying `List` is still mutable.
> **Fix:** defensive copy in compact constructor — `roles = List.copyOf(roles);`

> [!note] Spring Boot fit
> Great for immutable request/response DTOs, events, value objects — Jackson deserializes via the canonical constructor.
> **Don't** use as JPA entities — persistence/proxy/lifecycle needs don't fit the record model. Standard pattern: `JPA Entity → Service → Record DTO → API`.

---

## Sealed Interfaces

```java
sealed interface Payment permits CardPayment, UpiPayment {}

record CardPayment(String cardNumber) implements Payment {}
record UpiPayment(String upiId) implements Payment {}
```

- `permits` = closed, fixed set of allowed implementers (compile error if an unpermitted class tries)
- Every permitted subtype must itself be **`final`**, **`sealed`**, or **`non-sealed`**
- Records are implicitly `final` → ideal leaf implementations of a sealed hierarchy (common combo for modeling closed sets of immutable variants + pattern matching — see above)

> [!warning] Sealed ≠ immutable
> Sealed only controls the **inheritance hierarchy** (who can implement it) — it says nothing about whether instances are mutable. That's a separate property (records get it from being records, not from being sealed).

> [!tip] Mental model
> Sealed interface = interface + controlled inheritance hierarchy. Finalized in Java 17 (records: 16→17).
