# code-story-skill

A portable, orchestrated fork of Will Ness's `code-story` skill for explaining code changes as mechanism-first interactive stories.

This fork keeps the original core idea — **one chapter per mechanism, not per file or commit** — and adapts the workflow for agent orchestration and automatic model routing.

## Pull it into another harness

Clone or pull the repo normally:

```bash
git clone https://github.com/freddymercury/code-story-skill.git
# later
git -C code-story-skill pull --ff-only
```

Then point the harness's skill loader at `SKILL.md`, or copy/symlink the repo into that harness's normal skills directory.

The skill deliberately does **not** depend on Pi, OpenRouter, Ollama, or any specific model name. A harness only needs to understand the workflow described in `SKILL.md` and provide whatever repo/file/sub-agent tools it has available.

## What changed from upstream

- Splits investigation into focused sub-agent roles.
- Uses capability hints instead of hard-coding model names.
- Produces a structured mechanism handoff before final authoring.
- Removes assumptions about a specific Artifact/scratchpad host.
- Writes output to `.code-story/<change-id>/index.html` by default.
- Keeps the final synthesis/review step on a higher-capability path while allowing mechanical work to run locally or cheaply.
- Requires generated stories to include a persistent Light/Dark mode toggle while respecting the OS preference initially.

## Harness-neutral routing

The skill uses three capability hints:

| Work | Capability hint |
|---|---|
| Diff inventory, symbol extraction, snippet collection, HTML assembly | `local-ok` |
| Mechanism discovery, rationale research, moderate reasoning | `economy` |
| Final teaching narrative, architecture interpretation, adversarial review | `premium` |

These are **not model names**.

A harness with local inference can map `local-ok` to Ollama, LM Studio, llama.cpp, MLX, or whatever local provider it already exposes. A harness without local inference should map `local-ok` to its cheapest suitable hosted worker instead. `economy` and `premium` should likewise map to that harness's own model catalog.

Recommended behavior:

```text
local-ok
  ├─ local model available + task fits → local
  └─ otherwise                         → cheap hosted

economy  → economical capable hosted/local model
premium  → strongest reliable model available
```

Do not make agent identity equal model identity. `evidence-finder` should not permanently mean one particular model; the harness should choose the execution target per task.

For small-memory local machines, keep local concurrency conservative until measured. If the harness already has its own routing or concurrency policy, defer to it.

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
