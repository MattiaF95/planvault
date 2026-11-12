# plan-draft

## Responsibilities

1. Draft only after the `planvault` authorization gate confirms an explicit request to create a plan.
2. Within the authorized scope, persist decisions incrementally as they stabilise.
3. Group requirements by verifiable outcome — never by file or internal implementation step.
4. Never pre-empt triage: draft as a single file; triage decides later whether a structural split is needed.

## Plan file format

```markdown
# Plan: [title]
ANALYSIS: [COMPLETE | IN_PROGRESS]
RETRIAGE: REQUIRED

## Agent execution header
- `PLAN_STATUS: OPEN`
- `EXECUTION_READINESS: NOT_READY`
- `EXECUTION_SCOPE: FULL_PLAN`
- `CURRENT_REQ: none`
- `NEXT_ACTION: complete triage, then execute the first open REQ-ID`
- `COMPLETION_ALLOWED: NO`

Use these exact field names and values. Do not replace the header with a custom
prose status.

## 1. Objective
[What and why. Keep it to 3–5 sentences.]

## 2. Context and constraints
- Authoritative evidence: [files, reports, prior fixes already analysed]
- Confirmed decisions: [decisions that should not be reopened without new evidence]
- Out of scope: [explicitly excluded from this plan]

The plan must contain enough repository orientation for an agent without
session memory to locate the relevant functionality. Keep one authoritative
context location. In a 3-core-file plan, link to that location instead of
duplicating the same context across core files.

Distinguish repository orientation from execution strategy: explain where the
relevant functionality lives, then explain the ordered changes needed to
achieve the objective. Commands, expected observations, and acceptance proofs
belong in the execution and validation sections.

## 3. Phase N — [name]

### N.M [logical area or capability]
- [ ] REQ-N.M.1 [short outcome label]: [what must be true after this is done — one verifiable outcome, any number of files]
- [ ] REQ-N.M.2 [short outcome label]: [what must be true after this is done]

## Agent strategy
- User limit: [maximum simultaneously active subagents, excluding the coordinator;
  source of the user's preference, or explicit sequential fallback]
- Runtime capacity: [verified capacity or unknown; recheck at execution]
- Per phase or independent batch: [REQ-IDs, dependencies, safe concurrent count,
  file ownership, shared contracts/services/test resources, integration order]
- Role proposals: [scope, proposed available model identifier, effort, rationale,
  and any capability still to verify; or no subagents needed with reason]

## Verification strategy
- Per phase and for the overall objective: [unit, integration, functional
  black-box, and other relevant checks selected or excluded with a brief reason]
- For each selected check: [REQ-ID/outcome, scenario including relevant errors
  and regressions, command/procedure, expected result, environment and evidence]
- Evidence limits: [real, mock, provider/runtime dependencies, unavailable checks]

## Documentation updates
| Document path | Required change | Phase or closure | Verification |
|---|---|---|---|
| [exact path] | [content to align] | [checkpoint] | [consistency criterion] |

If no documentation updates are needed, state that explicitly with a reason.
Keep direct implementation documentation within its outcome's REQ-ID.

## Closure and retention
- Final action: [ARCHIVE or DELETE; propose ARCHIVE unless the user chooses DELETE]
- Artifacts: [exact plan/core/support files; never an unspecified parent folder]
- Destination: [existing archive directory in the repository where the plan was
  created, confirmed by the user, plus exact target artifact paths; not applicable
  for DELETE]
- Retained evidence: [durable location outside files to delete, or archive contents]
- References to update: [exact affected indexes/links, or none with reason]
- Authorization: [user instruction covering this final action; pending if unresolved]
- Finalization status: PENDING [allowed values: PENDING | DONE; initially PENDING]
- Recovery: [how to resume a failed move/delete without reopening completed work]

## Commit order
1. [phase-level commit message for phase 1]
2. [phase-level commit message for phase 2]

Each phase must define one atomic commit outcome. The commit is created only
after all REQ-IDs, verification, review, and evidence for that phase are
complete.

## Exit criteria (required before closure)
- [binary, measurable condition — not "it works"]
- [observable proof that the overall objective was achieved]

## Final delivery conditions (checked after closure)
- Recorded retention action verified; Finalization status: DONE.
- [any required finalization commit or reference/evidence checks]

## Execution instructions
- The plan is the operational prompt and source of truth during execution.
- When an execution intent is received, execute the requested scope; if no
  narrower scope is stated, execute all remaining active REQ-IDs.
- Start with the first active incomplete REQ-ID in plan order after lifecycle
  and structure validation succeed.
- Do not stop after analysis, inventory, preflight, or validation when execution
  prerequisites pass.
- Do not skip an active REQ-ID or mark it complete without implementation,
  required verification, review, and evidence.
- If work is blocked, record the blocker, evidence, required resolution, and
  resume action. The REQ-ID, phase, and plan remain incomplete.
- The plan can be complete only when every active REQ-ID and every exit
  criterion is complete and evidenced.
- Every phase must define an observable result that advances the objective and
  a verification that demonstrates that result. Final validation must also
  demonstrate the overall objective, not merely completed REQ-IDs.
- After full verification and documentation audit, close the plan, then apply
  and verify the recorded retention action. Final delivery requires completed
  finalization; a failure leaves a closed plan with pending recovery.
- For risky commands, migrations, or operations that may fail partially,
  document a safe retry or recovery action.
```

