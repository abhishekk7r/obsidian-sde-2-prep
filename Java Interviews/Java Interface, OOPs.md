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

> [!tip] Chain Interface → Strategy → Factory selects → DI provides

---

## Interface vs Abstract Class

|Interface|Abstract Class|
|---|---|---|
|Keyword|`implements`|`extends`|
|Multiple?|Extends multiple interfaces|Extends only one class|
|Methods|Abstract + concrete (default)|Abstract + concrete|
|Can implement interface without all methods?|—|✅ yes|

> [!tip] Interface = **WHAT** · Abstract class = **WHAT + shared HOW** Interface _extends_ interface — never _implements_

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
> 
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

> [!important] The one rule `equals() == true` → `hashCode()` **must** match (mandatory) `hashCode()` match → `equals()` **may** be false (collision — legal, just slower)

> [!warning] Trap 1 — override equals() only **Symptom:** `HashSet` allows "duplicate" equal elements; `contains()`/`get()` silently fail **Why:** equal objects → different default hashCode → different buckets → never compared **Fix:** always override both. `Objects.hash(fields)` or IDE/Lombok

> [!warning] Trap 2 — mutate a field used in equals()/hashCode() after using object as a key **Symptom:** `get(key)` → null, even though entry still shows in iteration **Why:** entry sits in the bucket from insert-time hash; mutation changes the hash → `get()` searches the wrong bucket **Fix:** never mutate key-identity fields post-insert. Prefer immutable keys

> [!note] Trap 3 — collision ≠ contract violation Unequal objects **can** share hashCode (legal collision, O(1)→O(n)) Equal objects **must always** share hashCode (mandatory)

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

> [!tip] Why immutability enables pooling Pooled strings are shared **by reference** JVM-wide. If mutable, changing one ref would corrupt every holder. Immutability = that can never happen → `StringBuilder` is deliberately mutable, so it can't be pooled. Bonus: immutable → `hashCode()` cached after first call → immune to Trap 2 → ideal HashMap key.

---

## Collections — ArrayDeque, PriorityQueue, TreeMap/Set, LinkedHashMap

**ArrayDeque vs LinkedList**

|ArrayDeque|LinkedList|
|---|---|---|
|Backing|Circular resizable array|Doubly-linked nodes|
|Nulls|❌ not allowed|✅ allowed|
|Both ends|O(1) amortized|O(1) amortized|
|Mid-removal|—|O(1) _only_ with node ref (iterator)|
|Preferred for stack/queue|✅|—|

> [!note] Both implement `Deque` — front/back access is **not** the differentiator. Cache locality + memory overhead is.

**PriorityQueue**

- Backed by **binary heap** (array-based) — unrelated to Deque despite the name
- `add`/`offer`: O(log n) · `poll`/`remove`: O(log n) · `peek`: O(1)

> [!warning] Java vs C++ default — classic mixup Java no-arg `PriorityQueue` = **min-heap** C++ `std::priority_queue` default = **max-heap** (opposite!)

```java
// Java max-heap
new PriorityQueue<>((a, b) -> b[0] - a[0]);
```

```cpp
// C++ max-heap comparator — must be a[0] < b[0] (NOT >)
auto cmp = [](auto& a, auto& b){ return a[0] < b[0]; };
```

- No ordering guarantee among equal elements (not stable) — add explicit tiebreaker if needed
- Use `Integer.compare(a,b)` over subtraction (overflow risk)

**TreeMap / TreeSet**

- Backed by **Red-Black tree**. get/put/contains: O(log n) vs O(1) avg HashMap
- Navigation (read-only, O(log n), single root-to-leaf walk):

|Method|Returns|
|---|---|
|`floorKey(k)`|largest key ≤ k|
|`ceilingKey(k)`|smallest key ≥ k|
|`lowerKey(k)`|largest key < k (strict)|
|`higherKey(k)`|smallest key > k (strict)|
|`firstKey()`/`lastKey()`|O(log n)|

> [!warning] Trap 2, TreeMap edition Mutating a field used in `compareTo`/comparator **after insertion** breaks the BST invariant for that node. Lookups walk the wrong path → node present but unreachable. Iteration order silently corrupts too. **HashMap fails via wrong bucket (hash). TreeMap fails via wrong tree path (comparison). Same rule either way: don't mutate key identity post-insert.**

**LinkedHashMap → LRU Cache**

```java
class LRUCache<K,V> extends LinkedHashMap<K,V> {
    private final int capacity;
    LRUCache(int capacity) {
        super(16, 0.75f, true); // true = access-order
        this.capacity = capacity;
    }
    @Override
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return size() > capacity;
    }
}
```

- 3rd ctor arg `true` = access-order (get/put → moves entry to MRU end). Default `false` = insertion-order
- **Must override `removeEldestEntry`** — without it, map never evicts, grows unbounded
- C++ has no equivalent — manual: `unordered_map<K, list<pair<K,V>>::iterator> + list<pair<K,V>>`

**ArrayDeque + for-each removal**

> [!danger] Direct `deque.remove(x)` inside a for-each → `ConcurrentModificationException` (fail-fast iterator, mod-count check on `next()`) **Fix:** `Iterator.remove()` in a manual `while(it.hasNext())`, or `deque.removeIf(predicate)` (Java 8+)