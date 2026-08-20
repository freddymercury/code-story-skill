# code-story-skill

A portable, orchestrated fork of Will Ness's `code-story` skill for explaining code changes as mechanism-first interactive stories.

This fork keeps the original core idea — **one chapter per mechanism, not per file or commit** — and adapts the workflow for agent orchestration and automatic model routing.

## What changed from upstream

- Splits investigation into focused sub-agent roles.
- Uses capability hints instead of hard-coding model names.
- Produces a structured mechanism handoff before final authoring.
- Removes assumptions about a specific Artifact/scratchpad host.
- Writes output to `.code-story/<change-id>/index.html` by default.
- Keeps the final synthesis/review step on a higher-capability path while allowing mechanical work to run locally or cheaply.

## Suggested routing

| Work | Capability hint |
|---|---|
| Diff inventory, symbol extraction, snippet collection, HTML assembly | `local-ok` |
| Mechanism discovery, rationale research, moderate reasoning | `economy` |
| Final teaching narrative, architecture interpretation, adversarial review | `premium` |

Your router decides the concrete provider/model. On a small-memory local machine, keep local concurrency conservative and let hosted models absorb heavier reasoning.

## Dogfood example

The repo includes a full run of the skill against the actual **`freddymercury/pi-model-orchestrator`** design and implementation:

- `examples/pi-model-orchestrator/mechanisms.json` — mechanism-first intermediate representation
- `examples/pi-model-orchestrator/index.html` — rendered five-chapter story
- `examples/pi-model-orchestrator/review.json` — adversarial reviewer output

This example is grounded in the real orchestrator repo and deliberately surfaces unresolved implementation questions instead of inventing answers.

## Layout

```text
SKILL.md
agents/
  change-mapper.md
  evidence-finder.md
  rationale-researcher.md
  story-reviewer.md
references/
  template.html
schemas/
  mechanism.json
examples/
  pi-model-orchestrator/
    mechanisms.json
    index.html
    review.json
```

## Upstream

Adapted from `will-ness-ai/skills/skills/code-story` under the MIT License. See `NOTICE.md` and `LICENSE`.
