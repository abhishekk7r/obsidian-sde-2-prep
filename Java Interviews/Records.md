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

## Records + Sealed Interfaces

Records work well as implementations of sealed interfaces:

```
sealed interface Payment
    permits CardPayment, UpiPayment {}

record CardPayment(String card) implements Payment {}
record UpiPayment(String upiId) implements Payment {}
```