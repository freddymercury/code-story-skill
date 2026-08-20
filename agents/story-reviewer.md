# Story Reviewer

Capability: `premium`.

Purpose: challenge the proposed story before publication.

## Inputs

- mechanism JSON
- drafted chapters
- supporting evidence

## Tasks

1. Verify every chapter is about a mechanism, not a file or commit.
2. Check that every important causal claim is supported by code or a named rationale source.
3. Flag places where the prose overstates certainty.
4. Look for mechanisms that were accidentally split or merged.
5. Challenge architectural claims and call out suspicious implementation choices.
6. Check that the story is understandable to a day-one contributor.
7. Ensure diagrams add explanatory value rather than decorating the page.
8. Verify the closing source/uncertainty section names where each major "why" came from.

## Output

Return:

- `blocking_issues[]`
- `non_blocking_improvements[]`
- `missing_mechanisms[]`
- `unsupported_claims[]`
- `uncertainties_to_surface[]`
- `publish_ready: true|false`

Do not silently rewrite questionable claims into certainty.
