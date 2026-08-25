# plan-triage

## Responsibilities

1. Count total active REQ-IDs across the plan.
2. Decide whether the plan remains single-file or uses the 3-core-file split.
3. Detect repetitive review/test patterns across phases.
4. If a reusable pattern is detected, create `review-audit-template.md` alongside the plan files.
5. Leave the plan in an execution-ready structure and set `RETRIAGE: NO` only after structural validation succeeds.

## Preconditions

Triage runs when:
- a new draft has `RETRIAGE: REQUIRED`;
- an update has set `RETRIAGE: REQUIRED`;
- the current plan structure is missing or malformed.

Do not clear the retriage state while unresolved structural conflicts remain.

## Decision: single file vs 3-core-file split

Keep the plan as a **single file** when ALL of the following are true:
- Total active REQ-IDs ≤ 30.
- Phases are cohesive and have clear boundaries.
- The plan covers a single feature or a tightly related set of changes.
- No phase contains more than 10 independent REQ-IDs that cannot be grouped into clear sub-phases.

Split into **3 core files** when ANY of the following is true:
- Total active REQ-IDs > 30.
- The plan spans multiple unrelated features or domains.
- A single phase contains more than 10 independent REQ-IDs that cannot be cleanly separated into sub-phases.
- The plan is expected to span multiple execution sessions and separating specification, implementation strategy, and task state materially reduces context drift.

The threshold is deterministic. Do not use ranges such as `25–30` as decision boundaries.

### 3-core-file structure

```text
spec.md    ← what and why: lifecycle state, objective, constraints, confirmed decisions, out-of-scope, exit criteria
plan.md    ← how: architecture, phases, dependencies, commit order
tasks.md   ← active and historical REQ-ID checklist with status and implementation evidence
```

These are the **three core plan files**. Optional supporting artifacts such as `review-audit-template.md` may exist alongside them.

When converting from a single file to the 3-core-file structure:
- Move both lifecycle fields to `spec.md`.
- `spec.md` becomes the only authoritative location for `ANALYSIS` and `RETRIAGE`.
- Do not duplicate lifecycle fields in `plan.md` or `tasks.md`.

### Sub-phase split

If a single phase is dense but the plan otherwise qualifies as single-file, split the phase with headings while keeping the canonical numeric REQ-ID grammar unchanged.

```markdown
## Phase 2 — [name]

### Phase 2A — [sub-objective A]
- [ ] REQ-2.1.1 ...
- [ ] REQ-2.1.2 ...

### Phase 2B — [sub-objective B]
- [ ] REQ-2.2.1 ...
- [ ] REQ-2.2.2 ...
```

Do not introduce alternate identifiers such as `REQ-2a.1.1`.

## Detecting a reusable review pattern

Scan the plan for sections that:
- Repeat identical or near-identical review/verification steps after multiple phases.
- Use the same independent-review criteria across phases.
- Repeat the same REQ-ID completion audit.

If the same pattern appears across at least 2 phases:
- Create `review-audit-template.md` alongside the plan files.
- Replace each duplicated review block with an explicit reference such as `[review-audit-template]`.
- Preserve phase-specific verification inline when it differs materially from the shared template.

If there is no meaningful repetition, do not create the file.

## `review-audit-template.md` format

```markdown
# review-audit-template

## Independent review
Review the completed phase against the following checks:
- Every active REQ-ID in the phase was implemented with no silent omission.
- No redundant, duplicated, or unused code was introduced by the phase.
- Targeted tests and required verification pass.
- No confirmed regression in pre-existing observable behaviour was introduced.

Fix confirmed findings and repeat the relevant verification before marking the phase COMPLETE.

## REQ-ID audit
For every active REQ-ID in the phase, preserve the requirement text and attach implementation evidence:

- [x] REQ-N.M.n [original requirement text]
  - Evidence: `path/to/file.ext:line-range`

If any active REQ-ID is incomplete or lacks required evidence, the phase cannot be marked COMPLETE.
```

## Finalise lifecycle state

After structure and review-pattern validation succeed:

- Set `RETRIAGE: NO` in the authoritative lifecycle location.
- Preserve the current `ANALYSIS` value; triage does not decide semantic completeness.
- If `ANALYSIS: IN_PROGRESS`, the plan remains non-executable even though structural triage is complete.

Execution is allowed only when both are true:

```text
ANALYSIS: COMPLETE
RETRIAGE: NO
```

## Output

After triage, record or report:
- Structure decision and reason.
- Total active REQ-ID count.
- Whether `review-audit-template.md` was created and why.
- Any phases split into sub-phases.
- Final `RETRIAGE` state.
- Whether the plan is ready for execution.

Do not require an extra user confirmation when the structure follows already-confirmed plan decisions and introduces no semantic change. Ask only when triage itself exposes a real structural or requirements conflict.
