# Java Records

> Introduced as a concise way to model immutable data carriers. Finalized in Java 16; available in Java 17.

## Core

```
record User(String name, int age) {}
```

Automatically provides:

- `private final` component fields
- Canonical constructor
- Accessors: `name()`, `age()`
- `equals()`
- `hashCode()`
- `toString()`

## Key Properties

- Record is implicitly `final`
- Cannot extend another class
- Can implement interfaces
- Can have methods and static members
- Cannot have additional instance fields
- Cannot have mutable components themselves
- Can be generic
- Supports annotations

## Accessors

```
user.name()
user.age()
```

Not JavaBean getters:

```
user.getName()
```

## Immutability

Records provide **shallow immutability**.

```
record User(String name, List<String> roles) {}
```

The `roles` reference cannot be reassigned, but the underlying `List` may still be mutable.

For defensive immutability:

```
public User {
    roles = List.copyOf(roles);
}
```

## Constructors

### Canonical Constructor

Constructor containing all record components.

```
record User(String name, int age) {}
```

### Compact Constructor

Useful for validation/normalization.

```
record User(String name, int age) {
    public User {
        if (age < 0)
            throw new IllegalArgumentException();
    }
}
```

The compiler handles component assignment.

Additional constructors must ultimately delegate to the canonical constructor.

## Methods

Records can contain normal methods and implement interfaces.

```
record Rectangle(double length, double width) {
    double area() {
        return length * width;
    }
}
```

## Equality

Record `equals()` and `hashCode()` are based on component values.

```
record User(String name, int age) {}

new User("A", 20).equals(new User("A", 20)); // true
```

`==` still compares object references.

## Spring Boot

Excellent for immutable:

- Request DTOs
- Response DTOs
- Events/messages
- Value objects
- API projections

Example:

```
record CreateUserRequest(
    String name,
    String email
) {}
```

Jackson/Spring Boot can deserialize records using their canonical constructor.

## Records vs JPA

Generally **don't use records as JPA entities**.

Typical pattern:

```
JPA Entity → Service → Record DTO → API
```

Entities have persistence/lifecycle/proxy requirements that don't fit the record model well.

## Records + [[Sealed Interfaces]]

Records work well as implementations of sealed interfaces:

```
sealed interface Payment
    permits CardPayment, UpiPayment {}

record CardPayment(String card) implements Payment {}
record UpiPayment(String upiId) implements Payment {}
```

## Common Interview Questions

1. **What is a Record?** → A special class designed for concise, immutable data carriers with minimal boilerplate.
2. **Why were Records introduced?** → To eliminate boilerplate in data-centric classes.
3. **What does Java automatically provide for a Record?** → Canonical constructor, accessors, `equals()`, `hashCode()`, and `toString()`.
4. **When were Records introduced?** → Finalized in Java 16; available in Java 17.
5. **Are Records immutable?** → Shallowly immutable; components are final, but referenced objects can still be mutable.
6. **What are Record accessors?** → Methods named after components, e.g. `user.name()`, not `user.getName()`.
7. **What is a canonical constructor?** → A constructor whose parameters correspond exactly to all Record components.
8. **What is a compact constructor?** → A concise canonical constructor mainly used for validation or normalization.
9. **Can Records have additional constructors?** → Yes, but they must ultimately delegate to the canonical constructor.
10. **Can Records have methods?** → Yes, including instance and static methods.
11. **Can Records implement interfaces?** → Yes.
12. **Can Records extend another class?** → No; a Record already extends `java.lang.Record`.
13. **Can a Record be extended?** → No; Records are implicitly `final`.
14. **Can Records have additional instance fields?** → No; instance state is defined by the Record components.
15. **Can Records have static fields?** → Yes.
16. **Can Records be generic?** → Yes, e.g. `record Pair<T, U>(T first, U second) {}`.
17. **How does** `**equals()**` **work in Records?** → It compares the Record's component values.
18. **Difference between** `**==**` **and** `**equals()**` **for Records?** → `==` compares references; `equals()` compares component values.
19. **Are Records deeply immutable?** → No; they provide only shallow immutability.
20. **Can a Record contain a mutable** `**List**` **or** `**Map**`**?** → Yes, but the referenced collection can still be modified.
21. **How can you make a collection inside a Record safely immutable?** → Use a defensive copy such as `List.copyOf()`.
22. **Can a Record be abstract?** → No.
23. **Can a Record have setters?** → Not meaningful for components because their fields are final.
24. **Is a Record just syntactic sugar?** → No; it also has specific language/JVM semantics and Record metadata.
25. **Can you override** `**equals()**`**,** `**hashCode()**`**, and** `**toString()**`**?** → Yes.
26. **Can Record components have annotations?** → Yes; commonly used for validation in Spring Boot.
27. **Why are Records useful in Spring Boot?** → They are ideal for immutable request/response DTOs, events, projections, and value objects.
28. **Can Jackson deserialize Records?** → Yes; it can use the canonical constructor.
29. **Should Records generally be used as JPA entities?** → No; JPA entities have lifecycle, proxy, and persistence requirements better suited to normal classes.
30. **Record vs Lombok** `**@Data**`**?** → `@Data` typically creates a mutable class; a Record is designed as a final data carrier.
31. **Record vs normal class?** → Use a Record for immutable data-centric objects; use a class when you need mutation, inheritance, or complex lifecycle.

### Important Traps

- **Does** `**record User(String name) {}**` **have** `**getName()**`**?** → No, it has `name()`.
- **Does** `**record User(List<String> roles) {}**` **guarantee deep immutability?** → No.
- **Can** `**class Admin extends User**` **work if** `**User**` **is a Record?** → No, Records are final.
- **Can** `**record Admin(...) extends User**` **work?** → No, Records cannot extend classes.
- **Can a Record contain business logic?** → Yes; Records can contain methods.
- **Can a Record implement** `**Serializable**`**?** → Yes.