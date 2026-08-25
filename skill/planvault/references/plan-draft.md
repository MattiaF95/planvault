# plan-draft

## Responsibilities

1. Parse the user's intent and produce a structured plan file.
2. During reasoning sessions, persist decisions incrementally as they stabilise.
3. Enforce atomic REQ-IDs — never bundle multiple independent requirements into one entry.
4. Never pre-empt triage: draft as a single file; triage decides later whether a structural split is needed.

## Plan file format

```markdown
# Plan: [title]
ANALYSIS: [COMPLETE | IN_PROGRESS]
RETRIAGE: REQUIRED

## 1. Objective
[What and why. Keep it to 3–5 sentences.]

## 2. Context and constraints
- Authoritative evidence: [files, reports, prior fixes already analysed]
- Confirmed decisions: [decisions that should not be reopened without new evidence]
- Out of scope: [explicitly excluded from this plan]

## 3. Phase N — [name]

### N.M [component or file group]
- [ ] REQ-N.M.1 [path/to/file.ext]: [single atomic action — one verb, one outcome]
- [ ] REQ-N.M.2 [path/to/file.ext:line]: [single atomic action]

## Commit order
1. [commit message for phase 1]
2. [commit message for phase 2]

## Exit criteria
- [binary, measurable condition — not "it works"]
```

## Lifecycle fields

Every newly drafted plan starts with both lifecycle fields.

- `ANALYSIS: IN_PROGRESS` while material decisions are still open, being investigated, or may change.
- `ANALYSIS: COMPLETE` when the plan contains enough confirmed information to execute without reopening architectural reasoning.
- `RETRIAGE: REQUIRED` for every new or materially changed draft until `plan-triage` validates the plan structure.
- `RETRIAGE: NO` is owned by `plan-triage` and must not be set by `plan-draft`.

For a single-file plan, both fields remain at the top of that file.

If triage later converts the plan to the 3-core-file structure, both fields move to `spec.md` and become authoritative there.

## REQ-ID rules

- Format: `REQ-{phase}.{section}.{n}`.
- One bullet = one REQ-ID.
- A REQ-ID covers one logical change regardless of how many files it touches.
- Include the primary file path when known; include a line reference only when it is stable and useful.
- Never renumber an existing REQ-ID later to make the list look cleaner.

## ANALYSIS field

- User confirmation may establish `COMPLETE`, but it is not the only valid signal: a plan may also become complete when all material questions are resolved from authoritative evidence and no unresolved decision remains.
- `ANALYSIS: COMPLETE` freezes confirmed decisions, not the underlying files. Execution may still read current source code, tests, diffs, configuration, and implementation state as required.

## Checkpoint rule

- Persist a decision once it is stable enough that losing it would create context drift.
- Prefer writing confirmed or evidence-backed decisions; do not persist speculative branches as if they were settled.
- During a long reasoning session, update the plan incrementally instead of waiting until the end and reconstructing it from memory.
- Do not interrupt the user merely because a fixed number of exchanges has passed. Use the actual amount of stable, not-yet-persisted information as the trigger for a checkpoint.
- Keep `RETRIAGE: REQUIRED` while drafting. Structural validation belongs to `plan-triage`.

## What not to do

- Do not summarise multiple requirements into one REQ-ID to reduce length.
- Do not add implementation requirements that are not supported by the user's request, confirmed decisions, or necessary dependencies discovered from authoritative evidence.
- Do not start implementing code.
- Do not run triage logic — that is `plan-triage`'s responsibility.
- Do not set `RETRIAGE: NO` from draft.
