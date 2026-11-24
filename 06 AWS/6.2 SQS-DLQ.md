## Internal implementation

![[sqs-visibility-timeout-dlq.svg]]

- Consumer receives a message — SQS makes it **invisible** to other consumers for `visibilityTimeout` (default 30s), a lease, not an immediate delete
- Consumer finishes successfully → calls `DeleteMessage` → gone for good
- Consumer crashes / never deletes → timeout expires → message becomes visible again → any consumer can pick it up on next poll
- No crash-detection mechanism — purely a timeout-based lease expiring silently

### DLQ — direction matters

> [!danger] Trap — retries happen on the main queue, NOT the DLQ
> A message does not "go to the DLQ for retries." Retries happen via the visibility-timeout-expiry loop on the **main queue**. The DLQ is where a message lands **after** retries are exhausted — a quarantine/dead-end, not a retry mechanism.

- **`maxReceiveCount`** — redrive policy setting: after a message has been received this many times without deletion, SQS moves it out of the main queue into the DLQ
- Purpose: stop a permanently-broken ("poison") message from endlessly recycling and wasting consumer capacity
- Set too low → healthy-but-slow messages prematurely exiled; too high → poison messages burn retries too long before quarantine

**Mnemonic:** *Visibility timeout = lease, not delete. DLQ = quarantine after exhausted retries, not a retry queue itself.*

## When to use

- **Decouple** producer from a slower/unreliable downstream consumer — producer pushes and moves on, consumer pulls at its own pace, absorbing spikes as backpressure
- Retry-with-backoff semantics + dead-letter safety net for a downstream step that may keep failing
- Own project: exactly this shape — decoupling + retry/poison-message handling via SQS+DLQ in production

> [!tip] Don't conflate with SNS
> "Push a notification, consumers react" is closer to SNS pub/sub language. SQS is point-to-point polling, not broadcast — see "when not to use" below.

## How to use

- Configure main queue + separate DLQ, link via redrive policy: `maxReceiveCount` on the main queue points to the DLQ's ARN
- Consumer polls main queue → processes → deletes on success; on repeated failure past `maxReceiveCount`, SQS auto-moves the message to the DLQ

## When NOT to use

- **Needs an immediate synchronous response within the request lifecycle** — a client waiting on an HTTP response can't wait on queue polling latency; that's a direct call, not a queue
- **Needs broadcast/fan-out to multiple independent consumers** — SQS is point-to-point (one message, one consumer). For "multiple independent systems all need to react to the same event" (e.g. order-placed → inventory + notifications + analytics), use **SNS fan-out** (or SNS→multiple SQS queues) instead
