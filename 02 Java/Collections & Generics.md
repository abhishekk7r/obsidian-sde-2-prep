# Collections & Generics

> [!note] Open question
> So no idea what is the tree map and tree. What do they do?

## ArrayDeque vs LinkedList

|ArrayDeque|LinkedList|
|---|---|
|Backing|Circular resizable array|
|Nulls|❌ not allowed|
|Both ends|O(1) amortized|
|Mid-removal|O(1) _only_ with node ref (iterator)|
|Preferred for stack/queue|✅|

> [!note] Both implement `Deque` — front/back access is **not** the differentiator. Cache locality + memory overhead is.

---

## ArrayDeque + for-each removal

> [!danger]
> Direct `deque.remove(x)` inside a for-each → `ConcurrentModificationException` (fail-fast iterator, mod-count check on `next()`)
> **Fix:** `Iterator.remove()` in a manual `while(it.hasNext())`, or `deque.removeIf(predicate)` (Java 8+)

---

## PriorityQueue

- Backed by **binary heap** (array-based) — unrelated to Deque despite the name
- `add`/`offer`: O(log n) · `poll`/`remove`: O(log n) · `peek`: O(1)

> [!warning] Java vs C++ default — classic mixup
> Java no-arg `PriorityQueue` = **min-heap**
> C++ `std::priority_queue` default = **max-heap** (opposite!)

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

---

## TreeMap / TreeSet

- Backed by **Red-Black tree**. get/put/contains: O(log n) vs O(1) avg HashMap
- Navigation (read-only, O(log n), single root-to-leaf walk):

|Method|Returns|
|---|---|
|`floorKey(k)`|largest key ≤ k|
|`ceilingKey(k)`|smallest key ≥ k|
|`lowerKey(k)`|largest key < k (strict)|
|`higherKey(k)`|smallest key > k (strict)|
|`firstKey()`/`lastKey()`|O(log n)|

> [!warning] Trap — TreeMap edition
> Mutating a field used in `compareTo`/comparator **after insertion** breaks the BST invariant for that node. Lookups walk the wrong path → node present but unreachable. Iteration order silently corrupts too.
> **HashMap fails via wrong bucket (hash). TreeMap fails via wrong tree path (comparison). Same rule either way: don't mutate key identity post-insert.**

---

## LinkedHashMap → LRU Cache

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

---

## Generics — bounded types & wildcards

- Generics are **compile-time only** — type safety without casts, checked by compiler

**Bounded type param:** `<T extends Comparable<T>>` restricts T to types implementing `Comparable`. Loosened form `<T extends Comparable<? super T>>` also accepts subclasses that inherit `compareTo` from a parent (e.g. `Dog extends Animal implements Comparable<Animal>` — `Dog` fails the strict bound, passes the loose one). JDK uses this in `Collections.max()`.

**Wildcards (PECS — Producer Extends, Consumer Super):**

| Form | Meaning | Use when |
|---|---|---|
| `List<?>` | Unknown type | Only need `size()`/`isEmpty()`, no type-specific ops |
| `List<? extends T>` | Producer, read-only | Only reading T out |
| `List<? super T>` | Consumer, write-only (beyond Object) | Only writing T in |

**Type erasure** — generic type info removed after compile; `List<String>` and `List<Integer>` are both just `List` (`ArrayList.class`) at runtime.

Consequences:
```java
list instanceof List<String>   // compile error — erased, can't check
T[] arr = new T[10];           // compile error — JVM doesn't know T at runtime
```
Workaround for generic arrays:
```java
@SuppressWarnings("unchecked")
T[] arr = (T[]) new Object[10];               // unsafe but common
T[] arr = (T[]) Array.newInstance(clazz, 10); // cleaner, needs Class<T> token
```

> [!tip] Mnemonic
> PECS: Producer Extends, Consumer Super. Erasure = compiler's safety net, gone by runtime.
