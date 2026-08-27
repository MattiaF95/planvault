# plan-execute

## Responsibilities

1. Validate lifecycle state and plan structure before execution.
2. Execute the requested phase, task, or full plan from its REQ-IDs.
3. Preserve requirement text and update completion state with implementation evidence.
4. Apply shared or phase-specific review checks when required.
5. Stop and return to plan-update if current evidence invalidates a confirmed plan decision.

## Execution intent and scope

Treat any instruction that asks the agent to execute, implement, apply, carry out, complete, run, or otherwise perform the work in the plan as an execution request. The wording may vary; the operational meaning is the same.

Determine the scope in this order:
- an explicitly named REQ-ID;
- an explicitly named phase or set of phases;
- the full plan's remaining active REQ-IDs when no narrower scope is stated.

Do not convert an execution request into an analysis-only response.

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
- The standard execution header with all six required fields at the top.
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
`spec.md` must contain the standard execution header with all six required
fields.

Optional support files such as `review-audit-template.md` may exist alongside the three core files.

If lifecycle fields are missing, duplicated inconsistently, or stored in the wrong authoritative location:
- Stop execution.
- Route to `plan-triage` for structural normalization.

If the execution header is missing, uses custom field names, or omits a required
field:
- Stop execution.
- Route to `plan-update` or `plan-triage` to restore the standard header.

If `ANALYSIS` is `IN_PROGRESS`:
- Stop execution.
- Return to planning/update instead of guessing or reopening decisions during implementation.

`ANALYSIS: COMPLETE` does not imply that all REQ-IDs are complete: open REQ-IDs are the work to execute and must remain tracked in `tasks.md` until implementation, verification, review, and evidence are complete.

The lifecycle fields authorize execution only. They do not authorize a completion claim.

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

### Mandatory execution loop

After lifecycle and structure validation succeed:

1. Read the plan and the complete standard execution header once for the current
   execution session.
2. Resolve the requested scope and create an in-memory ordered execution
   cursor containing the active incomplete REQ-IDs in that scope.
3. Set the plan status to `IN_PROGRESS`, set `CURRENT_REQ` to the first open
   item, and set `COMPLETION_ALLOWED: NO`.
4. For each cursor item, implement it completely and run its required
   verification.
5. At the REQ-ID checkpoint, append implementation and verification evidence,
   update the status, `CURRENT_REQ`, and `NEXT_ACTION`, then advance the cursor.
6. Continue with the next cursor item without rereading the entire plan.
7. Audit all REQ-IDs and exit criteria in the requested scope once, at the end
   of the execution session.

The execution cursor is authoritative during the current session. Do not
repeatedly reload or rewrite the full plan between REQ-IDs. Re-read the plan
only when execution resumes in a new session, the user changes the scope, a
plan update is applied, or a blocker/conflict changes the execution path.

Do not execute the same REQ-ID more than once in the same session unless its
verification failed and a concrete fix was applied, or the user explicitly
requests a rerun.

If the cursor does not advance after one implementation/verification cycle,
stop and report the blocking condition instead of retrying indefinitely.

A successful preflight is not an execution result. If prerequisites pass, begin
the first open REQ-ID immediately.

If one REQ-ID is blocked, record the blocker and continue with independent
REQ-IDs when this is safe. If no safe work remains, set the plan status to
`BLOCKED` and preserve the exact resume action.

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
- A REQ-ID covers one verifiable outcome and may touch any number of files. Implement all necessary files together as a single unit.
- If a requirement is materially ambiguous and cannot be resolved from the plan or current repository state, ask one focused question rather than guessing.
- Do not skip, silently defer, or narrow an active REQ-ID.

### Implementation rules

- Implement the requirement as written, including all files necessary to achieve the stated outcome.
- Necessary dependencies are part of the REQ-ID — they do not require a new REQ-ID. Record them in the implementation evidence.
- Do not add unrelated improvements or changes not required by the current REQ-ID.
- Do not silently implement requirements from future phases.
- Preserve pre-existing observable behaviour unless the plan explicitly changes it.

### Necessary dependency vs. unrelated improvement

| | Necessary dependency | Unrelated improvement |
|---|---|---|
| **Required for correctness?** | Yes — the REQ-ID outcome is wrong or incomplete without it | No — the REQ-ID works correctly without it |
| **Action** | Implement inline; record in evidence | Do not implement; flag to user if genuinely useful |
| **Creates new REQ-ID?** | Never | Only if user explicitly adds it to the plan |

Examples of necessary dependencies: a validation function needed by the feature being implemented; a config key required to enable the new behaviour; a type definition consumed by changed code. These are part of the REQ-ID, not additions to the plan.

---

## Step 2 — Status and evidence update

Never replace the original requirement description with only a path or line reference.

Preserve the requirement and append evidence:

```markdown
- [x] REQ-N.M.n [original requirement text]
  - Evidence: `path/to/primary.ext:line-range`
```

If multiple files materially implement the REQ-ID:

```markdown
- [x] REQ-N.M.n [original requirement text]
  - Evidence:
    - `path/to/primary.ext:line-range`
    - `path/to/secondary.ext:line-range`
```

Rules:
- Mark `[x]` only after implementation and required verification succeed.
- Keep REQ-ID and requirement wording stable.
- Record the files that materially implement the outcome. Do not enumerate every file touched by a minor dependency — use the narrowest verifiable reference that confirms the outcome.
- When exact line numbers are unstable or unavailable, use the narrowest verifiable file/symbol/diff reference rather than inventing a line range.
- Historical or removed REQ-IDs do not block phase completion if the plan clearly marks them inactive/removed.

Use these execution states when the plan supports them:

```text
[ ] NOT_STARTED
[~] IN_PROGRESS
[!] BLOCKED
[x] COMPLETE
```

`IN_PROGRESS`, `BLOCKED`, partial, skipped, planned, or unverified work is not
complete. A blocked item must include the blocker, evidence, required
resolution, and resume action.

A phase is COMPLETE only when every active REQ-ID is `[x]`, the stated outcome is verifiable, and lifecycle state still reads:

```text
ANALYSIS: COMPLETE
RETRIAGE: NO
```

The full plan is COMPLETE only when every active REQ-ID in every phase is `[x]`,
every exit criterion is checked with evidence, no blocker or pending review
remains, and the execution header says `PLAN_STATUS: COMPLETE` and
`COMPLETION_ALLOWED: YES`.

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
- Do not mark a phase COMPLETE while any active REQ-ID is incomplete, unverified, or its outcome is not confirmed.
- Do not mark the full plan COMPLETE because one phase, scenario, test, or batch of REQ-IDs passed.
- Do not report completion while any active REQ-ID or exit criterion is open, blocked, partial, skipped, planned, or missing evidence.
- Do not erase the original REQ-ID description when recording completion evidence.
- Do not silently implement future-phase requirements.
- Do not create new REQ-IDs for necessary dependencies discovered during execution — record them in evidence.
- Do not create, modify, or delete `review-audit-template.md` during normal execution; structural ownership belongs to `plan-triage`.
- Do not redesign the plan structure during execution; route structural changes back to triage/update.
- Do not set `RETRIAGE: NO`; only `plan-triage` may clear that state.
