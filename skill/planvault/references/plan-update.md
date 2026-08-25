# plan-update

## Responsibilities

1. Identify the exact REQ-IDs or sections affected by the requested change.
2. Detect conflicts between the new instruction and existing plan content.
3. Apply the minimum semantic diff — change only what the request or necessary dependency requires.
4. Preserve all unaffected REQ-IDs and status markers.
5. Update the `ANALYSIS` state when the change invalidates or reopens confirmed reasoning.
6. Set `RETRIAGE` explicitly according to whether the update invalidates current structural validation.

## Lifecycle fields

Every plan must expose:

```text
ANALYSIS: IN_PROGRESS | COMPLETE
RETRIAGE: REQUIRED | NO
```

For a single-file plan, both fields live at the top of the plan file.

For a 3-core-file plan, both fields live only in `spec.md`, which is authoritative for lifecycle state.

`plan-update` may set `RETRIAGE: REQUIRED` when an update invalidates the current structural decision. It must never set `RETRIAGE: NO`; only `plan-triage` may clear the retriage requirement after validating structure.

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

After applying a non-conflicting update, determine whether the current structural decision remains valid.

Set `RETRIAGE: REQUIRED` when any of the following is true:

- Active REQ-ID count crosses the triage threshold.
- A phase grows beyond the supported density and cannot be cleanly sub-phased in place.
- The update adds or removes a materially separate feature/domain.
- The update changes whether repeated review logic should be shared.
- The update changes dependencies or phase boundaries enough that the current split is no longer coherent.
- A single-file plan may now require the 3-core-file structure, or a 3-core-file plan may now be unnecessarily split.
- The authoritative lifecycle fields are missing, duplicated, or stored in the wrong file and structural normalization is required.

If the plan already has `RETRIAGE: REQUIRED`, preserve it until `plan-triage` clears it.

If none of the structural conditions applies and the current state is already `RETRIAGE: NO`, preserve `RETRIAGE: NO`; do not force triage for a small update.

When `RETRIAGE: REQUIRED` is present, execution must not continue until `plan-triage` has run and set `RETRIAGE: NO`.

## Lifecycle examples

Small implementation-detail update:

```text
before:
ANALYSIS: COMPLETE
RETRIAGE: NO

after:
ANALYSIS: COMPLETE
RETRIAGE: NO
```

Architectural update that also changes plan structure:

```text
before:
ANALYSIS: COMPLETE
RETRIAGE: NO

after:
ANALYSIS: IN_PROGRESS
RETRIAGE: REQUIRED
```

Structure-only change with semantic decisions still valid:

```text
before:
ANALYSIS: COMPLETE
RETRIAGE: NO

after:
ANALYSIS: COMPLETE
RETRIAGE: REQUIRED
```

## What not to do

- Do not silently resolve conflicts.
- Do not add unrelated requirements.
- Do not restructure unaffected sections.
- Do not renumber REQ-IDs after additions or removals.
- Do not force a re-triage for small updates that leave the current structure valid.
- Do not set `RETRIAGE: NO`; structural clearance belongs exclusively to `plan-triage`.
