## Internal implementation

- Partition key (+ optional sort key) determines physical partition placement
- DynamoDB **hashes** the partition key value — you never manually assign a range of values to a partition (that's RDBMS-style manual sharding, not how DynamoDB works)
- Hash spread across partitions is automatic and dynamic based on key cardinality and access pattern

### Hot partitions

> [!danger] What causes one
> **Low cardinality** on the partition key (few distinct values — e.g. a date-bucket or status field) or **skewed access** on an otherwise fine key (a handful of "celebrity" items getting vastly more traffic than the rest). Not caused by uniqueness — caused by too few distinct values or uneven traffic across the values that exist.

### Partition key choice — order-tracking example

| Choice | Verdict | Why |
|---|---|---|
| `orderId` (random/unique) | Safe | High cardinality — every item hashes to a different value, spreads evenly |
| Date/quarter as key | Hot-partition risk | Only a handful of distinct values (e.g. 4 quarters) — nearly all traffic for a period hashes to the same partitions |

> [!tip] Mnemonic
> High-cardinality + uniform access = spread. Low-cardinality OR skewed access (even on a technically-unique key) = hot.

## When to use
_Not covered yet._

## How to use
_Not covered yet._

## When NOT to use
_Not covered yet._

---
_In progress: follow-up question on `customerId` as partition key + `orderId` as sort key (get-all-orders-for-customer access pattern) — does this introduce hot-partition risk, under what condition. Resume here._
