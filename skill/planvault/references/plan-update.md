# plan-update

## Responsibilities

1. Identify the exact REQ-IDs or sections affected by the requested change.
2. Detect conflicts between the new instruction and existing plan content.
3. Apply the minimum semantic diff — change only what the request or necessary dependency requires.
4. Preserve all unaffected REQ-IDs and status markers.
5. Update the `ANALYSIS` state when the change invalidates or reopens confirmed reasoning.
6. Flag structural re-triage when the update materially changes plan size, boundaries, or review structure.

## Conflict detection

Before applying a change, check for content that contradicts the new instruction:

- The same REQ-ID or confirmed decision is addressed differently elsewhere in the plan.
- A confirmed constraint blocks the requested change.
- An out-of-scope declaration excludes the new requirement.
- `spec.md`, `plan.md`, and `tasks.md` disagree in a 3-core-file plan.
- The requested update depends on an assumption that conflicts with current authoritative evidence.

If a real conflict is found:

- Stop before applying the conflicting semantic change.
- Report the specific conflicting items and where they appear.
- Resolve the conflict from an explicit user instruction or new authoritative evidence.
- Do not silently choose one side.

## Update rules

- **Minimum diff**: modify only affected requirements, decisions, constraints, dependencies, or structure.
- **No cosmetic rewrite**: do not rewrite unrelated narrative sections merely to improve wording.
- **REQ-ID continuity**: never renumber existing REQ-IDs.
- **Removed requirements**: preserve the historical ID and mark the requirement as removed instead of deleting it silently.
- **New requirements**: assign the next available ID within the relevant phase/section namespace.
- **Completed requirement changes**: if a completed `[x]` requirement changes materially, reset it to `[ ]` and mark it as reopened while preserving its identity.
- **Evidence preservation**: when reopening a completed requirement, retain previous evidence as historical evidence if useful, but do not present it as proof of the new requirement state.
- **ANALYSIS state**: set `ANALYSIS: IN_PROGRESS` when the change reopens a material architectural decision, introduces contradictory evidence, or invalidates a confirmed assumption. Pure execution-detail updates do not automatically reopen analysis.

## Structural re-triage

After applying a non-conflicting update, determine whether plan structure may no longer be appropriate.

Set or report `RETRIAGE: REQUIRED` when any of the following is true:

- Active REQ-ID count crosses the triage threshold.
- A phase grows beyond the supported density and cannot be cleanly sub-phased in place.
- The update adds or removes a materially separate feature/domain.
- The update changes whether repeated review logic should be shared.
- The update changes dependencies or phase boundaries enough that the current split is no longer coherent.

When `RETRIAGE: REQUIRED` is set, execution must not continue until `plan-triage` has run again.

If none of these conditions applies, do not run triage again unnecessarily.

## What not to do

- Do not silently resolve conflicts.
- Do not add unrelated requirements.
- Do not restructure unaffected sections.
- Do not renumber REQ-IDs after additions or removals.
- Do not force a re-triage for small updates that leave the current structure valid.
