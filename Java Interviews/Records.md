Records purpose is to model **immutable data carriers with much less boilerplate**

## 1. What problem do Records solve?
- Before records a simple class to represent users would look like this, they would carry a lot of boiler plate code:
```java
public final class User { 
private final String name; 
private final int age; 
public User(String name, int age) { this.name = name; this.age = age; }

/*
getter 
setter
hashcode
equals
*/
}
```

- `Records` solves this.
```java
  public record User(String name, int age) {}
  
  //You can create objects like:
  User user = new User("Abhishek", 26);
  System.out.println(user.name());
  System.out.println(user.age());
```
- The compiler automatically provides the fields, constructor, accessors, `equals()`, `hashCode()`, and `toString()`

- The accessor has the **same name as the record component**.
```java
//Normal Class
user.getName();
user.getAge();

//Records
user.name();
user.age();
```

## 2. What does Java generate automatically?
The exact generated implementation is compiler/JVM machinery. 
The important semantic point is that a record automatically provides:
	- private final component fields
	- public accessors
	- canonical constructor
	- `equals()`
	- `hashCode()`
	- `toString()`

## 3. Records are immutable — but understand what that REALLY means

Records are `shallowly immutable`, not necessarily deeply immutable.

Suppose:

```
record User(String name, int age) {}
```

You cannot do:

```
user.age = 30;
```

There are no setters.

The components are represented by final fields.
So this is impossible:

```
user.setAge(30);
```

or:

```
user.age = 30;
```

Records provide **shallow immutability**, not deep immutability.
This is extremely important.

Consider:

```
record User(String name, List<String> hobbies) {}
```

You cannot change the `hobbies` reference:

```
user.hobbies() = anotherList;
```

But you can potentially modify the underlying list:

```
user.hobbies().add("Swimming");
```

if that list itself is mutable.

So:

```
Record
  |
  +-- String -> immutable
  |
  +-- List -> reference is final
              but List may be mutable
```