## Planning decision rules

### Agent strategy

Ask for the user's maximum simultaneous subagents when it is not already known.
Continue independent planning while waiting. If no answer is available, record
zero subagents as a conservative sequential fallback, not as a user preference.
Zero is a valid decision even when a higher limit is authorized.

Compute the safe concurrency ceiling independently of runtime capacity and the
user limit; distinguish that ceiling from the chosen worker count and the
effective launch count. A runtime capacity of zero does not imply that no
independent work could be delegated in another environment.

For each phase/batch, derive safe parallelism from independent outcomes and
non-conflicting ownership. Different files alone do not prove independence:
shared contracts, generated outputs, databases, services, fixtures, and test
resources can conflict. Record dependency barriers and integration order.

Effective concurrency is the minimum of the user limit, runtime capacity, and
safe independent work available now. Count all active subagents against that
limit, including reviewers. The coordinator owns plan writes, integration, and
Git operations; workers own only their assigned scope. Sequential execution
remains valid when delegation is unavailable or not authorized.

Propose a model and effort for each useful role with a task-specific reason.
Use identifiers and effort levels supported by the current environment; if
unverified, label them as proposals to verify before launch. Do not invent a
fixed model catalogue or silently substitute an unavailable model. A simpler
available configuration or sequential fallback must remain within user choices.

### Verification, documentation, and retention

Evaluate test categories, rather than requiring every category for every plan.
Unit tests suit isolated logic; integration tests suit contracts between parts;
functional black-box checks exercise observable behavior through public
interfaces without depending on implementation internals. Select proportionate
checks and explain exclusions. A mock or static check must not stand in for an
unverified runtime/provider result.

Inspect repository documentation and identify exact affected paths during
initial drafting, including final cross-document consistency checks. Do not
leave a generic "update docs later" task. New impacts discovered during execution
must be recorded through plan-update within the authorized scope.

Inspect retention conventions and existing archive directories in the repository
where the plan was created. Resolve paths against that repository, never the
skill installation or an unrelated working directory. There is no universal
archive folder name or location.

Propose ARCHIVE by default and record concrete artifact paths from the start.
Ask the user to confirm the discovered archive directory unless their earlier
instruction already clearly approves that exact destination. If none exists,
ask where to archive; do not invent a folder name or create directories on your
own. Creating a new archive directory, including a per-plan subdirectory inside an
existing archive, requires explicit user authorization.
While awaiting the answer, continue independent planning and keep the destination
unresolved. Do not mark ANALYSIS: COMPLETE until the destination is resolved.

DELETE requires the user's explicit choice. Record authorization for finalization;
an explicit execution request covering a disclosed ARCHIVE action authorizes the
local move only when the destination has also been confirmed. It does not imply
a DELETE choice, a new directory, or external actions. If the action or destination
is unresolved, keep ANALYSIS: IN_PROGRESS.

The closure procedure must first close the verified plan, then apply its recorded
retention action and verify the result. Include this procedure in the plan's
execution instructions. Final delivery requires completed finalization; the
retention action must not be a REQ-ID or exit criterion required before closure.
Keep it under separate final delivery conditions to avoid a circular gate.
Use exactly PENDING and DONE for Finalization status, not COMPLETE.
For DELETE, preserve closure and verification evidence outside the artifacts
before removing them. For ARCHIVE, keep all core/support files together and
preserve internal links; update external references as specified.

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
- Do not list file paths in the REQ-ID definition. Add implementation evidence only when the plan explicitly requires it, and use an existing evidence field rather than appending paths beneath task rows.
- Never renumber an existing REQ-ID later to make the list look cleaner.
- The section heading (`N.M`) describes a logical area or capability, not a file name.

## Sizing a REQ-ID correctly

Ask: *"Can this be verified as a single observable outcome?"*

- YES → it is one REQ-ID regardless of how many files it touches.
- NO, it contains two independently verifiable outcomes → split into two REQ-IDs.
- NO, it is one step inside a larger outcome → merge it into the parent REQ-ID.

A phase with 8–10 well-sized REQ-IDs is healthy. If a phase has more than 10,
review whether REQ-IDs are really sub-steps of the same outcome before adding
more; triage may split the phase when independent density remains high.

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
