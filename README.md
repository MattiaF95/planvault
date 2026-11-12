# PLANVAULT

`planvault` is an AI agent skill for creating, updating, and executing reliable
technical implementation plans without losing requirements, decisions, or
execution state.

The plan is more than project documentation. Once its decisions are confirmed
and its requirements are executable, it also becomes the agent's operational
prompt: it tells the agent what to do, what to verify, where to record evidence,
and when the work is actually complete.

## What PlanVault solves

PlanVault gives long-running AI coding sessions a stable workflow for software
implementation, code changes, testing, debugging, and technical delivery.
It helps the agent to:

- keep every requirement under a stable REQ-ID;
- preserve confirmed architectural decisions;
- avoid silently skipping tasks;
- resume from the correct open task after an interruption;
- separate product bugs, test errors, and environment blockers;
- prevent false completion without implementation and verification evidence.

## Skill structure

```text
skill/planvault/
├── SKILL.md
└── references/
    ├── plan-draft.md
    ├── plan-triage.md
    ├── plan-update.md
    └── plan-execute.md
```

`SKILL.md` is the entry point. The reference files define how an AI agent
drafts, reviews, updates, and executes a structured technical plan.

## Plan lifecycle

```text
draft → triage → update when needed → execute → verify → close → archive/delete
```

### Planning decisions

PlanVault does not require a plan for small, self-contained tasks. A new plan
is written only on an explicit user request.

From its first draft, each plan defines:

- The user's subagent limit, safe parallel work per phase, and justified model
  and effort proposals, including a sequential fallback.
- A verification strategy selecting unit, integration, functional black-box,
  and other relevant checks, with reasons for exclusions.
- Exact documentation updates and their verification, plus the final ARCHIVE
  or explicitly chosen DELETE action, artifact paths, evidence, and recovery.

The archive name and location belong to the repository where the plan was
created. An existing archive destination must be confirmed unless already
approved. If no archive exists, the agent asks where to archive and keeps the
destination unresolved. It never creates a new archive directory, including a
per-plan subdirectory, without explicit authorization.

The user limit counts simultaneously active subagents, including reviewers,
and excludes the coordinator. Safe concurrency is assessed separately from
runtime capacity, using dependencies, file ownership, shared contracts, and
test resources. Effective concurrency is the minimum of the user limit,
runtime capacity, and currently safe independent work. The coordinator owns
plan updates, integration, and commits; workers return changes and evidence.
Model and effort proposals are checked against actual availability before launch.

Test categories are evaluated rather than required indiscriminately. Each
selected check records its scenario, command or procedure, expected result,
environment, and evidence limits. Excluded categories have a brief rationale.

Documentation is identified during drafting by exact path, required change,
checkpoint, and verification criterion. If no updates are needed, the plan
states why; final verification checks the recorded documents for consistency.

### Draft

A new plan starts with:

```text
ANALYSIS: IN_PROGRESS | COMPLETE
RETRIAGE: REQUIRED
```

The plan records the objective, constraints, confirmed decisions, phases,
REQ-IDs, commit order, and measurable exit criteria.

Each REQ-ID must describe one verifiable outcome, not one file or one internal
implementation step.

```markdown
- [ ] REQ-2.1.1 Token validation: reject expired tokens and verify the behavior
  with the required test.
```

### Triage

Triage validates the plan structure and decides whether the plan should remain
in one file or use the three core files:

```text
spec.md   what and why
plan.md   how and in which order
tasks.md  REQ-ID status and evidence
```

Only triage can change:

```text
RETRIAGE: REQUIRED → NO
```

Execution is allowed only when:

```text
ANALYSIS: COMPLETE
RETRIAGE: NO
```

These values make the open work executable. They do not mean that the plan is
complete.

### Update

When requirements or decisions change, PlanVault preserves traceability:

- only the affected plan content is changed;
- existing REQ-IDs are not silently renumbered or removed;
- a completed requirement is reopened when its meaning changes;
- conflicts are reported instead of being resolved silently.

### Execute

Any user instruction that means execute, implement, apply, carry out, complete,
run, or otherwise perform the work described by the plan is an execution
request. The exact wording does not matter.

