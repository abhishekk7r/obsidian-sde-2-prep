---
tags: [java, collections, hashmap, amex]
---

# HashMap internals

## put flow
- `hashCode()` then spread: `h ^ (h >>> 16)`
- index = `(n - 1) & hash`
- empty bucket: insert node
- equal key (`equals()` true): **replace** value
- different key, same bucket: append to chain (collision)

> [!note] Why `&` and not `%`
> capacity always power of two, so `(n-1) & hash` == `hash % n`, cheaper
> spread step needed because `&` only sees low bits

## Two separate alarms

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

## Interview cards
- Q: same key put twice? A: value replaced, no new node
- Q: index formula? A: `(n-1) & hash`, why: power of two + spread
- Q: resize vs treeify? A: load factor resizes, long chain treeifies only at cap >= 64, else resizes
- Q: cap 16, 9 colliding keys? A: resize to 32, no tree
- Q: why bad hashCode hurts even in Java 8? A: chains stay long, tree is O(log n) not O(1)
