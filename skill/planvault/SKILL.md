---
name: plan-master
description: Router skill for structured plan lifecycle management. Detects the current phase (reasoning, drafting, triage, update, execution) and dispatches to the correct reference. Use when discussing, creating, modifying or executing any technical implementation plan. Prevents context drift, silent requirement skipping, and specification loss across long agentic sessions.
when_to_use: Whenever the user mentions a plan, wants to plan something, needs to update a plan, or wants to execute a plan phase.
---

# plan-master

You are the entry point for the planvault skill. Your only job is to detect the current phase and load the correct reference file from `references/`. Do not execute plan logic yourself.

## Phase detection

```
1. Does a plan file (or set of plan files) already exist?
   NO  → Is the user actively reasoning/discussing (no stable decisions yet)?
         YES → Load: references/plan-draft.md (reasoning mode, incremental checkpoints)
         NO  → Load: references/plan-draft.md (write first draft immediately)

   YES → Is the user asking to modify, correct or update the plan?
         YES → Load: references/plan-update.md

         Is the plan freshly written and not yet triaged?
         YES → Load: references/plan-triage.md

         Is the user asking to implement/execute a phase or task?
         YES → Load: references/plan-execute.md
```

## Rules

- Load ONE reference at a time. Never merge logic from multiple references.
- Do not summarise or interpret the plan. Pass context as-is to the loaded reference.
- If the phase is ambiguous, ask the user one direct question: "Should I draft, update or execute the plan?"
- Never start implementing code from this router level.
