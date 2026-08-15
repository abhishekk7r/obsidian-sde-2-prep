# Java Interview — Fast Revision

---

## JVM / JRE / JDK

| |Contains|Use case|
|---|---|---|
|**JVM**|Runs bytecode → machine code. Platform-dependent (bytecode itself is not). Includes JIT compiler.|Executes the program|
|**JRE**|JVM + class libraries|Enough to just *run* a Java program|
|**JDK**|JRE + dev tools (compiler, debugger, etc.)|Needed to *build* Java programs|

> [!tip] JDK ⊃ JRE ⊃ JVM — each layer wraps the previous one

---

## OOP — 4 Pillars

|Pillar|Meaning|Mnemonic|
|---|---|---|
|Encapsulation|Protect internal state|**protect**|
|Abstraction|Hide implementation details|**hide**|
|Inheritance|`IS-A` relationship|**reuse**|
|Polymorphism|Same interface, different implementations|**substitute**|

---

## Interface vs Abstract Class

|Interface|Abstract Class|
|---|---|---|
|Keyword|`implements`|`extends`|
|Multiple?|Extends multiple interfaces|Extends only one class|
|Methods|Abstract + concrete (default)|Abstract + concrete|
|Can implement interface without all methods?|—|✅ yes|

> [!tip]
> Interface = **WHAT** · Abstract class = **WHAT + shared HOW**
> Interface _extends_ interface — never _implements_

---

## Abstract vs Virtual

- **Abstract method** → no implementation, concrete subclass must implement
- **Virtual** → Java has no `virtual` keyword; normal instance methods override + runtime-dispatch by default
- `static`, `private`, `final` methods → **not** dynamically overridden

> [!tip] Abstract = must implement · Virtual = can override

---

## Composition Over Inheritance

- Prefer **HAS-A** > `extends` when possible → lower coupling, easier testing/change
- Inheritance → only for genuine **IS-A**

**Example — Notification flow**

```
OrderService → Factory/Map → NotificationService → {Email, SMS, Slack}
```

- **Interface** = contract
- **Strategy** = Email/SMS/Slack are interchangeable behaviors
- **Factory** = selects implementation
- **DI** = supplies dependency to `OrderService`
- **DIP** = business logic depends on interface, not concrete class

> [!tip] Chain
> Interface → Strategy → Factory selects → DI provides

---

## Overloading vs Overriding

|Overloading|Overriding|
|---|---|---|
|Signature|Same name, diff params|Same signature|
|Binding|Compile-time (static)|Runtime (dynamic dispatch)|
|Needs|Same class OK|Inheritance + `@Override`|

> [!warning] Traps
> - Resolution priority: **exact match > widening > autoboxing > varargs**
> - `null` + ambiguous overloads (String vs StringBuilder) → compile error, needs cast
> - Return type **alone** can't distinguish overloads
> - `static` methods → **hidden**, not overridden — resolved by reference type
> - `private` methods → not inherited → "override" in subclass is actually a new method
> - Covariant return types **are** allowed on override
> - Access modifier can only **widen** on override (protected→public), never narrow
> - Override can throw same/narrower/no checked exception — never new/broader
> - Fields are **not polymorphic** — resolved by reference type (hiding, not overriding)
> - Constructors are never overridden

---

## Static Keyword & Initialization Order

- `static` field/method belongs to the **class**, not an instance — shared across all objects
- `static` block runs **once**, at class-load time, before `main()` / before first instantiation
- Static members live in the **method area** (Metaspace, Java 8+; was PermGen before) — not stack, not heap-per-object

**Init order (per class load, then per instance):**
```
static fields/blocks (once, class load, textual top-to-bottom order)
      ↓
instance fields/blocks (every instantiation, textual order)
      ↓
constructor body (instance init runs after super(), before rest of ctor)
```

**Nested classes don't cascade init:**
- A static nested class's own static block only runs when *it* is first actively used — outer class loading does **not** trigger it

> [!warning] Trap
> Can't access instance members directly from a static context (`static void main` can't touch instance fields without an object reference) — no `this` exists in static scope.

> [!danger] Trap — static init failure is permanent
> If a static block throws, JVM wraps it as `ExceptionInInitializerError` and the class is marked erroneous for the **rest of the JVM run**. Every later `new` on it throws `NoClassDefFoundError` — it does not retry the static block.

> [!tip] Mnemonic: "Load once, poison forever."

---

## final vs finally vs finalize

|Keyword|What it does|
|---|---|
|`final`|Variable → constant (no reassignment). Method → can't override. Class → can't extend|
|`finally`|Block that **always** runs after try/catch (cleanup) — skipped only on `System.exit()` or JVM crash|
|`finalize()`|Deprecated (Java 9+) GC hook, called before object collection — **unreliable, never rely on it** for cleanup|

> [!tip] Prefer try-with-resources / `Cleaner` over `finalize()` — deterministic vs "whenever GC feels like it"

**Why `finalize()` is dead:** no guarantee it ever runs or when; runs on a random GC thread (race conditions); object can "resurrect" itself by re-adding a reference; real perf overhead on collection.

> [!danger] Trap — `finally` + `return` in `try`
> `finally` always runs, even after a `try` `return`. But the `try`'s return value is **captured before `finally` runs** — so a `finally` that reassigns a local primitive doesn't change what's returned. A `finally` with its **own `return`** silently overrides the `try`'s return — legal, but a bug magnet.
> ```java
> int f() {
>   try { return 1; }
>   finally { return 2; }   // returns 2 — try's return is discarded
> }
> ```

> [!tip] Mnemonic: "finally always fires; last return wins."

