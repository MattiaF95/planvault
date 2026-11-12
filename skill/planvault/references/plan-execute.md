# plan-execute

## Responsibilities

1. Validate lifecycle state and plan structure before execution.
2. Execute the requested phase, task, or full plan from its REQ-IDs.
3. Preserve requirement text and update each existing completion marker in place.
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

Recovery exception: for a closed plan with pending, already-authorized
finalization, read its closure/retention section and resume Step 4 directly.
Do not run open-work validation or implementation again.

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

`ANALYSIS: COMPLETE` does not imply that all REQ-IDs are complete: open REQ-IDs are the work to execute and must remain tracked in `tasks.md` until implementation, verification, review, and any evidence required by the plan are complete.

The lifecycle fields authorize execution only. They do not authorize a completion claim.

If `RETRIAGE: REQUIRED`:
- Stop execution.
- Run `plan-triage` before continuing.

If the required structure is otherwise missing or malformed:
- Stop execution.
- Report exactly what is missing or inconsistent.
- Route to `plan-triage` when the problem is structural.
- Route to `plan-update` when the problem is semantic or caused by a changed requirement.

### Planning decisions and runtime checks

Require the agent strategy, verification strategy, documentation inventory, and
closure/retention instructions defined by plan-draft and validated by triage.
Route missing decisions to plan-update/triage; do not invent them during execution.
Before delegation, recheck available slots, supported models/effort, authorization,
and conflicts. Apply the minimum of user limit, runtime capacity, and safe ready
work; count reviewers too. Use the recorded fallback if capacity is lower or
models cannot be used; otherwise resolve the mismatch before launching workers.

If a closed plan has PENDING finalization, resume Step 4 only when its recorded
retention action is already authorized. Do not rerun implementation, reopen
requirements, or require open REQ-IDs for this recovery path.

---

## Step 1 — Execution

### Mandatory execution loop

After lifecycle and structure validation succeed:

1. Read the authoritative plan and complete standard execution header once for
   the current execution session. For a 3-core-file plan, read `spec.md`,
   `plan.md`, and `tasks.md` together.
2. Resolve the requested scope and create an in-memory ordered execution
   cursor containing the active incomplete REQ-IDs in that scope.
   The cursor may group independent REQ-IDs into the plan's validated parallel
   batches, respecting phase boundaries, dependencies, and integration order.
3. Set the plan status to `IN_PROGRESS`, set `CURRENT_REQ` to the first open
   item, and set `COMPLETION_ALLOWED: NO`.
4. For each cursor item or validated batch, implement each outcome completely
   and run its required verification. Workers return changes and evidence to
   the coordinator, who integrates and verifies the combined result before
   marking affected REQ-IDs complete. Workers do not edit the plan or commit.
   Keep CURRENT_REQ as a single coordinator-owned cursor item; the validated
   strategy identifies batch membership, without adding custom header fields.
5. At the REQ-ID checkpoint, change the existing row to `[x]` only if the
   requirement is fully implemented and its required verification passes;
   otherwise leave it incomplete. Update `CURRENT_REQ` and `NEXT_ACTION`, then
   advance the cursor.
6. Continue with the next cursor item without rereading the entire plan.
7. At each phase boundary, re-read the authoritative plan context before
   starting the next phase.
8. Audit all REQ-IDs and exit criteria in the requested scope once, at the end
   of the execution session.

The execution cursor is authoritative during the current session. Do not
repeatedly reload or rewrite the full plan between REQ-IDs. Re-read the plan
only when execution resumes in a new session, the user changes the scope, a
plan update is applied, a blocker/conflict changes the execution path, a phase
boundary is reached, or context is compacted. For a 3-core-file plan, the
authoritative context is `spec.md`, `plan.md`, and `tasks.md` read together.

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

When a command, migration, or other operation can fail partially, follow the
plan's safe retry or recovery action. If the plan does not provide one and the
operation is risky, stop before proceeding destructively and report the gap.

### Trust boundary

