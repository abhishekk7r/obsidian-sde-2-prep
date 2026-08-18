## Internal implementation

- Managed infra to run foundation models — no hosting/GPU provisioning yourself
- **The real differentiator vs. calling a provider's API directly:** one unified API across multiple model providers (Anthropic, Meta Llama, Cohere, Mistral, Amazon Titan) — no separate SDK/credentials per vendor
- Stays inside your **AWS account's security boundary**: IAM-based auth (not a separate API key to manage/rotate), traffic stays within your VPC, your data isn't used to train the underlying models

> [!tip] "Managed infra" alone isn't the pitch
> OpenAI's API is also managed infra you don't host — that description alone doesn't distinguish Bedrock. The actual differentiator is the unified multi-vendor API + staying inside your AWS security/compliance boundary.

## When to use

- Invoking a model **from an already-AWS-resident service** — same IAM role, same VPC, no new outbound network path or separate vendor credential to provision/rotate
- If a service already runs in AWS with an IAM role, Bedrock is a same-ecosystem call; a direct third-party API is a new external dependency with its own auth/network-egress/compliance surface

## How to use

- AWS SDK `bedrock-runtime` client
- `InvokeModel` (or `InvokeModelWithResponseStream` for streaming) with a model ID (e.g. `anthropic.claude-*`) and a JSON request body
- Auth flows through the calling service's IAM role, not a hardcoded API key

## When NOT to use

- Need full control over a custom/self-hosted model AWS doesn't offer
- A cheaper alternative genuinely fits the use case better
- Need a provider Bedrock doesn't support, or need to operate outside AWS's compliance boundary entirely

> [!danger] Trap — data privacy is NOT a reason to avoid Bedrock
> Backwards instinct to watch for: data privacy/compliance concerns are typically a reason **to choose** Bedrock over calling a third-party model API directly (data stays inside your AWS boundary, isn't used for provider-side training) — not a reason to avoid it.
