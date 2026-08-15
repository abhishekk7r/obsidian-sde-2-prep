# Collections — Fast Revision

- Collision → new `Node` appended to that bucket's linked list (`equals()` decides key-match/overwrite vs new chained entry)
- Resize: triggers when `size > capacity * loadFactor` (default load factor **0.75**) → capacity **doubles** → nearly every entry **rehashed** (bucket index depends on capacity) — O(n) operation, not a cheap copy

**Treeification (Java 8+):**
- Bucket → red-black tree at **8** entries (`TREEIFY_THRESHOLD`), tree → back to list at **6** (`UNTREEIFY_THRESHOLD`) — gap is deliberate hysteresis, prevents thrashing at the boundary
- Needs `Comparable`/`Comparator` — tree requires total ordering for left/right placement, `equals()`/`hashCode()` alone can't do that. Falls back to class-name + `identityHashCode()` if keys aren't `Comparable`

|Time complexity|Pre-Java-8|Java 8+ (treeified)|
|---|---|---|
|`get()` worst case|O(n) — all keys collide into one bucket|O(log n) per bucket|

> [!danger] Trap — why treeification exists
> Pre-Java-8 O(n) worst case was a real **DoS vector**: attacker crafts keys with colliding hash codes (e.g. HTTP params), forces one bucket to hold everything, degrades every op to O(n). Treeification caps worst case at O(log n), closing the attack.

> [!tip] Why power-of-2 capacity: `capacity - 1` is a clean bitmask → `hash & (capacity-1)` ≡ `hash % capacity` but as fast bitwise AND

> [!danger] Trap — HashMap is NOT thread-safe
> Concurrent `put()`s → no exception, **silent** failure: lost writes, corrupted bucket structure, Java 7 had a famous **infinite loop on concurrent resize** (100% CPU hang in prod). `ConcurrentModificationException` is a *different*, narrower issue — from iterating while structurally modifying (fail-fast iterator), not concurrency per se.
> **Fix:** `ConcurrentHashMap` — bucket-level locking (CAS + synchronized on bin heads), no global lock.

**Mnemonic:** "8 up, 6 down; power-of-2 for speed; never trust HashMap with threads."


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

## ArrayDeque + for-each removal

> [!danger]
> Direct `deque.remove(x)` inside a for-each → `ConcurrentModificationException` (fail-fast iterator, mod-count check on `next()`)
> **Fix:** `Iterator.remove()` in a manual `while(it.hasNext())`, or `deque.removeIf(predicate)` (Java 8+)