`ANALYSIS: COMPLETE` freezes confirmed decisions, constraints, and conclusions. It does not make implementation files unreadable.

During execution:
- Do not reopen a confirmed architectural decision merely to reconsider it.
- Do not re-run analysis that has already been settled unless new contradictory evidence appears.
- Read current source code, tests, configuration, diffs, generated files, and other implementation state whenever required to implement or verify a REQ-ID correctly.
- If current evidence contradicts a confirmed decision or makes a REQ-ID invalid, stop execution and return to `plan-update` rather than silently changing direction.

### Scope

- Execute one phase at a time unless the user explicitly requests multiple phases or the full plan.
- Work through active REQ-IDs in plan order, using only validated independent batches or documented dependencies to change scheduling.
- Implement one REQ-ID fully before marking it complete.
- A REQ-ID covers one verifiable outcome and may touch any number of files. Implement all necessary files together as a single unit.
- If a requirement is materially ambiguous and cannot be resolved from the plan or current repository state, ask one focused question rather than guessing.
- Do not skip, silently defer, or narrow an active REQ-ID.

### Implementation rules

- Implement the requirement as written, including all files necessary to achieve the stated outcome.
- Necessary dependencies are part of the REQ-ID — they do not require a new REQ-ID. Do not add checklist rows to describe them; mention them only in an existing evidence field when the plan requires one.
- Do not add unrelated improvements or changes not required by the current REQ-ID.
- Do not silently implement requirements from future phases.
- Preserve pre-existing observable behaviour unless the plan explicitly changes it.

### Necessary dependency vs. unrelated improvement

| | Necessary dependency | Unrelated improvement |
|---|---|---|
| **Required for correctness?** | Yes — the REQ-ID outcome is wrong or incomplete without it | No — the REQ-ID works correctly without it |
| **Action** | Implement inline; do not add a task or narrative row | Do not implement; flag to user if genuinely useful |
| **Creates new REQ-ID?** | Never | Only if user explicitly adds it to the plan |

Examples of necessary dependencies: a validation function needed by the feature being implemented; a config key required to enable the new behaviour; a type definition consumed by changed code. These are part of the REQ-ID, not additions to the plan.

---

## Step 2 — Update status in place

For an executed REQ-ID, update its existing checklist row. Preserve the REQ-ID
and requirement wording; change `[ ]` to `[x]` only after implementation and
required verification succeed.

```markdown
- [x] REQ-N.M.n [original requirement text]
```

Do not append a duplicate task, subtask, implementation recap, or evidence
bullet beneath the row. Do not add prose after the task list to narrate work.
Update an evidence field only when the plan already defines one or explicitly
requires per-REQ evidence, and keep it to a concise pointer or result. Otherwise
leave evidence in the verification artifacts and summarize it briefly in the
response to the user.

Keep plan edits minimal: update existing REQ-ID and exit-criterion markers,
standard execution-header fields, and lifecycle fields only when their state
changes. Do not add tasks, subtasks, evidence rows, or execution summaries
unless the user or the existing plan explicitly requires them.

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

Before closing the full plan, verify every planned documentation update and
its cross-document consistency criteria. If new documentation impacts emerge,
route them through plan-update rather than silently omitting them.

The full plan is COMPLETE only when every active REQ-ID in every phase is `[x]`,
every exit criterion is checked with evidence, no blocker or pending review
remains, and the execution header says `PLAN_STATUS: COMPLETE` and
`COMPLETION_ALLOWED: YES`.

The final audit must also demonstrate the overall objective through an
observable result or an equivalent project-specific proof.

---

## Step 3 — Review

After all active REQ-IDs in a phase are implemented:

1. Check whether the phase has inline verification criteria.
2. Check whether it explicitly references `[review-audit-template]`.
3. If the reference is present, require `review-audit-template.md` alongside the plan and apply it.
4. Apply phase-specific criteria in addition to the shared template when both are present and non-duplicative.
5. If neither exists, perform only the verification required by the individual REQ-IDs and exit criteria.

