# plan-execute

## Responsibilities

1. Validate plan structure (Step 0)
2. Execute the current phase REQ-IDs (Step 1)
3. Update REQ-ID status markers after each implementation (Step 2)
4. Invoke review-audit-template only if applicable (Step 3)

---

## Step 0 — Structure validation (always first)

**Single-file plan:**
- One plan file with `ANALYSIS: COMPLETE`
- File contains REQ-IDs with `[ ]`/`[x]` markers
- Phase structure is present

**3-file split:**
- Exactly three files: `spec.md`, `plan.md`, `tasks.md`
- `tasks.md` contains all REQ-IDs with status markers

If structure is **not** conformant:
→ **Stop. Do not execute.**
→ Report exactly what is missing or malformed.
→ Suggest: run `plan-triage` or `plan-update` to fix the structure first.

If `ANALYSIS` is missing or `IN_PROGRESS`:
→ **Stop.**
→ Report: *"Plan analysis is not marked COMPLETE. Confirm with the user that the plan is ready before executing."*

---

## Step 1 — Execution

### Trust boundary

If the plan declares `ANALYSIS: COMPLETE` and cites specific files or reports:
- **Do not re-read, re-analyse or re-confirm** any cited evidence
- Treat cited content as ground truth
- Proceed directly to implementation

### Scope

- Execute **one phase at a time** unless explicitly requested otherwise
- Work through REQ-IDs in order
- Each REQ-ID is a single atomic action — implement it fully before moving to the next
- If a REQ-ID is ambiguous, stop and ask one focused question. Do not guess.

### Implementation rules

- Implement exactly what the REQ-ID states — no additions, no omissions, no unrequested improvements
- Do not anticipate future phases
- Do not modify files not referenced by the current REQ-ID unless a dependency is unavoidable (note it explicitly)

---

## Step 2 — Status update (after each REQ-ID)

```
- [x] REQ-N.M.n [path/to/file.ext:line-range]
```

- `[x]` + REQ-ID + file path + line range of the diff
- No prose, no explanation
- Update only the line of the completed REQ-ID

A phase is **COMPLETE** only when every REQ-ID has `[x]` and a file:line reference. No exceptions.

---

## Step 3 — Review (conditional)

After all REQ-IDs in a phase are marked `[x]`:

1. Check if `review-audit-template.md` exists alongside the plan files
2. Check if the current phase explicitly references `[review-audit-template]`
3. If **both** are true → read the file and apply its checks before marking COMPLETE
4. If the phase has its own inline verification criteria → apply those instead
5. If neither → mark COMPLETE and proceed

The decision to invoke the template is contextual — never automatic.

---

## What not to do

- Do not re-analyse cited evidence
- Do not mark a phase COMPLETE if any REQ-ID lacks a file:line diff reference
- Do not implement requirements from future phases
- Do not create, modify or delete `review-audit-template.md` — that is plan-triage's responsibility
- Do not decide whether the plan needs structural changes — that is plan-triage and plan-update's responsibility
