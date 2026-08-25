# plan-execute

## Responsibilities

1. Validate lifecycle state and plan structure before execution.
2. Execute the requested phase, task, or full plan from its REQ-IDs.
3. Preserve requirement text and update completion state with implementation evidence.
4. Apply shared or phase-specific review checks when required.
5. Stop and return to plan-update if current evidence invalidates a confirmed plan decision.

---

## Step 0 — Lifecycle and structure validation

Execution is allowed only when the authoritative lifecycle state is exactly:

```text
ANALYSIS: COMPLETE
RETRIAGE: NO
```

### Single-file plan

Require:
- One plan file containing both lifecycle fields at the top.
- `ANALYSIS: COMPLETE`.
- `RETRIAGE: NO`.
- REQ-IDs with `[ ]` / `[x]` status markers.
- Clear phase structure.

### 3-core-file plan

Require the three core files:

```text
spec.md
plan.md
tasks.md
```

`spec.md` must contain the authoritative lifecycle fields:

```text
ANALYSIS: COMPLETE
RETRIAGE: NO
```

`plan.md` and `tasks.md` must not define conflicting lifecycle values.
`tasks.md` must contain the REQ-ID checklist and status markers.

Optional support files such as `review-audit-template.md` may exist alongside the three core files.

If lifecycle fields are missing, duplicated inconsistently, or stored in the wrong authoritative location:
- Stop execution.
- Route to `plan-triage` for structural normalization.

If `ANALYSIS` is `IN_PROGRESS`:
- Stop execution.
- Return to planning/update instead of guessing or reopening decisions during implementation.

If `RETRIAGE: REQUIRED`:
- Stop execution.
- Run `plan-triage` before continuing.

If the required structure is otherwise missing or malformed:
- Stop execution.
- Report exactly what is missing or inconsistent.
- Route to `plan-triage` when the problem is structural.
- Route to `plan-update` when the problem is semantic or caused by a changed requirement.

---

## Step 1 — Execution

### Trust boundary

`ANALYSIS: COMPLETE` freezes confirmed decisions, constraints, and conclusions. It does not make implementation files unreadable.

During execution:
- Do not reopen a confirmed architectural decision merely to reconsider it.
- Do not re-run analysis that has already been settled unless new contradictory evidence appears.
- Read current source code, tests, configuration, diffs, generated files, and other implementation state whenever required to implement or verify a REQ-ID correctly.
- If current evidence contradicts a confirmed decision or makes a REQ-ID invalid, stop execution and return to `plan-update` rather than silently changing direction.

### Scope

- Execute one phase at a time unless the user explicitly requests multiple phases or the full plan.
- Work through active REQ-IDs in plan order unless a documented dependency requires a different order.
- Implement one REQ-ID fully before marking it complete.
- A REQ-ID may touch multiple files when required by the logical change; referenced paths are scope guidance, not an absolute prohibition on necessary dependencies.
- If a requirement is materially ambiguous and cannot be resolved from the plan or current repository state, ask one focused question rather than guessing.

### Implementation rules

- Implement the requirement as written, including necessary dependencies that are required for correctness.
- Do not add unrelated improvements.
- Do not silently implement requirements from future phases.
- When an unavoidable dependency causes a file outside the stated path to change, record that dependency in the implementation evidence or execution notes.
- Preserve pre-existing observable behaviour unless the plan explicitly changes it.

---

## Step 2 — Status and evidence update

Never replace the original requirement description with only a path or line reference.

Preserve the requirement and append evidence:

```markdown
- [x] REQ-N.M.n `path/to/file.ext`: [original requirement text]
  - Evidence: `path/to/file.ext:line-range`
```

If multiple files materially implement the REQ-ID:

```markdown
- [x] REQ-N.M.n `path/to/primary.ext`: [original requirement text]
  - Evidence:
    - `path/to/primary.ext:line-range`
    - `path/to/dependency.ext:line-range`
```

Rules:
- Mark `[x]` only after implementation and required verification succeed.
- Keep REQ-ID and requirement wording stable.
- Evidence should point to the resulting implementation when stable line references are available.
- When exact line numbers are unstable or unavailable, use the narrowest verifiable file/symbol/diff reference available rather than inventing a line range.
- Historical or removed REQ-IDs do not block phase completion if the plan clearly marks them inactive/removed.

A phase is COMPLETE only when every active REQ-ID is `[x]`, required verification has passed, implementation evidence is present, and lifecycle state still remains:

```text
ANALYSIS: COMPLETE
RETRIAGE: NO
```

---

## Step 3 — Review

After all active REQ-IDs in a phase are implemented:

1. Check whether the phase has inline verification criteria.
2. Check whether it explicitly references `[review-audit-template]`.
3. If the reference is present, require `review-audit-template.md` alongside the plan and apply it.
4. Apply phase-specific criteria in addition to the shared template when both are present and non-duplicative.
5. If neither exists, perform only the verification required by the individual REQ-IDs and exit criteria.

Do not invoke a shared review template merely because the file exists; the phase must reference it.

If review discovers a normal implementation defect, fix it within the current phase and re-run the relevant verification.

If review discovers evidence that contradicts the plan itself:
- Stop execution.
- Return to `plan-update`.
- Allow `plan-update` to set `ANALYSIS: IN_PROGRESS` and/or `RETRIAGE: REQUIRED` according to the impact of the finding.

---

## What not to do

- Do not execute when `ANALYSIS` is not `COMPLETE`.
- Do not execute when `RETRIAGE` is not `NO`.
- Do not reinterpret confirmed plan decisions without contradictory evidence or an explicit user request.
- Do not avoid reading current code merely because it was cited during planning.
- Do not mark a phase COMPLETE while any active REQ-ID is incomplete, unverified, or lacks evidence.
- Do not erase the original REQ-ID description when recording completion evidence.
- Do not silently implement future-phase requirements.
- Do not create, modify, or delete `review-audit-template.md` during normal execution; structural ownership belongs to `plan-triage`.
- Do not redesign the plan structure during execution; route structural changes back to triage/update.
- Do not set `RETRIAGE: NO`; only `plan-triage` may clear that state.
