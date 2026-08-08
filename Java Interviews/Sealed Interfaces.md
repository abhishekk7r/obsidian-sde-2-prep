## Core

A **sealed interface** restricts which classes/interfaces are allowed to implement or extend it.

```java
sealed interface Payment
    permits CardPayment, UpiPayment {
}
```

Only permitted types can implement it:

```java
record CardPayment(String cardNumber) implements Payment {}
record UpiPayment(String upiId) implements Payment {}
```

```text
Payment
├── CardPayment
└── UpiPayment
```

## Why Use Sealed Interfaces?

Use them when you want a **closed/fixed hierarchy**.

Normal interface:

```java
interface Payment {}
```

→ Any class can implement it.

Sealed interface:

```java
sealed interface Payment
    permits CardPayment, UpiPayment {}
```

→ Only explicitly permitted types can implement it.

Useful for:

- Fixed type hierarchies
    
- Domain modeling
    
- State/result types
    
- Pattern matching
    
- Modeling mutually exclusive variants
    

## `permits`

Specifies which classes/interfaces are allowed.

```java
sealed interface Payment
    permits CardPayment, UpiPayment {}
```

`CashPayment` cannot implement `Payment` unless it is added to `permits`.

## Permitted Types

A permitted subclass must be one of:

- `final` → cannot be extended
    
- `sealed` → restricts its own subclasses
    
- `non-sealed` → opens inheritance again
    

Example:

```java
sealed interface Payment
    permits CardPayment, UpiPayment {}

final class CardPayment implements Payment {}

sealed class UpiPayment implements Payment
    permits GooglePay {}

non-sealed class CashPayment implements Payment {}
```

## Records + Sealed Interfaces

Records are implicitly `final`, making them excellent leaf implementations.

```java
sealed interface Payment
    permits CardPayment, UpiPayment {}

record CardPayment(String cardNumber)
    implements Payment {}

record UpiPayment(String upiId)
    implements Payment {}
```

This combination is common in modern Java for modeling a fixed set of immutable variants.

## Important

- Sealed interfaces were finalized in **Java 17**.
    
- `permits` controls direct implementations.
    
- A permitted type must be `final`, `sealed`, or `non-sealed`.
    
- `record` implementations are automatically `final`.
    
- Sealed does **not** mean immutable.
    
- Sealed interfaces can extend other interfaces.
    
- A sealed interface can have abstract/default/static methods.
    

## Common Interview Questions

- **What is a sealed interface?** → An interface that restricts which classes/interfaces can directly implement or extend it.
    
- **Why use a sealed interface?** → To model a closed/fixed hierarchy and prevent unknown implementations.
    
- **What is `permits`?** → It specifies the types allowed to directly implement/extend the sealed type.
    
- **What must a permitted subclass be?** → `final`, `sealed`, or `non-sealed`.
    
- **Can a sealed interface be extended?** → Yes, but the extending interface must follow the sealed/non-sealed rules.
    
- **Can a Record implement a sealed interface?** → Yes; Records are implicitly `final`.
    
- **What happens if an unpermitted class implements a sealed interface?** → Compilation error.
    
- **Sealed vs normal interface?** → Normal interfaces allow arbitrary implementations; sealed interfaces restrict implementations to a known set.
    
- **Does sealed mean immutable?** → No. Sealed controls inheritance, not object mutability.
    
- **When were sealed interfaces finalized?** → Java 17.
    

### Mental Model

> **Sealed interface = interface + controlled inheritance hierarchy.**