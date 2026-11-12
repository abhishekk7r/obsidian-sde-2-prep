# HashMap & equals / hashCode

---
tags: [java, collections, hashmap, amex]
---

## HashMap internals

### put flow
- `hashCode()` then spread: `h ^ (h >>> 16)`
- index = `(n - 1) & hash`
- empty bucket: insert node
- equal key (`equals()` true): **replace** value
- different key, same bucket: append to chain (collision)

> [!note] Why `&` and not `%`
> capacity always power of two, so `(n-1) & hash` == `hash % n`, cheaper
> spread step needed because `&` only sees low bits

### Two separate alarms

| Alarm | Watches | Action |
|---|---|---|
| Load factor | total size > capacity x 0.75 | double table, rehash all |
| Long chain | 9th node added to one bucket | capacity < 64: **resize**. capacity >= 64: **treeify** that bucket |

> [!warning] Trap
> chain alarm and load factor are independent
> cap 16, 9 keys in one bucket: size 9 < 12, load factor silent, resize still happens (chain alarm)
> same hashCode for all keys: resizing never splits chain, resizes repeat until cap 64, then tree

- tree bucket untreeifies at <= 6 nodes
- tree: O(log n) lookup vs O(n) list

> [!tip] Mnemonic
> 0.75 grows the table. 8 grows a tree, but only from 64 up.

### Interview cards
- Q: same key put twice? A: value replaced, no new node
- Q: index formula? A: `(n-1) & hash`, why: power of two + spread
- Q: resize vs treeify? A: load factor resizes, long chain treeifies only at cap >= 64, else resizes
- Q: cap 16, 9 colliding keys? A: resize to 32, no tree
- Q: why bad hashCode hurts even in Java 8? A: chains stay long, tree is O(log n) not O(1)

---

## HashMap Internals — Buckets, Resize, Treeification

- `put()`: key → `hashCode()` → HashMap spreads it (`hash ^ (hash >>> 16)`) to mix high bits into low bits → bucket index = `hash & (capacity - 1)`
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

---

## equals() / hashCode() + String Immutability

> [!important] The one rule
> `equals() == true` → `hashCode()` **must** match (mandatory)
> `hashCode()` match → `equals()` **may** be false (collision — legal, just slower)

> [!warning] Trap 1 — override equals() only
> **Symptom:** `HashSet` allows "duplicate" equal elements; `contains()`/`get()` silently fail
> **Why:** equal objects → different default hashCode → different buckets → never compared
> **Fix:** always override both. `Objects.hash(fields)` or IDE/Lombok

> [!warning] Trap 2 — mutate a field used in equals()/hashCode() after using object as a key
> **Symptom:** `get(key)` → null, even though entry still shows in iteration
> **Why:** entry sits in the bucket from insert-time hash; mutation changes the hash → `get()` searches the wrong bucket
> **Fix:** never mutate key-identity fields post-insert. Prefer immutable keys

> [!note] Trap 3 — collision ≠ contract violation
> Unequal objects **can** share hashCode (legal collision, O(1)→O(n))
> Equal objects **must always** share hashCode (mandatory)

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

> [!tip] Why immutability enables pooling
> Pooled strings are shared **by reference** JVM-wide. If mutable, changing one ref would corrupt every holder. Immutability = that can never happen → `StringBuilder` is deliberately mutable, so it can't be pooled.
> Bonus: immutable → `hashCode()` cached after first call → immune to Trap 2 → ideal HashMap key.
