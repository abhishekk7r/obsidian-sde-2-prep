## Internal implementation

- Partition key (+ optional sort key) determines physical partition placement
- DynamoDB **hashes** the partition key value — you never manually assign a range of values to a partition (that's RDBMS-style manual sharding, not how DynamoDB works)
- Hash spread across partitions is automatic and dynamic based on key cardinality and access pattern

### Partition key vs sort key (composite primary key)

![[dynamodb-composite-key.svg]]

| | Partition key | Sort key |
|---|---|---|
| Determines physical partition? | Yes — hashed | No — doesn't affect partition placement |
| Role | Groups items together on one partition | Orders items *within* that partition key's group |
| Uniqueness | Alone, not required to be unique when a sort key exists | (partition key, sort key) pair together is unique |

- Enables `Query` (not `Scan`) by partition key value alone → gets the whole item collection efficiently
- Sort key supports range conditions within that collection: `begins_with`, `between`, `>`, etc. — e.g. "this customer's orders after a given date"

### Hot partitions

> [!danger] Two separate causes
> **Low cardinality** — few distinct partition key values (e.g. a date-bucket or status field) — nearly all traffic hashes to the same few partitions.
> **Skewed access** — key has plenty of distinct values overall, but usage is uneven across them (a "celebrity" item/customer gets disproportionate traffic). Not a cardinality problem — a distribution problem.

### Partition key choice — order-tracking example

| Choice | Verdict | Why |
|---|---|---|
| `orderId` (random/unique) as partition key | Safe | High cardinality — every item hashes to a different value, spreads evenly |
| Date/quarter as partition key | Hot-partition risk (low cardinality) | Only a handful of distinct values — nearly all traffic for a period hashes to the same partitions |
| `customerId` (partition key) + `orderId` (sort key) | Hot-partition risk **if** a customer has disproportionately many orders | Skew, not cardinality — that customer's entire order collection piles onto one partition while typical customers barely touch theirs |

> [!tip] Mnemonic
> High-cardinality + uniform access = spread. Low cardinality OR skewed access (even on a technically-fine key) = hot. Two different causes, same symptom.

## When to use
> [!warning] The core design principle
> Design the table around **known access patterns upfront** — the opposite of RDBMS normalization + arbitrary querying. If you can't predict how you'll query it, DynamoDB is the wrong tool.

- Simple, predictable access patterns (get-by-key, get-by-key-range) needing single-digit-millisecond latency at massive, elastic scale
- Flexible/schema-less data
- Not for complex/ad-hoc/analytical queries — no joins, no arbitrary `WHERE` across attributes without a matching index (that's Postgres/Redshift territory)

> [!tip] "DynamoDB is more scalable than Postgres" — imprecise framing
> Not a categorical NoSQL-beats-SQL claim. Postgres can scale a long way (read replicas, partitioning, connection pooling) — it just takes real operational engineering. DynamoDB's actual edge: horizontal partitioning + elasticity are **fully managed**, no manual sharding/replica topology required. Trade query flexibility for operationally-free scale, not "more scalable" in the abstract.

## How to use
- Conceptual fluency is what's tested, not exact SDK syntax
- Example: `Query` with `KeyConditionExpression` on `customerId` (partition key), sorted by `orderId` (sort key) via `ScanIndexForward=false` for most-recent-first

## When NOT to use
- Need **JOINs across entities**
- Need **ad-hoc queries** not known in advance
- Need **multi-item ACID transactions** spanning many items
- All three trace back to the same root cause: DynamoDB requires upfront access-pattern design; RDBMS supports normalized schema + arbitrary querying
