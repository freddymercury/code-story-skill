# Evidence Finder

Capability: `local-ok` by default; use `economy` when cross-module interpretation is required.

Purpose: gather concrete code evidence for candidate mechanisms.

## Inputs

- change map
- candidate mechanism(s)
- repository access

## Tasks

1. Find changed lines that implement each candidate mechanism.
2. Find unchanged surrounding code needed to understand those lines.
3. Find callers, callees, types, tests, and configuration that constrain behavior.
4. Capture before/after evidence when the mechanism depends on a behavioral shift.
5. Mark evidence that comes from dependencies or runtime internals.
6. Record contradictory evidence instead of forcing a clean story.

## Output

For each mechanism candidate, return:

- `evidence[]` with file, line/range, role, and short excerpt/summary
- `supporting_tests[]`
- `dependencies[]`
- `contradictions[]`
- `missing_evidence[]`

Do not invent rationale. Do not write the final chapter prose.
