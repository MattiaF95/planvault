---
name: plan-master
description: Router skill for structured plan lifecycle management. Detects the current phase (reasoning, drafting, triage, update, execution) and dispatches to the correct sub-skill. Invoke when discussing, creating, modifying or executing any technical implementation plan.
when_to_use: Whenever the user mentions a plan, wants to plan something, needs to update a plan, or wants to execute a plan phase.
---

# plan-master

You are the entry point for the planvault skill system. Your only job is to detect the current phase and load the correct sub-skill. Do not execute plan logic yourself.

## Phase detection

Evaluate the current state using this decision tree:

```
1. Does a plan file (or set of plan files) already exist?
   NO  → Is the user actively reasoning/discussing (no stable decisions yet)?
         YES → Load: plan-draft (reasoning mode, incremental checkpoints)
         NO  → Load: plan-draft (write first draft immediately)

   YES → Is the user asking to modify, correct or update the plan?
         YES → Load: plan-update

         Is the plan freshly written and not yet triaged?
         YES → Load: plan-triage

         Is the user asking to implement/execute a phase or task?
         YES → Load: plan-execute
```

## Rules

- Load ONE sub-skill at a time. Never merge logic from multiple sub-skills.
- Do not summarise or interpret the plan. Pass context as-is to the sub-skill.
- If the phase is ambiguous, ask the user one direct question: "Should I draft, update or execute the plan?"
- Never start implementing code from this router level.
