# planvault

> A generic, agent-agnostic skill system for structured plan creation, triage, update and execution — preventing context drift and specification loss in long agentic sessions.

## Problem

AI coding agents (GPT, Claude, Gemini, etc.) tend to:
- **Summarise** a detailed plan instead of implementing it fully
- **Skip** requirements silently, especially mid-context in long sessions
- **Re-analyse** already-confirmed context, wasting tokens on redundant checks
- **Lose decisions** made early in a long reasoning session before the plan is written

planvault solves this with a router skill that dispatches to the right sub-skill for each phase of the plan lifecycle.

## How it works

```
planvault/
  skills/
    plan-master/        ← router: detects current phase, dispatches to sub-skill
    plan-draft/         ← creates first plan draft with incremental checkpoints
    plan-triage/        ← decides single-file vs 3-file split; creates review template if needed
    plan-update/        ← updates existing plan; handles conflicts
    plan-execute/       ← validates structure, executes current phase REQ-IDs
      references/
        review-audit-template.md  ← optional; created by triage if a repetitive review pattern is detected
```

## Installation

### Codex CLI

Copy the `skills/` folder into your project:

```bash
cp -r planvault/skills/ .codex/skills/
```

Or symlink for shared use across projects:

```bash
ln -s /path/to/planvault/skills/ .codex/skills/planvault
```

### Claude Code

```bash
cp -r planvault/skills/ .claude/skills/
```

## Usage

Simply mention the skill by name or describe what you need:

```
# Explicit trigger
$plan-master: I need to plan a pagination refactor across 5 components

# Implicit trigger (agent auto-detects)
"Let's plan the implementation of feature X"
"Update the plan: we're dropping approach A, going with B"
"Execute phase 2 of the plan"
```

The router (`plan-master`) detects the current state and loads only the relevant sub-skill.

## Core principles

- **Atomic requirements (REQ-IDs)**: every bullet in the plan becomes a traceable `REQ-{phase}.{section}.{n}` — the unit of tracking is the *requirement*, not the file or function
- **Minimal state markers**: `[x]`/`[ ]` + file:line reference, no prose — zero token waste on status updates
- **Trust boundary**: if the plan declares `ANALYSIS: COMPLETE` with cited evidence, the agent is forbidden from re-analysing
- **Incremental checkpoints**: the plan file is updated as decisions stabilise during reasoning, not only at the end
- **Conditional review**: the review template is created by triage only if a repetitive review pattern is detected; each task decides whether to invoke it
- **Execute = validate + run, not decide**: plan architecture decisions belong to triage, not execution

## File format

### Single-file plan

Used when total REQ-IDs ≤ 25–30 and phases are well-isolated.

```markdown
# Plan: [title]

## Context and constraints
...

## Phase 1 — [name]

### 1.1 [component]
- [ ] REQ-1.1.1 [path/to/file.ts]: description of atomic change
- [ ] REQ-1.1.2 [path/to/file.ts:34]: description

## Phase 2 — [name]
...
```

### 3-file split

Used when total REQ-IDs > 25–30, or multiple unrelated features/domains.

```
spec.md     ← what and why (objectives, constraints, out-of-scope)
plan.md     ← how (architecture, phases, dependencies)
tasks.md    ← REQ-ID checklist with [ ]/[x] markers and file:line refs
```

## License

MIT — see [LICENSE](LICENSE)
