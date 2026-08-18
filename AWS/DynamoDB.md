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
_Not covered yet._

## How to use
_Not covered yet._

## When NOT to use
_Not covered yet._
