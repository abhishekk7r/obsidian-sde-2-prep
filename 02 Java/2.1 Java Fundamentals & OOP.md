# Java Fundamentals & OOP

## JVM / JRE / JDK

|         | Contains                                                                                          | Use case                            |
| ------- | ------------------------------------------------------------------------------------------------- | ----------------------------------- |
| **JVM** | Runs bytecode → machine code. Platform-dependent (bytecode itself is not). Includes JIT compiler. | Executes the program                |
| **JRE** | JVM + class libraries                                                                             | Enough to just *run* a Java program |
| **JDK** | JRE + dev tools (compiler, debugger, etc.)                                                        | Needed to *build* Java programs     |

> [!tip] JDK ⊃ JRE ⊃ JVM — each layer wraps the previous one

---

## OOP — 4 Pillars

| Pillar        | Meaning                                   | Mnemonic       |
| ------------- | ----------------------------------------- | -------------- |
| Encapsulation | Protect internal state                    | **protect**    |
| Abstraction   | Hide implementation details               | **hide**       |
| Inheritance   | `IS-A` relationship                       | **reuse**      |
| Polymorphism  | Same interface, different implementations | **substitute** |

---

## Interface vs Abstract Class

|                                              | Interface                     | Abstract Class         |
| -------------------------------------------- | ----------------------------- | ---------------------- |
| Keyword                                      | `implements`                  | `extends`              |
| Multiple?                                    | Extends multiple interfaces   | Extends only one class |
| Methods                                      | Abstract + concrete (default) | Abstract + concrete    |
| Can implement interface without all methods? | —                             | ✅ yes                  |
An abstract class provides a partial code implementation with state and constructors for a single inheritance tree, whereas an interface acts as a pure behavior contract supporting multiple implementation.

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

|           | Overloading            | Overriding                 |
| --------- | ---------------------- | -------------------------- |
| Signature | Same name, diff params | Same signature             |
| Binding   | Compile-time (static)  | Runtime (dynamic dispatch) |
| Needs     | Same class OK          | Inheritance + `@Override`  |

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

| Keyword      | What it does                                                                                                 |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| `final`      | Variable → constant (no reassignment). Method → can't override. Class → can't extend                         |
| `finally`    | Block that **always** runs after try/catch (cleanup) — skipped only on `System.exit()` or JVM crash          |
| `finalize()` | Deprecated (Java 9+) GC hook, called before object collection — **unreliable, never rely on it** for cleanup |

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
