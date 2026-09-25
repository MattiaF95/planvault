---
name: planvault
description: Structured technical plan lifecycle management for long agentic coding sessions. Use when creating, refining, restructuring, updating, or executing an implementation plan. Keeps requirements traceable with stable REQ-IDs, preserves confirmed decisions, detects conflicts, and prevents silent requirement loss across long sessions.
---

# planvault

You are the entry point for the planvault skill. Detect the current lifecycle state, load the relevant reference file, and follow it. Do not implement plan logic at router level.

## Authorization before plan writes

Check authorization before drafting a new plan or changing an existing one.

- **New plan:** Write a plan only after the user explicitly asks to create,
  draft, or write one. Discussing work, invoking PlanVault, or supplying a plan
  as context does not by itself authorize drafting.
- **Existing open plan:** Modify requirements only when the user explicitly
  asks to update the plan or add/remove/change planned work. A request to
  execute the existing open scope authorizes its normal status updates.
- **Closed plan:** Treat a plan marked `PLAN_STATUS: COMPLETE`,
  `COMPLETION_ALLOWED: YES`, or explicitly closed/archived as read-only. Before
  reopening it, adding requirements, or changing its completion state, require
  an explicit request to update, reopen, or resume that plan. A reference to the
  plan, a generic request to use/execute it, or newly discovered work is not
  authorization to change it.
- An explicit authorization already given in the current conversation is
  sufficient when it clearly covers the same plan and requested change; do not
  ask again. Examples include “scrivi un piano”, “aggiorna questo piano”,
  “aggiungi questa attività al piano”, or “riprendiamo questo piano con
  PlanVault”.
- If authorization is missing, ask before writing and wait for the answer. Do
  not draft or update first and ask afterward. Read-only inspection and
  reporting do not require plan-write authorization.

## Execution intent

Treat any user instruction that clearly asks to execute, implement, apply, carry out, complete, run, or otherwise perform the work described by an existing plan as an execution request. The exact wording does not matter.

When such an instruction is received:
- treat the plan as the operational prompt and source of truth;
- determine the requested scope from the instruction, using the full plan when no narrower scope is stated;
- follow `references/plan-execute.md` and begin the execution loop after validation succeeds;
- do not answer with validation or analysis only when execution is allowed.
- If the plan is closed and the requested work would require reopening it or
  adding requirements, stop before changing the plan and use the authorization
  gate above.

## Lifecycle routing

1. If no plan exists:
   - First satisfy the authorization gate above.
   - Read `references/plan-draft.md`.
   - Draft or incrementally build the plan within the authorized scope.

2. If a plan exists and the user asks to modify, correct, extend, or remove requirements:
   - Read `references/plan-update.md`.
   - If the update leaves `RETRIAGE: REQUIRED`, read `references/plan-triage.md` before any execution.

3. If a plan exists and `RETRIAGE: REQUIRED`:
   - Read `references/plan-triage.md`.
   - Do not execute until triage has completed and changed the state to `RETRIAGE: NO`.

4. If a plan exists but has not yet been structurally triaged:
   - Read `references/plan-triage.md`.

5. If the user asks to implement or execute a phase, task, or full plan:
   - Read `references/plan-execute.md` only when `ANALYSIS: COMPLETE` and `RETRIAGE: NO`.

6. If execution discovers evidence that invalidates a confirmed plan decision:
   - Stop execution.
   - Return to `references/plan-update.md` to resolve the conflict and set lifecycle state appropriately.
   - Re-run triage only if the update leaves `RETRIAGE: REQUIRED`.

## Lifecycle state

Every plan must expose both lifecycle fields:

```text
ANALYSIS: IN_PROGRESS | COMPLETE
RETRIAGE: REQUIRED | NO
```

`ANALYSIS` describes the state of plan reasoning, not implementation progress:
- `ANALYSIS: IN_PROGRESS` means material requirements, assumptions, or architectural decisions are still unresolved or may change.
- `ANALYSIS: COMPLETE` means the plan contains enough confirmed information to execute without reopening that reasoning; REQ-IDs may still be open and are tracked independently in `tasks.md`.
Reopen `ANALYSIS` only when new evidence or a requested change invalidates a confirmed decision. Do not set it back to `IN_PROGRESS` merely because implementation tasks remain incomplete.

For a single-file plan, both fields live at the top of the plan file.

For a 3-core-file plan, both fields live in `spec.md` and are authoritative for the whole plan. Do not duplicate lifecycle state across `plan.md` or `tasks.md`.

Every plan must also contain one standard execution header in the authoritative
plan file:

```text
PLAN_STATUS: OPEN | IN_PROGRESS | BLOCKED | COMPLETE
EXECUTION_READINESS: NOT_READY | READY | BLOCKED
EXECUTION_SCOPE: REQ_ID | PHASE | FULL_PLAN
CURRENT_REQ: REQ-ID or none
NEXT_ACTION: concrete next action
COMPLETION_ALLOWED: NO | YES
```

Do not replace these fields with custom names or a prose-only status. For a
single-file plan the header is in that file. For a 3-core-file plan it is in
`spec.md`.

## Plan structure

Detect the plan shape before drafting, updating, or executing:

- **Single-file plan:** one self-contained plan file. Keep the lifecycle fields,
  objective, requirements, phases, and exit criteria in that file. Do not add
  core-file links or a triage structure merely for consistency.
- **Three-core-file plan:** use `spec.md`, `plan.md`, and `tasks.md` only when
  triage requires the split, including when active REQ-IDs exceed the threshold
  or the plan domains/phases need separate context. `spec.md` is the entrypoint
  and the only authoritative lifecycle file; `plan.md` describes execution
  strategy; `tasks.md` tracks REQ-IDs and their completion status. Keep
  implementation evidence concise and in an existing field only when the plan
  requires it; do not add evidence prose beneath each task by default.

When triage creates or validates the three-core-file structure, add explicit
relative Markdown links between all three files. At minimum, `spec.md` must
link to `plan.md` and `tasks.md`; `plan.md` and `tasks.md` must link back to
`spec.md` and to the complementary core file. A single-file plan does not need
these links.

If a three-core-file plan is missing a core file or its required links, mark the
structure incomplete and repair it during triage before execution. Do not infer
that a standalone file is part of a core plan only from its directory name.

Execution is allowed only when:

```text
ANALYSIS: COMPLETE
RETRIAGE: NO
```

These fields authorize execution of the plan's open work. They do not mean that the plan or its REQ-IDs are complete.

## Global rules

- Load only the reference needed for the current lifecycle step. Sequential transitions between references are allowed when the workflow explicitly requires them.
- Preserve the plan as the source of truth; do not reconstruct requirements from conversation memory when the plan already exists.
- Never silently remove, renumber, merge, or reinterpret existing REQ-IDs.
- Do not reopen confirmed architectural decisions without new contradictory evidence or an explicit user request.
- Reading current source code, tests, diffs, configuration, and other implementation state is always allowed when required to execute or verify a confirmed plan.
- If the standard execution header is missing or malformed, do not execute;
  route the plan to `plan-update` or `plan-triage` for repair.
- Do not start implementing code from this router level.
- Ask a focused question only when a required plan decision cannot be resolved from the existing plan or repository state.
