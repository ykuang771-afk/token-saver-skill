# Luna implementation packet

Read this reference immediately before delegating implementation.

Send Luna a self-contained packet with these fields:

```markdown
## Objective
One observable implementation outcome.

## Main-model decisions
Approved requirements, architecture, interfaces, and tradeoffs. Luna must implement these decisions rather than reopen them.

## Relevant evidence
Repository facts, failing behavior, baseline state, and user changes that must be preserved.

## Writable scope
Exact files or modules this worker owns. State whether new files are allowed.

## Out of scope
Files, systems, decisions, and external actions the worker must not change.

## Constraints
Compatibility, safety, privacy, performance, style, dependency, and rollback requirements.

## Acceptance criteria
Observable behavior, required edge cases, and conditions that must remain unchanged.

## Required validation
Exact commands or deterministic checks, including what counts as a pass.

## Expected return
1. Summary of the implementation.
2. Exact files changed.
3. Commands and checks run, with results.
4. Remaining risks, uncertainty, or blockers.
```

Tell Luna to stop and return evidence instead of guessing when repository facts contradict the packet, scope must materially expand, an unapproved dependency or public-interface change is required, sensitive security or data-integrity decisions appear, or validation is unavailable.

For a correction packet, include only the failed acceptance criterion, direct evidence, the required correction, the writable scope, and the exact rerun command. Do not ask for a broad second review.
