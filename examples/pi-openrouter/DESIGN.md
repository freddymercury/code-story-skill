# Pi + Ollama + OpenRouter routing design

This is a design-level input used to dogfood the `code-story` skill. It is not yet an implementation diff.

## Goal

Let a Pi coding agent spawn child/sub-agents without pinning every worker to one model. Local Ollama should handle mundane, latency-tolerant work. Hosted models should handle interactive or reasoning-heavy work. Model choice should remain automatic by default.

## Verified host capabilities

- `pi-model-router` exposes three logical tiers: `low`, `medium`, and `high`, with optional LLM classification, custom rules, budget limits, phase bias, and fallback chains.
- `pi-orchestration` supports sub-agent roles and `model: "auto"`, inheriting the parent Pi model registry.
- OpenRouter separates model selection from provider selection. Provider failover is automatic; model fallback can be configured separately.

## Proposed execution classes

| Work shape | Route |
|---|---|
| mundane + latency-tolerant/background | local Ollama |
| mundane + interactive/fast response required | small hosted fast model |
| substantial + cost-sensitive/non-urgent | economy hosted model through OpenRouter |
| difficult/ambiguous/correctness-sensitive | premium frontier model through OpenRouter |

The three `pi-model-router` tiers map naturally to:

- `low` -> local Ollama
- `medium` -> economy hosted model
- `high` -> premium frontier model

The fourth case, mundane + interactive, should be handled before the three-tier complexity router when invocation metadata says a human is synchronously waiting. The concrete hosted fast model is a router/config concern, not a worker-role property.

## Agent policy

Sub-agents should normally request `model: "auto"` or a capability hint, not a named model.

Examples:

- change mapper / file inventory -> `local-ok`
- normal implementation / mechanism discovery -> `economy`
- architectural synthesis / adversarial review -> `premium`

The routing layer maps those hints to concrete models.

## OpenRouter policy

Use OpenRouter only for hosted routes. Keep Ollama direct/local.

OpenRouter responsibilities:

1. provide a unified hosted-model API
2. choose among providers serving the selected model
3. fail over between providers automatically
4. optionally fail over to backup models
5. expose cost/latency/provider controls

Pi remains responsible for deciding which model class should answer the task.

## Local machine constraint

Target machine: 16-inch M3 Pro MacBook Pro, 18 GB unified memory.

Operational policy:

- prefer a small local coding model for routine/background work
- keep local concurrency conservative (one substantial local generation at a time)
- do not make 30B+ local models a design dependency
- move reasoning-heavy work to hosted economy/premium routes

## Proposed flow

```text
sub-agent task
    |
    v
invocation context available?
    |
    +-- mundane + interactive --> fast hosted small model
    |
    +-- otherwise --> pi-model-router
                       |
                       +-- low ----> Ollama local
                       +-- medium -> OpenRouter economy model
                       +-- high ---> OpenRouter premium model
                                      |
                                      +-> provider routing/failover
```

## Open questions

- exact Pi extension point for passing interactive/background invocation metadata into routing
- whether capability hints should be encoded as task metadata, role metadata, or router rules
- exact local model to use on 18 GB unified memory
- exact economy and premium hosted model aliases
- privacy-sensitive override behavior
- escalation policy after a weak/failed answer
- observability: log chosen tier, model, provider, latency, tokens, cost, and escalation reason

## Sources used to verify host behavior

- https://github.com/yeliu84/pi-model-router
- https://github.com/0xKobold/pi-orchestration
- https://openrouter.ai/blog/insights/model-routing/
