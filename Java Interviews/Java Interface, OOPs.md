
## OOP — 4 Pillars

- **Encapsulation:** Protect internal state.
- **Abstraction:** Hide implementation details.
- **Inheritance:** `IS-A` relationship.
- **Polymorphism:** Same interface, different implementations.

> **Encapsulation = protect**  
> **Abstraction = hide**  
> **Inheritance = reuse**  
> **Polymorphism = substitute**

---

## Composition Over Inheritance

- Prefer **HAS-A** over inheritance when possible.
- Composition → lower coupling, easier testing/change.
- Inheritance → use for genuine **IS-A** relationships.

## Notification Example

```
OrderService
     ↓
Factory / Map
     ↓
NotificationService
   ↙   ↓   ↘
Email SMS Slack
```

- **Interface:** Contract.
- **Strategy:** Email/SMS/Slack are interchangeable behaviors.
- **Factory:** Selects the implementation.
- **DI:** Supplies the dependency to `OrderService`.
- **DIP:** Business logic depends on the interface, not concrete classes.

> **Interface → Strategy → Factory selects → DI provides**

___

## Interface vs Abstract Class

- **Interface:** Contract/capability → `implements`
- **Abstract class:** Common base + shared state/implementation → `extends`
- Interface can extend multiple interfaces; class can extend only one class.
- Both can have abstract + concrete methods.
- **Abstract class can implement an interface** without implementing all methods.
- **Interface extends interface**, not implements.

> **Interface = WHAT**  
> **Abstract class = WHAT + shared HOW**

---

## Abstract vs Virtual

- **Abstract method:** No implementation; concrete subclass must implement.
- **Virtual method:** Java has no `virtual` keyword; normal instance methods can be overridden and use runtime dispatch.
- `static`, `private`, `final` methods are not dynamically overridden.

> **Abstract = must implement**  
> **Virtual = can override**
> ----


## Overloading vs Overriding

**Overloading** — same name, diff params, resolved at compile-time (static binding). Same class ok.
**Overriding** — same signature, subclass redefines parent method, resolved at runtime (dynamic dispatch). Needs inheritance + @Override.

### Rules / Traps
- **Overload resolution priority:** exact match > widening > autoboxing > varargs
- **Overloading with `null` + ambiguous types** (String vs StringBuilder) → compile error; needs cast
- **Return type ALONE can't distinguish overloads**
- Static methods → **hidden**, not overridden. Resolved by reference type, not object type
- Private methods → not inherited/visible → "same signature" in subclass = new method, not override
- Covariant return types ARE allowed in overriding (subtype return ok)
- Access modifier can only **widen** on override (protected→public), never narrow
- Overriding method can throw same/narrower/no checked exception, never new/broader checked one
- Fields are NOT polymorphic — resolved by reference type (field hiding, not overriding)
- Constructors are never overridden (not inherited)
---
# equals()/hashCode() + String Immutability — Quick Revision

## 🔑 THE ONE RULE TO KNOW COLD
equals() true  →  hashCode() MUST match   (mandatory)
hashCode() match → equals() MAY be false  (hash collision — legal, just slower)

---

## TRAP 1 — Override equals() only, skip hashCode()
Symptom: HashSet allows "duplicate" equal elements. contains()/get() silently fail.
Why: equal objects → different default hashCode → different buckets → never compared via equals().
Fix: Always override BOTH. Use Objects.hash(fields) or generate via IDE/Lombok.

## TRAP 2 — Mutate a field used in equals()/hashCode() AFTER using object as a key
Symptom: get(key) returns null even though the entry still exists (shows up in iteration).
Why: Entry sits in the bucket from INSERT time. Mutating the field changes the hash,
     so get() now searches a DIFFERENT bucket than where the entry actually lives.
Fix: Never mutate fields involved in equals()/hashCode() while object is a map key /
     set element. Prefer immutable keys.

## TRAP 3 — Hash collision ≠ contract violation
Two UNEQUAL objects CAN share a hashCode → totally legal (collision, O(1)→O(n) cost).
Two EQUAL objects must ALWAYS share a hashCode → mandatory, violating this = bug.
Don't confuse the two directions.

---

## String Immutability & Pool

| Comparison                          | == (reference) | .equals() (value) |
|--------------------------------------|:---:|:---:|
| "abc" vs "abc" (both literals)       | true  | true |
| "abc" vs new String("abc")           | false | true |
| new String("abc") vs new String("abc") | false | true |
| new String("abc").intern() vs "abc"  | true  | true |

- String pool = part of the main HEAP since Java 7 (was PermGen before — common wrong answer).
- Literals → pool reference automatically. new String(...) → separate heap object, NOT pooled.
- .intern() → looks up pool for equal value; returns pooled ref if found, else adds & returns.
- Any "modifying" method (concat, +, replace...) returns a NEW String — original never changes.

## Why immutability → safe pooling
Pooled strings are SHARED BY REFERENCE across the whole JVM.
If String were mutable → changing one reference would corrupt every other holder of that
pooled value. Immutability = guarantee that can never happen.
→ StringBuilder is deliberately mutable, so it CANNOT be pooled/shared this way.

## Why String is a great HashMap key
Immutable → hashCode() can never change after construction (JVM even CACHES the
computed hash on first call) → immune to Trap 2 → equal strings always hash the
same, forever.