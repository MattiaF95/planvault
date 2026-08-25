# plan-update

## Responsibilities

1. Identify the exact REQ-IDs or sections to change
2. Detect conflicts between the new instruction and existing plan content
3. Apply the minimum diff — touch only what is explicitly changing
4. Preserve all REQ-IDs not involved in the update, including their status markers
5. Update `ANALYSIS` field if the change affects completeness

## Conflict detection

Before writing any change, scan for content that contradicts the new instruction:

- Same REQ-ID addressed differently in another phase
- A constraint in "Do not reopen" that blocks the requested change
- An out-of-scope declaration that the new requirement violates
- A decision in `plan.md` that conflicts with a change in `tasks.md` (3-file split)

If a conflict is found:
→ **Stop. Do not apply the change.**
→ Report: *"Conflict detected: [new instruction] contradicts [existing content at X]. Which takes precedence?"*
→ Wait for explicit user confirmation.

Never silently overwrite conflicting content.

## Update rules

- **Diff-only**: modify only lines/sections explicitly involved in the change
- **No re-summarising**: do not rewrite narrative sections to "clean them up"
- **REQ-ID continuity**: if a requirement is removed, mark it `~~REQ-N.M.n~~ [removed]` — never delete the line
- **New requirements**: assign the next available REQ-ID; do not renumber existing IDs
- **Status reset**: if a completed `[x]` REQ-ID is materially changed, reset to `[ ]` and note `[reopened]`
- **ANALYSIS field**: if the update introduces new evidence or invalidates cited evidence, set `ANALYSIS: IN_PROGRESS`

## What not to do

- Do not silently resolve conflicts
- Do not add requirements not explicitly requested
- Do not restructure sections not involved in the update
- Do not run triage again — flag to the user if REQ-ID count changed significantly
