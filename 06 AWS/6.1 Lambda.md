## Internal implementation
_Not covered yet._

## When to use
- Event-driven reaction to another AWS service — S3 upload trigger, DynamoDB stream, API Gateway request
- Scheduled/cron-style work — Lambda + EventBridge scheduled rules, no server idling between runs, pay only for execution seconds
- Async queue processing — SQS/DLQ event-source mapping, Lambda polls and spins up only when messages arrive, concurrency scales with queue depth

> [!tip] Common thread
> Something else owns the triggering and the state — Lambda just reacts, runs, and exits. Opposite shape from a component that must persistently sit in a traffic path.

## How to use
_Not covered yet._

## When NOT to use
- Anything that needs to **persistently sit in a hot traffic path** intercepting every request (e.g. a proxy/routing layer) — cold starts introduce latency variance you can't tolerate on every request
- No persistent connection/state between invocations — wrong shape for components needing continuous state
- Scenarios needing **immediate, stateful rollback** — Lambda's invocation model isn't built for an instant state-flip
