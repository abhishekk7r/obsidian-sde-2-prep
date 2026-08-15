
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
- Return type ALONE can't distinguish overloads
- Static methods → **hidden**, not overridden. Resolved by reference type, not object type
- Private methods → not inherited/visible → "same signature" in subclass = new method, not override
- Covariant return types ARE allowed in overriding (subtype return ok)
- Access modifier can only **widen** on override (protected→public), never narrow
- Overriding method can throw same/narrower/no checked exception, never new/broader checked one
- Fields are NOT polymorphic — resolved by reference type (field hiding, not overriding)
- Constructors are never overridden (not inherited)
---
