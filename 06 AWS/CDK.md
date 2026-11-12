## Internal implementation

- `cdk synth` — compiles CDK code (TypeScript/Python/Java) into an actual **CloudFormation template** (JSON/YAML) — this is the real artifact CDK produces
- `cdk deploy` — runs synth, then hands the template to the **CloudFormation service**, which is the actual orchestration engine

> [!tip] CDK is a code-generation layer, not a separate deployment engine
> CloudFormation does all the real work: diffs against current stack state, builds a change set, executes it, owns rollback/state tracking, talks to individual AWS service APIs. CDK's only job is producing the template.

## When to use — CDK vs raw CloudFormation vs Terraform

| | CDK | Raw CloudFormation | Terraform |
|---|---|---|---|
| Language | Real programming language (TS/Python/Java) — loops, conditionals, type-checking | Declarative YAML/JSON | HCL DSL |
| Cloud scope | AWS-only | AWS-only | Multi-cloud (AWS/GCP/Azure/...) |
| Backing engine / state | CloudFormation (inherits its state model, drift detection, rollback) | CloudFormation directly | Terraform's own separate state file/provider model |
| Abstraction | L1/L2/L3 constructs — one call can generate multiple resources + sensible defaults | None — every resource hand-written | Modules (community/custom) |

- **Choose CDK:** all-in on AWS, want real-language abstractions/type safety/reusable constructs
- **Choose Terraform:** multi-cloud requirement, or org already standardized on it across teams

## How to use

- A `Stack` class extends `cdk.Stack`; inside its constructor, instantiate constructs — e.g. `new lambda.Function(this, 'MyFn', {...})`
- An `App` (`app.ts`/`app.py`) aggregates one or more stacks
- `cdk synth` / `cdk deploy` turn that into CloudFormation and ship it

## When NOT to use

- **Multi-cloud requirement** — CDK is AWS-only; Terraform is the right tool
- **Very simple/static infra needing easy audit** — raw CloudFormation has no hidden abstraction layer; reviewers can read exact resources without reasoning through generated code
- **Org already standardized on Terraform** across many teams/clouds — introducing CDK for one AWS-only service breaks tooling consistency
- **Need fine-grained transparency over exact template output** — L2/L3 constructs impose opinionated defaults that can complicate debugging when precise resource properties matter