---

## Autoboxing & Integer Cache

- Autoboxing = automatic primitive ↔ wrapper conversion (`int` ↔ `Integer`), via `Integer.valueOf()` — not `new Integer()`
- **Integer cache:** JVM caches boxed `Integer` values in range **-128 to 127**, returned by `valueOf()`. Range tunable up via `-XX:AutoBoxCacheMax`, not down

> [!danger] Classic trap
> ```java
> Integer a = 127, b = 127;
> a == b        // true  — both pulled from cache via valueOf()
> Integer x = 200, y = 200;
> x == y        // false — outside cache range, different objects
> ```
> **Fix:** always use `.equals()` for wrapper comparisons, never `==`.

> [!danger] Trap — `new Integer()` bypasses the cache entirely
> ```java
> Integer a = new Integer(127);   // forces new object, skips cache
> Integer b = 127;                // valueOf() → cached object
> a == b   // false — even though both are within cache range
> ```
> `Integer(int)` constructor is deprecated since Java 9 for exactly this reason.

> [!tip] Mnemonic: "`new` says no to the cache."

---

## equals() / hashCode() + String Immutability

> [!important] The one rule
> `equals() == true` → `hashCode()` **must** match (mandatory)
> `hashCode()` match → `equals()` **may** be false (collision — legal, just slower)

> [!warning] Trap 1 — override equals() only
> **Symptom:** `HashSet` allows "duplicate" equal elements; `contains()`/`get()` silently fail
> **Why:** equal objects → different default hashCode → different buckets → never compared
> **Fix:** always override both. `Objects.hash(fields)` or IDE/Lombok

> [!warning] Trap 2 — mutate a field used in equals()/hashCode() after using object as a key
> **Symptom:** `get(key)` → null, even though entry still shows in iteration
> **Why:** entry sits in the bucket from insert-time hash; mutation changes the hash → `get()` searches the wrong bucket
> **Fix:** never mutate key-identity fields post-insert. Prefer immutable keys

> [!note] Trap 3 — collision ≠ contract violation
> Unequal objects **can** share hashCode (legal collision, O(1)→O(n))
> Equal objects **must always** share hashCode (mandatory)

**String pool**

|Comparison|`==`|`.equals()`|
|---|:-:|:-:|
|`"abc"` vs `"abc"`|true|true|
|`"abc"` vs `new String("abc")`|false|true|
|`new String("abc")` vs `new String("abc")`|false|true|
|`new String("abc").intern()` vs `"abc"`|true|true|

- Pool = main **heap** since Java 7 (was PermGen — common wrong answer)
- Literals → pool ref automatically. `new String(...)` → separate heap object, not pooled
- `.intern()` → returns pooled ref if value exists, else adds it
- Any "modifying" method (`concat`, `+`, `replace`) → returns a **new** String, original untouched

> [!tip] Why immutability enables pooling
> Pooled strings are shared **by reference** JVM-wide. If mutable, changing one ref would corrupt every holder. Immutability = that can never happen → `StringBuilder` is deliberately mutable, so it can't be pooled.
> Bonus: immutable → `hashCode()` cached after first call → immune to Trap 2 → ideal HashMap key.

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

## Exception Handling

|Checked|Unchecked (Runtime)|
|---|---|
|Enforced at compile time — must catch or declare `throws`|Not enforced — compiler doesn't check|
|`IOException`, `SQLException`|`NullPointerException`, `ArrayIndexOutOfBoundsException`, `IllegalArgumentException`|
|Represents recoverable external conditions|Represents programming bugs|

**try-with-resources** — auto-closes any `AutoCloseable`, no manual `finally { close(); }`:
```java
try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
} // br.close() called automatically, even on exception
```

**Custom exceptions:** extend `Exception` (checked) or `RuntimeException` (unchecked) depending on whether callers should be forced to handle it.

> [!warning] Trap — multi-catch order
> ```java
> catch (IOException e) {}
> catch (FileNotFoundException e) {} // unreachable — compile error
> ```
> Subclass exceptions must be caught **before** their superclass, or the later catch block is unreachable.

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

## Generics — bounded types & wildcards

- Generics are **compile-time only** — type safety without casts, checked by compiler

**Bounded type param:** `<T extends Comparable<T>>` restricts T to types implementing `Comparable`. Loosened form `<T extends Comparable<? super T>>` also accepts subclasses that inherit `compareTo` from a parent (e.g. `Dog extends Animal implements Comparable<Animal>` — `Dog` fails the strict bound, passes the loose one). JDK uses this in `Collections.max()`.

**Wildcards (PECS — Producer Extends, Consumer Super):**

| Form | Meaning | Use when |
|---|---|---|
| `List<?>` | Unknown type | Only need `size()`/`isEmpty()`, no type-specific ops |
| `List<? extends T>` | Producer, read-only | Only reading T out |
| `List<? super T>` | Consumer, write-only (beyond Object) | Only writing T in |

**Type erasure** — generic type info removed after compile; `List<String>` and `List<Integer>` are both just `List` (`ArrayList.class`) at runtime.

Consequences:
```java
list instanceof List<String>   // compile error — erased, can't check
T[] arr = new T[10];           // compile error — JVM doesn't know T at runtime
```
Workaround for generic arrays:
```java
@SuppressWarnings("unchecked")
T[] arr = (T[]) new Object[10];               // unsafe but common
T[] arr = (T[]) Array.newInstance(clazz, 10); // cleaner, needs Class<T> token
```

> [!tip] Mnemonic
> PECS: Producer Extends, Consumer Super. Erasure = compiler's safety net, gone by runtime.

---
