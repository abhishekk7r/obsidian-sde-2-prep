Records purpose is to model **immutable data carriers with much less boilerplate**

## What problem do Records solve?
- Before records a simple class to represent users would look like this:
```java
public final class User { 
private final String name; 
private final int age; 
public User(String name, int age) { this.name = name; this.age = age; }

//getter 
//setter
//hashcode
//equal
}
```