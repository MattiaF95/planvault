# planvault

AI coding agents don't lose focus in a single exchange. They lose it across 30.

You start with a clear plan, five phases, twenty requirements. Halfway through phase 3 the agent quietly drops a constraint you established in phase 1, rewrites something you already confirmed, or silently skips two REQ-IDs because the context window is getting crowded. By the time you notice, the damage is already in the diff.

planvault is a provider-neutral agent skill that structures the entire lifecycle of a technical plan — from the first reasoning exchange to the last committed REQ-ID — so that nothing gets lost, reinterpreted, or silently skipped.

---

## The problem in plain terms

| What happens | Why it happens |
|---|---|
| Agent summarises the plan instead of following it | No atomic requirement tracking — agent treats bullets as suggestions |
| Requirements get silently skipped mid-execution | No trust boundary — agent re-reads and re-weighs already-confirmed decisions |
| Decisions from early in the session are forgotten | Plan written at the end from degraded memory, not incrementally |
| Agent re-analyses things you already confirmed | No `ANALYSIS: COMPLETE` signal — agent keeps reopening closed questions |
| Review steps get inconsistent across phases | No shared template — each phase defines its own criteria differently |

---

## Structured plan vs. no structure

| | Standard multi-step plan | planvault |
|---|---|---|
| **Requirements** | Prose bullets, no IDs | Atomic REQ-IDs with file:line refs |
| **Status tracking** | Agent memory or manual notes | `[ ]` / `[x]` inline in the plan file |
| **Analysis boundary** | Reopened every session | `ANALYSIS: COMPLETE` — cited evidence is ground truth |
| **Plan persistence** | Written at the end from memory | Incremental checkpoints as decisions stabilise |
| **File structure** | Single doc or ad-hoc split | Single file or enforced 3-file split decided by triage |
| **Review consistency** | Per-phase, per-agent improvisation | Shared `review-audit-template` if repetitive pattern detected |
| **Conflict handling** | Silent overwrite | Explicit stop + user confirmation before any change |
| **Phase completion** | "Looks done" | Every REQ-ID has `[x]` + `file:line` diff ref — no exceptions |

---

## How it works

planvault is a single skill with one entry point (`plan-master`) that routes to the correct sub-skill for each phase of the plan lifecycle:

```
plan-master        router — detects phase, loads sub-skill
  ├── plan-draft      first draft + incremental checkpoints during reasoning
  ├── plan-triage     single-file vs 3-file split; creates review template if needed
  ├── plan-update     modify plan with conflict detection; never silent overwrites
  └── plan-execute    validate structure, execute REQ-IDs, enforce trust boundary
```

Each sub-skill has a single responsibility. `plan-execute` validates and runs — it never makes architectural decisions about the plan. Those belong to `plan-triage`.

---

## Installation

### OpenAI Codex CLI

```bash
# Project scope
mkdir -p .agents/skills
cp -r skill/planvault .agents/skills/

# User scope (available across all projects)
mkdir -p ~/.agents/skills
cp -r skill/planvault ~/.agents/skills/
```

Or via `$skill-installer` if planvault is listed in a curated registry:

```
$skill-installer planvault
```

### Claude Code

```bash
# Project scope
mkdir -p .claude/skills
cp -r skill/planvault .claude/skills/

# Personal scope (available across all projects)
mkdir -p ~/.claude/skills
cp -r skill/planvault ~/.claude/skills/
```

### Gemini CLI

```bash
mkdir -p .agents/skills
cp -r skill/planvault .agents/skills/
```

---

## Usage

```
# Explicit invocation
$plan-master        ← Codex CLI / Gemini CLI
/plan-master        ← Claude Code

# Implicit invocation (agent picks it up from description)
"Let's plan the implementation of feature X"
"Update the plan: we're dropping approach A"
"Execute phase 2"
```

`plan-master` detects the current state (no plan yet / plan exists / plan needs update / ready to execute) and loads only the relevant sub-skill.

---

## Core principles

- **Atomic REQ-IDs** — every requirement is `REQ-{phase}.{section}.{n}`. The unit of tracking is the requirement, not the file.
- **Trust boundary** — `ANALYSIS: COMPLETE` means cited evidence is ground truth. The agent is forbidden from reopening it.
- **Incremental checkpoints** — decisions are written to the plan file as they stabilise, not at the end of a long session.
- **Triage owns structure** — single-file vs 3-file split, and whether a review template is needed, are decided once by `plan-triage`. `plan-execute` enforces the decision, never makes it.
- **Explicit conflict resolution** — `plan-update` stops and asks before overwriting any conflicting content.
- **Phase COMPLETE = verifiable** — a phase is done only when every REQ-ID has `[x]` and a `file:line` diff reference.

---

## Repository structure

```
planvault/
├── README.md
├── LICENSE                          MIT
└── skill/planvault/
    ├── SKILL.md                     Router + phase detection
    └── references/
        ├── plan-draft.md            First draft + checkpoint rules
        ├── plan-triage.md           Structure decision + review pattern detection
        ├── plan-update.md           Conflict-safe plan updates
        └── plan-execute.md          Structure validation + REQ-ID execution
```

`review-audit-template.md` is **not** a skill file. It is generated by `plan-triage` at plan time when a repetitive review pattern is detected across phases, and saved alongside the plan. Each task in `plan-execute` decides whether to invoke it based on phase context.

---

## Agent compatibility

| Agent | Skill location | Invocation |
|---|---|---|
| OpenAI Codex CLI | `.agents/skills/planvault/` | `$plan-master` |
| Claude Code | `.claude/skills/planvault/` | `/plan-master` |
| Gemini CLI | `.agents/skills/planvault/` | `activate plan-master` |

---

## License

MIT — see [LICENSE](LICENSE)