The AI agent must then use the plan as an operational prompt. It validates the
plan, selects the requested scope, starts from the first open REQ-ID, implements
the complete outcome, runs the required verification, records evidence, and
continues with the next open REQ-ID.

If no narrower scope is stated, the scope is all remaining active REQ-IDs.
A successful preflight is not an execution result: when prerequisites pass,
execution must begin.

## Bounded execution

The execution loop is bounded. The agent reads the plan once at the start of an
execution session and creates an in-memory ordered cursor for the selected
REQ-IDs, allowing validated batches of independent work within phase boundaries.
It does not reread or rewrite the full plan between every task.

The plan is updated at meaningful checkpoints: after a REQ-ID is completed or
blocked, when the execution cursor advances, and at the final scope audit. The
same REQ-ID is not repeated in one session unless verification failed and a
concrete fix was applied, or a rerun was explicitly requested.

The agent stops when the requested scope is complete, a real blocker prevents
safe progress, a plan conflict requires an update, or an essential decision
cannot be resolved. If the cursor cannot advance after one execution cycle, the
agent reports the blocker instead of retrying forever.

## Current execution state

Every plan must show its current operational state near the beginning, using
these exact field names:

```markdown
- `PLAN_STATUS: OPEN`
- `EXECUTION_READINESS: READY`
- `EXECUTION_SCOPE: FULL_PLAN`
- `CURRENT_REQ: REQ-2.1.1`
- `NEXT_ACTION: implement and verify REQ-2.1.1`
- `COMPLETION_ALLOWED: NO`
```

Allowed values are:

```text
PLAN_STATUS: OPEN | IN_PROGRESS | BLOCKED | COMPLETE
EXECUTION_READINESS: NOT_READY | READY | BLOCKED
```

REQ-ID states are:

```text
[ ] NOT_STARTED
[~] IN_PROGRESS
[!] BLOCKED
[x] COMPLETE
```

An in-progress, blocked, partial, planned, skipped, or unverified requirement
is not complete. A blocker must include the blocker, evidence, required
resolution, and resume action.

The execution log is historical evidence. The current state is authoritative in
the execution header, the REQ-ID checklist, and the exit criteria.

## Completion rules

The plan can be marked complete only when:

- every active REQ-ID in every phase is `[x]`;
- every requirement has implementation, verification, and evidence;
- every exit criterion is explicitly verified;
- no blocker, review, or follow-up action remains;
- `PLAN_STATUS` is `COMPLETE` and `COMPLETION_ALLOWED` is `YES`.

Passing one phase, scenario, test, or group of tasks does not complete the
whole plan.

A verified plan is first closed, then archived or deleted according to its
recorded, authorized retention action. The plan keeps pre-closure exit criteria
separate from post-closure delivery conditions: cleanup is never a requirement
that must pass before the plan can close.

Finalization uses `PENDING | DONE`. Final delivery requires verified cleanup;
marking the plan COMPLETE alone is insufficient. Archiving moves only the
listed artifacts to the confirmed destination, preserves links, updates affected
references, and never overwrites an existing archive. Deletion requires an
explicit user choice and a durable closure/evidence record outside the deleted
artifacts. Completing only part of a plan never triggers full-plan cleanup.

Failed finalization leaves verified implementation COMPLETE and finalization
PENDING, with an error and recovery action. The already-authorized cleanup can
resume without reopening requirements or repeating implementation.

## Evidence

The original requirement text must remain in the plan. Update its existing
status marker only after the outcome and required verification pass:

```markdown
- [x] REQ-3.1 Syntax update: support and document the new syntax.
```

Keep evidence in verification artifacts and summarize it in the delivery
response. If the plan explicitly defines an evidence field, update that field
concisely; do not append duplicate tasks or evidence prose beneath every row.

A changed file alone is not proof. Evidence must show that the requested
outcome was implemented and verified.

## Installation

Install the skill directly from the `skill/planvault` directory in this
repository with:

```bash
npx skills add https://github.com/MattiaF95/planvault/tree/main/skill/planvault
```

The source directory is available on
[GitHub](https://github.com/MattiaF95/planvault/tree/main/skill/planvault) and
must contain `SKILL.md` and the `references/` directory.

## License

MIT. See [LICENSE](LICENSE).
