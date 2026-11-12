# Indexing

---
tags: [database, indexing]
---

> [!note] Confidence check
> Only "SQL query writing" is confirmed-likely (from another candidate's account). Clustered/non-clustered indexing is a *reasonable guess*, not confirmed — know the basics, don't over-invest.

## Indexes — the short version

- **Index** = sorted structure (B+ tree) so the DB finds rows in O(log n) instead of scanning everything.
- **Clustered index** = the table's rows ARE stored in this sorted order. Like a **phone book** — entries sorted, data and order are the same thing. **Only 1 per table** (data can only be physically sorted one way). Usually the primary key.
- **Non-clustered index** = a separate sorted list of key → pointer to the real row. Like a **book's back index** — "topic → page number." Table itself stays untouched. **Many allowed** per table.
- **Downside of indexes** (the judgment question — say this unprompted): every index speeds up reads but **slows down writes** (has to be updated on every insert/update) and costs storage. Don't index everything.
- **Covering index** = a non-clustered index that already has every column the query needs → no extra trip to the table.

> Mnemonic: **Clustered = phone book (IS the order). Non-clustered = back-of-book index (POINTS to the order).**
