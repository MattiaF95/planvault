---
name: plan-update
description: Updates an existing plan file (or set of plan files) when decisions change, new requirements emerge, or the user corrects something. Handles conflicts between new instructions and existing plan content.
when_to_use: When a plan file already exists and the user wants to modify, correct, extend or reprioritise it.
---

# plan-update

## Your responsibilities

1. Identify the exact REQ-IDs or sections to change
2. Detect conflicts between the new instruction and existing plan content
3. Apply the minimum diff — touch only what is explicitly changing
4. Preserve all REQ-IDs not involved in the update, including their status markers
5. Update `ANALYSIS` field if the change affects the completeness of the analysis

## Conflict detection

Before writing any change, scan the existing plan for content that contradicts the new instruction:

- Same REQ-ID addressed differently in another phase
- A constraint in section 2 (Context and constraints / Do not reopen) that blocks the requested change
- An out-of-scope declaration that the new requirement violates
- A decision in `plan.md` that conflicts with a change requested in `tasks.md` (3-file split only)

If a conflict is found:
→ **Stop. Do not apply the change.**
→ Report to the user:
  *"Conflict detected: [new instruction] contradicts [existing content at location X]. Which takes precedence?"*
→ Wait for explicit user confirmation before proceeding.

Never silently overwrite conflicting content.

## Update rules

- **Diff-only**: modify only the lines/sections explicitly involved in the change
- **No re-summarising**: do not rewrite narrative sections to "clean them up" unless explicitly asked
- **REQ-ID continuity**: if a requirement is removed, mark it `~~REQ-N.M.n~~ [removed]` rather than deleting the line — preserves traceability
- **New requirements**: assign the next available REQ-ID in the correct phase/section; do not renumber existing IDs
- **Status reset**: if a completed `[x]` REQ-ID is materially changed, reset it to `[ ]` and note: `[reopened]`
- **ANALYSIS field**: if the update introduces new evidence or invalidates cited evidence, set `ANALYSIS: IN_PROGRESS` until the user re-confirms

## What not to do

- Do not silently resolve conflicts by choosing one side
- Do not add requirements not explicitly requested by the user
- Do not restructure or reformat sections not involved in the update
- Do not run triage again unless the update significantly changes REQ-ID count (flag to the user instead)
