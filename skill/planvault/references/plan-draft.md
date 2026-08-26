# plan-draft

## Responsibilities

1. Parse the user's intent and produce a structured plan file.
2. During reasoning sessions, persist decisions incrementally as they stabilise.
3. Group requirements by verifiable outcome — never by file or internal implementation step.
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

### N.M [logical area or capability]
- [ ] REQ-N.M.1 [short outcome label]: [what must be true after this is done — one verifiable outcome, any number of files]
- [ ] REQ-N.M.2 [short outcome label]: [what must be true after this is done]

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
- One REQ-ID = one verifiable outcome. It may touch any number of files.
- A REQ-ID is the right size when it can be verified as a unit: either it works end-to-end or it does not.
- Do not list file paths in the REQ-ID definition. File paths belong in implementation evidence after execution.
- Never renumber an existing REQ-ID later to make the list look cleaner.
- The section heading (`N.M`) describes a logical area or capability, not a file name.

## Sizing a REQ-ID correctly

Ask: *"Can this be verified as a single observable outcome?"*

- YES → it is one REQ-ID regardless of how many files it touches.
- NO, it contains two independently verifiable outcomes → split into two REQ-IDs.
- NO, it is one step inside a larger outcome → merge it into the parent REQ-ID.

A phase with 8–10 well-sized REQ-IDs is healthy. If a phase has more than 12, check whether any REQ-IDs are really sub-steps of the same outcome before adding more.

## Anti-patterns

**❌ File-driven splitting** — one REQ-ID per file touched:
```
- [ ] REQ-1.1.1 run-block.mjs: supportare più --block
- [ ] REQ-1.1.2 run-block.mjs: validare scenari contro l'unione dei blocchi
- [ ] REQ-1.1.3 scenario-catalog.json: dichiarare adapter e gate obbligatori
- [ ] REQ-1.1.4 validate-scenarios.mjs: validare adapter, target, gate e report
```

**✅ Behavior-driven grouping** — one REQ-ID per verifiable outcome, any files:
```
- [ ] REQ-1.1 Multi-block selection: run-block.mjs accetta più --block, valida
  scenari contro l'unione dei blocchi e garantisce un risultato esplicito per
  ogni scenario selezionato; scenario-catalog.json dichiara adapter e gate
  obbligatori; validate-scenarios.mjs valida adapter, target, gate e report.
```

**❌ Function-level atomicity** — splitting a single feature by internal step:
```
- [ ] REQ-2.1.1 validare input
- [ ] REQ-2.1.2 normalizzare formato
- [ ] REQ-2.1.3 scrivere output
```

**✅ Feature-level atomicity** — one REQ-ID for the whole verifiable behavior:
```
- [ ] REQ-2.1 Input pipeline: valida, normalizza e scrive l'output nel formato
  richiesto; input malformati producono un errore esplicito.
```

**❌ Splitting implementation from its direct documentation:**
```
- [ ] REQ-3.1.1 aggiornare funzione X in run-block.mjs
- [ ] REQ-3.1.2 aggiornare SKILL.md con la sintassi di X
```

**✅ Same REQ-ID covers implementation and its direct documentation:**
```
- [ ] REQ-3.1 Syntax update: aggiorna funzione X e documenta la sintassi
  reale in SKILL.md.
```

**Smell test:** se due REQ-ID adiacenti diventano sempre `[x]` insieme e nessuno dei due ha senso verificato da solo, sono lo stesso REQ-ID scritto su due righe.

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

- Do not create one REQ-ID per file touched — group by verifiable outcome.
- Do not use file names as section headings — use logical areas or capabilities.
- Do not add implementation requirements not supported by the user's request, confirmed decisions, or necessary dependencies from authoritative evidence.
- Do not start implementing code.
- Do not run triage logic — that is `plan-triage`'s responsibility.
- Do not set `RETRIAGE: NO` from draft.
