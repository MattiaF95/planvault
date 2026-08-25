# plan-triage

## Responsibilities

1. Count total REQ-IDs across the plan
2. Decide: single file or 3-file split
3. Detect repetitive review/test patterns across phases
4. If pattern detected: create `review-audit-template.md` alongside the plan files
5. Confirm the structure to the user before any execution begins

## Decision: single file vs 3-file split

Keep as **single file** if ALL of the following are true:
- Total REQ-IDs ≤ 25–30
- Phases are well-isolated with clear boundaries
- The plan covers a single feature or tightly related changes
- No phase has more than 8–10 independent REQ-IDs

Split into **3 files** if ANY of the following is true:
- Total REQ-IDs > 25–30
- Multiple unrelated features or domains
- A single phase contains > 8–10 independent REQ-IDs that cannot be sub-phased
- The plan is expected to span multiple sessions (high context rot risk)

### 3-file structure

```
spec.md    ← what and why: objective, constraints, out-of-scope, exit criteria
plan.md    ← how: architecture, phases, dependencies, commit order
tasks.md   ← REQ-ID checklist only: [ ]/[x] + file:line ref, nothing else
```

### Sub-phase split (same file, high density)

If a single phase has > 8–10 REQ-IDs but the plan otherwise qualifies as single-file:

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
- Define the same review criteria across multiple phases

If this pattern is detected across ≥ 2 phases:
→ Create `review-audit-template.md` alongside the plan files
→ In the plan, replace the repetitive review block with: `[review-audit-template]`
→ Inform the user: *"Repetitive review pattern detected across N phases. Created review-audit-template.md. Each task in plan-execute will decide whether to invoke it based on phase context."*

If no repetitive pattern: do not create the file.

## review-audit-template.md format

```markdown
# review-audit-template

## Independent review (sub-agent)
Launch an independent sub-agent with the following checks:
- Full and correct application of all REQ-IDs in this phase (no silent omissions)
- No redundant, duplicated or unused code introduced
- No bugs introduced by the changes
- No regressions on pre-existing observable behaviour

Fix all confirmed findings. Re-run targeted specs and repeat review before proceeding.

## REQ-ID audit (before marking phase COMPLETE)
For each REQ-ID in this phase:
`REQ-N.M.n [x] path/to/file.ext:line-range`

If any REQ-ID has no diff reference, the phase cannot be marked COMPLETE.
```

## Output

After triage, report:
- Structure decision (single file / 3-file split) and reason
- Total REQ-ID count
- Whether review-audit-template.md was created and why
- Any phases internally sub-split
- Confirmation: *"Plan is ready for execution with plan-execute."*
