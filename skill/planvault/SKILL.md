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
   - If that workflow marks the plan as needing structural re-triage, then read `references/plan-triage.md` before execution.

3. If a plan exists but has not yet been structurally triaged:
   - Read `references/plan-triage.md`.

4. If the user asks to implement or execute a phase, task, or full plan:
   - Read `references/plan-execute.md`.

5. If execution discovers evidence that invalidates a confirmed plan decision:
   - Stop execution.
   - Return to `references/plan-update.md` to resolve the conflict and set analysis state appropriately.
   - Re-run triage only if the update materially changes plan structure.

## Global rules

- Load only the reference needed for the current lifecycle step. Sequential transitions between references are allowed when the workflow explicitly requires them.
- Preserve the plan as the source of truth; do not reconstruct requirements from conversation memory when the plan already exists.
- Never silently remove, renumber, merge, or reinterpret existing REQ-IDs.
- Do not reopen confirmed architectural decisions without new contradictory evidence or an explicit user request.
- Reading current source code, tests, diffs, configuration, and other implementation state is always allowed when required to execute or verify a confirmed plan.
- Do not start implementing code from this router level.
- Ask a focused question only when a required plan decision cannot be resolved from the existing plan or repository state.
