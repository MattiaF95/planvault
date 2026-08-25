---
name: plan-execute
description: Validates plan structure and executes the current phase or specific REQ-IDs. Enforces trust boundary (no re-analysis of already-confirmed context), applies atomic requirement tracking, and optionally invokes review-audit-template if present and referenced by the current phase.
when_to_use: When a triaged plan exists and the user wants to implement a phase or specific tasks.
---

# plan-execute

## Your responsibilities

1. Validate plan structure (Step 0)
2. Execute the current phase REQ-IDs (Step 1)
3. Update REQ-ID status markers after each implementation (Step 2)
4. Invoke review-audit-template only if applicable (Step 3)

---

## Step 0 — Structure validation (always first)

Before doing anything, verify the plan structure:

**Single-file plan:**
- One plan file exists with `ANALYSIS: COMPLETE`
- File contains REQ-IDs with `[ ]`/`[x]` markers
- Phase structure is present

**3-file split:**
- Exactly three files exist: `spec.md`, `plan.md`, `tasks.md`
- `tasks.md` contains all REQ-IDs with status markers

If structure is **not** conformant:
→ **Stop. Do not execute.**
→ Report exactly what is missing or malformed.
→ Suggest: run `plan-triage` or `plan-update` to fix the structure first.

If `ANALYSIS` field is missing or set to `IN_PROGRESS`:
→ **Stop.**
→ Report: *"Plan analysis is not marked COMPLETE. Confirm with the user that the plan is ready before executing."

---

## Step 1 — Execution

### Trust boundary

If the plan declares `ANALYSIS: COMPLETE` and cites specific files, reports or prior analysis as evidence:
- **Do not re-read, re-analyse or re-confirm** any cited evidence
- Treat all cited content as ground truth
- Proceed directly to implementation

Violating the trust boundary wastes tokens and re-introduces the context drift the plan was designed to prevent.

### Scope

- Execute **one phase at a time** unless the user explicitly requests multiple phases
- Within a phase, work through REQ-IDs in order
- Each REQ-ID is a single atomic action — implement it fully before moving to the next
- If a REQ-ID is ambiguous, stop and ask one focused question. Do not guess.

### Implementation rules

- Implement exactly what the REQ-ID states — no additions, no omissions, no improvements not in the plan
- Do not anticipate future phases
- Do not modify files not referenced by the current REQ-ID unless a dependency is unavoidable (if so, note it explicitly)

---

## Step 2 — Status update (after each REQ-ID)

After implementing each REQ-ID, update its marker inline in the plan file:

```
- [x] REQ-N.M.n [path/to/file.ext:line-range]
```

Format rules:
- `[x]` + REQ-ID + file path + line range of the diff
- No prose, no explanation, no summary
- Update only the line of the completed REQ-ID — do not rewrite surrounding content

A phase is **COMPLETE** only when every REQ-ID in that phase has `[x]` and a file:line reference. No exceptions.

---

## Step 3 — Review (conditional)

After all REQ-IDs in a phase are marked `[x]`:

1. Check if `plan-execute/references/review-audit-template.md` exists
2. Check if the current phase explicitly references `[review-audit-template]`
3. If **both** are true → read the file and apply its checks before marking the phase COMPLETE
4. If the phase has its own inline verification criteria → apply those instead; the template is not needed
5. If neither condition is met → mark the phase COMPLETE and proceed

The decision to invoke the template is contextual — it is never automatic.

---

## What not to do

- Do not re-analyse cited evidence even if it seems relevant
- Do not mark a phase COMPLETE if any REQ-ID lacks a file:line diff reference
- Do not implement requirements from future phases
- Do not create, modify or delete the review-audit-template — that is plan-triage's responsibility
- Do not decide whether the plan needs structural changes — that is plan-triage and plan-update's responsibility
