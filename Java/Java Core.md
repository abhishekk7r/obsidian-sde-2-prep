# Java Interview — Fast Revision

---

## OOP — 4 Pillars

|Pillar|Meaning|Mnemonic|
|---|---|---|
|Encapsulation|Protect internal state|**protect**|
|Abstraction|Hide implementation details|**hide**|
|Inheritance|`IS-A` relationship|**reuse**|
|Polymorphism|Same interface, different implementations|**substitute**|

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

## JVM / JRE / JDK

| |Contains|Use case|
|---|---|---|
|**JVM**|Runs bytecode → machine code. Platform-dependent (bytecode itself is not). Includes JIT compiler.|Executes the program|
|**JRE**|JVM + class libraries|Enough to just *run* a Java program|
|**JDK**|JRE + dev tools (compiler, debugger, etc.)|Needed to *build* Java programs|

> [!tip] JDK ⊃ JRE ⊃ JVM — each layer wraps the previous one

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
- Records are implicitly `final` → ideal leaf implementations of a sealed hierarchy (common combo for modeling closed sets of immutable variants + pattern matching)

> [!warning] Sealed ≠ immutable
> Sealed only controls the **inheritance hierarchy** (who can implement it) — it says nothing about whether instances are mutable. That's a separate property (records get it from being records, not from being sealed).

> [!tip] Mental model
> Sealed interface = interface + controlled inheritance hierarchy. Finalized in Java 17 (records: 16→17).