After the phase review and required plan-state updates succeed, create one atomic,
phase-level Git commit when the repository uses Git and committing is authorized. When a phase commit is required, do not advance to the
next phase until it succeeds. If committing is blocked, preserve the
implementation, report the exact blocker and resume action, and do not claim
final delivery.

Do not invoke a shared review template merely because the file exists; the phase must reference it.

If review discovers a normal implementation defect, fix it within the current phase and re-run the relevant verification.

If review discovers evidence that contradicts the plan itself:
- Stop execution.
- Return to `plan-update`.
- Allow `plan-update` to set `ANALYSIS: IN_PROGRESS` and/or `RETRIAGE: REQUIRED` according to the impact of the finding.

---

## Step 4 — Close and finalize the full plan

Run this step only after all active REQ-IDs, required documentation, reviews,
and pre-closure exit criteria for the full plan are verified. Retention is a
post-closure delivery condition, not a prerequisite for closing. Completing a requested subset
never archives or deletes a plan with remaining work.

1. Record closure in the authoritative plan: PLAN_STATUS: COMPLETE,
   COMPLETION_ALLOWED: YES, CURRENT_REQ: none, and NEXT_ACTION describing the
   recorded finalization action. Keep Finalization status: PENDING until the
   retention action is verified. Closure and final delivery are distinct.
2. Apply only the recorded, authorized action to the exact listed artifacts.
   ARCHIVE: use the user-confirmed archive directory in the plan's repository.
   If it is missing, ask before creating or substituting any directory unless
   its creation was explicitly authorized already. This includes new per-plan
   subdirectories inside an existing archive. Move core/support files
   together to the declared destination, do not
   overwrite an existing archive, preserve links between plan files, rebase
   relative links to repository files as needed, and update recorded external
   references. DELETE: require the recorded explicit user choice and
   preserve a closure/verification record at the declared durable location
   before removing any listed artifact. Do not remove unrelated directory contents.
3. Verify the original artifacts are absent from the active location and that
   archive contents, retained evidence, and updated references are correct.
   For ARCHIVE, set Finalization status: DONE and NEXT_ACTION: none in the
   archived authoritative file. For DELETE, record DONE in the retained closure
   record. This administrative update is allowed without reopening the plan.
4. If finalization fails, preserve COMPLETE for verified implementation and
   PENDING for finalization, recording the error and exact recovery action in
   the surviving authoritative plan or retained closure record. Resume only
   missing finalization steps; never overwrite an archive or repeat deletion
   blindly. Report final delivery as pending until verification succeeds.

Integrate finalization with the last phase commit when authorized; if that
commit was already created, use a separate scoped finalization commit under the
same Git authorization. Repository/user permission rules still govern commits.
Do not claim delivery while required finalization or its required commit remains
pending. Report the archive or retained closure-record location in the final response.

---

## What not to do

- Do not execute when `ANALYSIS` is not `COMPLETE`.
- Do not execute when `RETRIAGE` is not `NO`.
- Do not reinterpret confirmed plan decisions without contradictory evidence or an explicit user request.
- Do not avoid reading current code merely because it was cited during planning.
- Do not mark a phase COMPLETE while any active REQ-ID is incomplete, unverified, or its outcome is not confirmed.
- Do not mark the full plan COMPLETE because one phase, scenario, test, or batch of REQ-IDs passed.
- Do not report completion while any active REQ-ID or exit criterion is open, blocked, partial, skipped, planned, or missing evidence required by the plan.
- Preserve the original REQ-ID and requirement text when updating its status.
- Do not silently implement future-phase requirements.
- Do not create new REQ-IDs for necessary dependencies discovered during execution or add rows beneath existing REQ-IDs to describe them.
- Do not create, modify, or delete `review-audit-template.md` during implementation; structural ownership belongs to `plan-triage`. Step 4 may move/delete a listed support file as part of authorized finalization.
- Do not redesign the plan structure during execution; route structural changes back to triage/update.
- Do not set `RETRIAGE: NO`; only `plan-triage` may clear that state.
