---
name: code-story
description: Build a mechanism-first HTML walkthrough that teaches how and why a code change works.
disable-model-invocation: true
---

`/code-story <request>` accepts a PR, diff, branch, commit, or change description plus anything the human says they do not understand.

A **code story** explains a change as a sequence of mechanisms. Each chapter answers one question, and its title is that question. A chapter may span several files when those files implement the same idea.

**Non-negotiable rule:** one chapter per mechanism, not per file and not per commit.

Write for a competent day-one contributor who knows nothing about this repository yet. Explain why the change works, where the reasoning came from, what remains uncertain, and where the implementation appears questionable.

## Operating model

This fork is orchestration-first. The coordinating agent should delegate evidence-heavy work to focused workers, collect structured findings, and reserve premium reasoning for synthesis and review.

Do **not** hard-code concrete model names in this skill. Use these capability hints instead:

- `local-ok` — deterministic, repetitive, mechanical, extraction-heavy, or latency-tolerant work.
- `economy` — moderate reasoning, mechanism discovery, codebase context, and rationale research.
- `premium` — architecture interpretation, ambiguous reasoning, final teaching narrative, and adversarial review.

The host/router maps capability hints to actual models. If routing metadata is unsupported, treat them as prioritization guidance rather than required syntax.

## Workflow

### 1. Establish the change boundary

Identify the exact target: PR, diff, branch range, commit, or design description. Gather the changed-file inventory, additions/deletions, relevant metadata, or design boundary.

Delegate to [`agents/change-mapper.md`](agents/change-mapper.md) with capability `local-ok` unless the change boundary itself is ambiguous.

The mapper should return:

- changed files and hunks, or design components if no code exists yet
- important symbols added/removed/changed
- likely cross-file or cross-component relationships
- obvious mechanical/noise-only changes
- candidate mechanisms, clearly marked as hypotheses rather than conclusions

### 2. Investigate evidence beyond the diff

A useful story cannot stop at changed lines. Read what explains the change:

- PR body and linked issue
- review comments and author notes
- code before the change
- unchanged callers/callees and surrounding modules
- tests that demonstrate intended behavior
- third-party or runtime code the change relies on, when relevant
- for design-only stories, authoritative docs for the systems being composed

Use [`agents/evidence-finder.md`](agents/evidence-finder.md) for code evidence (`local-ok` or `economy`) and [`agents/rationale-researcher.md`](agents/rationale-researcher.md) for documented reasoning (`economy`).

Never invent rationale. If no source explains a design choice, record that explicitly.

### 3. Build mechanisms, not file summaries

Merge worker findings into mechanism objects that follow [`schemas/mechanism.json`](schemas/mechanism.json).

A mechanism should answer a question such as:

- Why does authentication now happen before routing?
- How is identity carried through the request?
- Why did the database lookup disappear?

Avoid chapter plans such as `auth.ts`, `router.ts`, or `schema.ts`.

Each mechanism needs:

- a reader-facing question
- a concise explanation
- supporting code/design evidence
- source(s) for the "why"
- uncertainty or contradictions
- risk/watch-out notes where warranted

### 4. Author the story

Read [`examples/pi-openrouter/index.html`](examples/pi-openrouter/index.html) as the local finished example. It demonstrates the chapter shape, source/uncertainty treatment, diagram restraint, and how to handle a design story with no implementation diff yet.

Copy [`references/template.html`](references/template.html) to the output location and write one `<article>` per mechanism into `<main>`.

Use capability `premium` for final authoring when the change is genuinely complex or correctness-sensitive. Straightforward stories may use `economy` if the evidence package is already strong.

Writing rules:

- short, plain sentences
- teach the mechanism before naming abstractions unnecessarily
- pin claims to concrete lines when possible
- show unchanged code when it is essential to understanding the mechanism
- for design stories, clearly label proposed behavior versus verified host behavior
- use diagrams only when they reveal structure the code does not
- call out code/design choices that appear wrong or fragile instead of rationalizing them
- include a final section explaining where each important "why" came from
- include unresolved questions rather than hiding them

### 5. Review the story adversarially

Run [`agents/story-reviewer.md`](agents/story-reviewer.md) with capability `premium` for important or complex changes.

The reviewer should challenge:

- invented rationale
- file-by-file organization disguised as mechanisms
- claims not supported by cited code/docs
- missing contradictory evidence
- misleading simplifications
- omitted risks or questionable implementation details
- chapters that assume repository knowledge a new contributor would not have

Revise only where the reviewer identifies concrete issues.

### 6. Publish portably

Do not assume an Artifact tool or scratchpad exists.

Default output path:

```text
.code-story/<change-id>/index.html
```

If the host has a richer publishing mechanism, it may additionally publish the same file there, but the local HTML artifact remains the portable source of truth.

## Concurrency guidance

Mechanical workers may run in parallel when they inspect independent files or evidence sources. Do not spawn multiple heavyweight local models merely because parallelism is available; respect host memory limits and router concurrency policy.

For small-memory local machines, prefer one substantial local worker at a time and move reasoning-heavy work to hosted `economy`/`premium` routes.

## Failure modes to avoid

- forcing every worker onto one named model
- using local solely because a task is easy even when an interactive caller needs low latency
- sending all repository reading to a premium model
- treating the diff as the only source of truth
- turning filenames into chapter titles
- silently filling gaps in rationale
- polishing an explanation before the evidence package is complete
