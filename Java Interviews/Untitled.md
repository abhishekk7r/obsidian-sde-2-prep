# Java OOP & Design Patterns — Quick Revision

## Interface vs Abstract Class

- **Interface:** Contract/capability → `implements`
- **Abstract class:** Common base + shared state/implementation → `extends`
- Interface can extend multiple interfaces; class can extend only one class.
- Both can have abstract + concrete methods.
- **Abstract class can implement an interface** without implementing all methods.
- **Interface extends interface**, not implements.

> **Interface = WHAT**  
> **Abstract class = WHAT + shared HOW**

## Abstract vs Virtual

- **Abstract method:** No implementation; concrete subclass must implement.
- **Virtual method:** Java has no `virtual` keyword; normal instance methods can be overridden and use runtime dispatch.
- `static`, `private`, `final` methods are not dynamically overridden.

> **Abstract = must implement**  
> **Virtual = can override**

## OOP — 4 Pillars

- **Encapsulation:** Protect internal state.
- **Abstraction:** Hide implementation details.
- **Inheritance:** `IS-A` relationship.
- **Polymorphism:** Same interface, different implementations.

> **Encapsulation = protect**  
> **Abstraction = hide**  
> **Inheritance = reuse**  
> **Polymorphism = substitute**

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