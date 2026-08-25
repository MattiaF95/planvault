# review-audit-template

> This file is created by `plan-triage` when a repetitive review pattern is detected across plan phases.
> It is **not loaded automatically**. Each phase must explicitly reference `[review-audit-template]` for this file to be applied.
> If the current phase has its own inline verification criteria, use those instead.

---

## Independent review (sub-agent)

Launch an independent sub-agent with the following checks:

- **Completeness**: every REQ-ID in this phase is fully implemented — no silent omissions, no partial stubs
- **No redundancy**: no duplicated, dead or unreachable code introduced
- **No bugs**: no logic errors, unhandled edge cases or broken contracts
- **No regressions**: pre-existing observable behaviour is preserved — CRUD, navigation, state, API contracts, accessibility
- **Scope discipline**: no changes outside the files and functions referenced by the phase REQ-IDs, unless noted as an explicit dependency

Fix all confirmed findings. Re-run targeted specs. Repeat review until clean before proceeding to the next phase.

---

## REQ-ID audit (mandatory before marking phase COMPLETE)

For each REQ-ID declared in this phase:

```
REQ-N.M.n  [x]  path/to/file.ext:line-range
```

- Every REQ-ID must have `[x]` status
- Every REQ-ID must have an associated `file:line-range` diff reference
- If any REQ-ID is missing either, the phase **cannot** be marked COMPLETE
- Report the full table to the user before closing the phase
