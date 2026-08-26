---
name: planvault
description: Structured technical plan lifecycle management for long agentic coding sessions. Use when creating, refining, restructuring, updating, or executing an implementation plan. Keeps requirements traceable with stable REQ-IDs, preserves confirmed decisions, detects conflicts, and prevents silent requirement loss across long sessions.
---

# planvault

You are the entry point for the planvault skill. Detect the current lifecycle state, load the relevant reference file, and follow it. Do not implement plan logic at router level.

## Lifecycle routing

1. If no plan exists:
   - Read `references/plan-draft.md`.
   - Draft or incrementally build the plan from the current discussion.

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

For a single-file plan, both fields live at the top of the plan file.

For a 3-core-file plan, both fields live in `spec.md` and are authoritative for the whole plan. Do not duplicate lifecycle state across `plan.md` or `tasks.md`.

## Plan structure

Detect the plan shape before drafting, updating, or executing:

- **Single-file plan:** one self-contained plan file. Keep the lifecycle fields,
  objective, requirements, phases, and exit criteria in that file. Do not add
  core-file links or a triage structure merely for consistency.
- **Three-core-file plan:** use `spec.md`, `plan.md`, and `tasks.md` only when
  triage requires the split, including when active REQ-IDs exceed the threshold
  or the plan domains/phases need separate context. `spec.md` is the entrypoint
  and the only authoritative lifecycle file; `plan.md` describes execution
  strategy; `tasks.md` tracks REQ-IDs and evidence.

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

## Global rules

- Load only the reference needed for the current lifecycle step. Sequential transitions between references are allowed when the workflow explicitly requires them.
- Preserve the plan as the source of truth; do not reconstruct requirements from conversation memory when the plan already exists.
- Never silently remove, renumber, merge, or reinterpret existing REQ-IDs.
- Do not reopen confirmed architectural decisions without new contradictory evidence or an explicit user request.
- Reading current source code, tests, diffs, configuration, and other implementation state is always allowed when required to execute or verify a confirmed plan.
- Do not start implementing code from this router level.
- Ask a focused question only when a required plan decision cannot be resolved from the existing plan or repository state.
