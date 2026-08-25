---
name: plan-triage
description: Evaluates a completed plan draft and decides whether it should remain a single file or be split into spec.md + plan.md + tasks.md. Also detects repetitive review/test patterns across phases and creates the review-audit-template reference file if needed.
when_to_use: After plan-draft has produced a first draft and the user wants to validate the plan structure before execution.
---

# plan-triage

## Your responsibilities

1. Count total REQ-IDs across the plan
2. Decide: single file or 3-file split
3. Detect repetitive review/test patterns across phases
4. If pattern detected: create `plan-execute/references/review-audit-template.md`
5. Confirm the structure to the user before any execution begins

## Decision: single file vs 3-file split

Keep as **single file** if ALL of the following are true:
- Total REQ-IDs ≤ 25–30
- Phases are well-isolated (each phase has a clear boundary and its own review gate)
- The plan covers a single feature or a set of tightly related changes
- No phase has more than 8–10 independent REQ-IDs

Split into **3 files** if ANY of the following is true:
- Total REQ-IDs > 25–30
- Multiple unrelated features or domains in the same plan
- A single phase contains > 8–10 independent REQ-IDs that cannot be sub-phased
- The plan is expected to span multiple sessions (high context rot risk)

### 3-file structure

```
spec.md    ← what and why: objective, constraints, out-of-scope, exit criteria
plan.md    ← how: architecture, phases, dependencies, commit order
tasks.md   ← REQ-ID checklist only: [ ]/[x] + file:line ref, nothing else
```

When splitting:
- Move all REQ-IDs and status markers to `tasks.md`
- Move all prose reasoning to `spec.md` and `plan.md`
- `tasks.md` must be purely mechanical — no narrative, no explanations

### Sub-phase split (same file, high density)

If a single phase has > 8–10 REQ-IDs but the plan otherwise qualifies as single-file, split that phase internally:

```markdown
## Phase 2 — [name]
### Phase 2a — [sub-objective A]
- [ ] REQ-2a.1.1 ...
### Phase 2b — [sub-objective B]
- [ ] REQ-2b.1.1 ...
```

## Detecting the review pattern

Scan the plan for sections that:
- Repeat an identical or near-identical review/verification block after each phase
- Involve launching an independent sub-agent for review
- Define the same set of review criteria (no regressions, no unused code, diff coverage, etc.) across multiple phases

If this pattern is detected across ≥ 2 phases:
→ Create `skills/plan-execute/references/review-audit-template.md` using the template below
→ In the plan, replace the repetitive review block with: `[review-audit-template]`
→ Inform the user: *"Repetitive review pattern detected across N phases. Created review-audit-template.md. Each task in plan-execute will decide whether to invoke it based on phase context."

If no repetitive pattern: do not create the file. Individual tasks will include their own inline verification criteria.

## review-audit-template.md content

```markdown
# review-audit-template

## Independent review (sub-agent)
Launch an independent sub-agent with the following checks:
- Full and correct application of all REQ-IDs in this phase (no silent omissions)
- No redundant, duplicated or unused code introduced
- No bugs introduced by the changes
- No regressions on pre-existing observable behaviour (CRUD, navigation, state, accessibility)
- Business logic and user-visible actions produce identical results to pre-fix, except where explicitly changed by the plan

Fix all confirmed findings. Re-run targeted specs and repeat review before proceeding to next phase.

## REQ-ID audit (before marking phase COMPLETE)
For each REQ-ID in this phase, the agent must produce:
`REQ-N.M.n [x] path/to/file.ext:line-range`

If any REQ-ID has no associated diff reference, the phase cannot be marked COMPLETE.
```

## Output

After triage, report to the user:
- Structure decision (single file / 3-file split) and reason
- Total REQ-ID count
- Whether review-audit-template.md was created and why
- Any phases that were internally sub-split
- Confirmation: *"Plan is ready for execution with plan-execute."
