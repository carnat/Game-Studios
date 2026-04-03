# Codebase Explanation — Claude Code Game Studios

This document is a comprehensive guide to the structure, components, and inner workings of the **Claude Code Game Studios** repository. Read it to understand how each part of the framework fits together and why it exists.

---

## What This Repository Is

**Claude Code Game Studios** is a *project template* — not a game, not a game engine, and not a standalone application. It is a structured AI-agent framework designed to be cloned as the foundation for a new game project. Once you run `claude` inside the clone, the framework activates and turns a single Claude Code session into a coordinated team of 48 specialized AI agents.

The repository ships with **zero game-specific code**. Every directory under `src/`, `assets/`, `design/`, and `tests/` is intentionally empty — they exist only to anchor the path-scoped coding rules and to define where your game code will live.

---

## Top-Level Layout

```
/
├── CLAUDE.md                          # Master agent configuration (read by Claude Code at startup)
├── README.md                          # User-facing project overview
├── UPGRADING.md                       # Migration guide for version upgrades
├── LICENSE                            # MIT License
├── .gitignore
│
├── .claude/                           # The entire agent framework lives here
│   ├── settings.json                  # Platform configuration: hooks, permissions, status line
│   ├── statusline.sh                  # Terminal status line script
│   ├── agents/                        # 48 agent definition files
│   ├── skills/                        # 37 slash-command workflows
│   ├── hooks/                         # 8 automated validation scripts
│   ├── rules/                         # 11 path-scoped coding-standard rules
│   └── docs/                          # Internal framework documentation & templates
│
├── docs/                              # Project-level documentation and guides
│   ├── COLLABORATIVE-DESIGN-PRINCIPLE.md
│   ├── WORKFLOW-GUIDE.md
│   ├── examples/                      # Annotated session transcripts
│   └── engine-reference/              # Version-pinned API snapshots (Godot / Unity / Unreal)
│
└── production/                        # Runtime-only session state (gitignored)
    ├── session-state/
    └── session-logs/
```

---

## Component Deep-Dive

### 1. `CLAUDE.md` — Master Configuration

`CLAUDE.md` is the entry point that Claude Code reads at the start of every session. It uses the `@<path>` import syntax to pull in several other files:

- `@.claude/docs/directory-structure.md` — explains where game files should live
- `@docs/engine-reference/godot/VERSION.md` — the pinned engine version (swap for Unity/Unreal)
- `@.claude/docs/technical-preferences.md` — filled in by the user with their stack choices
- `@.claude/docs/coordination-rules.md` — delegation protocol all agents follow
- `@.claude/docs/coding-standards.md` — links to path-scoped rules
- `@.claude/docs/context-management.md` — strategy for managing long sessions

The file also embeds the **collaboration protocol** inline: agents must ask before writing, must show drafts before finalizing, and may not commit without user instruction. This protocol is the single most important behavioral constraint in the whole system.

---

### 2. `.claude/settings.json` — Platform Configuration

This JSON file configures Claude Code itself (not the agents). It has three sections:

#### `statusLine`
Runs `.claude/statusline.sh` to display the current production stage (e.g., `[PRE-ALPHA]`) in the terminal status bar.

#### `permissions`
Defines which shell commands agents may run **without asking**:

| Allow (no prompt) | Deny (blocked outright) |
|---|---|
| `git status`, `git diff`, `git log` | `rm -rf`, `git push --force`, `git reset --hard` |
| `ls`, `dir` | `sudo`, `chmod 777` |
| `python -m json.tool` (JSON validation) | Reading or writing `.env` files |
| `python -m pytest` (test runs) | |

#### `hooks`
Wires the 8 hook scripts to Claude Code lifecycle events. See §5 (Hooks) for details.

---

### 3. `.claude/agents/` — The 48 Agents

Each file is a Markdown document with a YAML front-matter block followed by a natural-language system prompt. Claude Code reads these when an agent is invoked (either directly by the user or delegated to by another agent).

#### Tier Structure

```
Tier 1 — Directors (Opus model)         [3 agents]
  creative-director   technical-director   producer

Tier 2 — Department Leads (Sonnet)      [8 agents]
  game-designer       lead-programmer      art-director
  audio-director      narrative-director   qa-lead
  release-manager     localization-lead

Tier 3 — Specialists (Sonnet/Haiku)     [23 agents]
  gameplay-programmer   engine-programmer   ai-programmer
  network-programmer    tools-programmer    ui-programmer
  systems-designer      level-designer      economy-designer
  technical-artist      sound-designer      writer
  world-builder         ux-designer         prototyper
  performance-analyst   devops-engineer     analytics-engineer
  security-engineer     qa-tester           accessibility-specialist
  live-ops-designer     community-manager

Engine Specialists                       [14 agents]
  Godot 4:   godot-specialist, godot-gdscript-specialist,
             godot-shader-specialist, godot-gdextension-specialist
  Unity:     unity-specialist, unity-dots-specialist,
             unity-shader-specialist, unity-ui-specialist,
             unity-addressables-specialist
  Unreal 5:  unreal-specialist, ue-gas-specialist,
             ue-blueprint-specialist, ue-replication-specialist,
             ue-umg-specialist
```

#### Coordination Rules

All agents follow the same delegation model encoded in `coordination-rules.md`:

1. **Vertical delegation** — directors → leads → specialists (never skip tiers)
2. **Horizontal consultation** — same-tier agents may consult, not decide
3. **Conflict resolution** — escalates to the shared parent director
4. **Change propagation** — cross-department changes are coordinated by `producer`
5. **Domain boundaries** — agents never touch files outside their domain without explicit delegation

---

### 4. `.claude/skills/` — The 37 Slash Commands

Each subdirectory under `skills/` maps to one `/command` available in Claude Code. A skill directory typically contains a single Markdown file (`skill.md` or `prompt.md`) that instructs Claude how to run the workflow when the command is invoked.

#### Full Skill List by Category

| Category | Skills |
|---|---|
| **Reviews & Analysis** | `/design-review` `/code-review` `/balance-check` `/asset-audit` `/scope-check` `/perf-profile` `/tech-debt` |
| **Production** | `/sprint-plan` `/milestone-review` `/estimate` `/retrospective` `/bug-report` |
| **Project Management** | `/start` `/project-stage-detect` `/reverse-document` `/gate-check` `/map-systems` `/design-system` `/architecture-decision` |
| **Release** | `/release-checklist` `/launch-checklist` `/changelog` `/patch-notes` `/hotfix` |
| **Creative** | `/brainstorm` `/playtest-report` `/prototype` `/onboard` `/localize` `/setup-engine` |
| **Team Orchestration** | `/team-combat` `/team-narrative` `/team-ui` `/team-release` `/team-polish` `/team-audio` `/team-level` |

**Team skills** are the most powerful: `/team-combat`, for example, spins up game-designer, gameplay-programmer, ai-programmer, technical-artist, sound-designer, and qa-tester in sequence to design, implement, and validate a full combat feature in a single session.

---

### 5. `.claude/hooks/` — 8 Automated Validation Scripts

All hooks are POSIX-compatible Bash scripts. They are wired to Claude Code lifecycle events in `settings.json`. Every hook fails gracefully if optional tools (`jq`, `python`) are missing.

| File | Trigger | Purpose |
|---|---|---|
| `session-start.sh` | `SessionStart` | Prints active sprint, recent git log, and current task from `production/session-state/active.md` |
| `detect-gaps.sh` | `SessionStart` | Detects a fresh project (no design docs, no src files) and suggests `/start`; warns if prototypes exist without a GDD |
| `validate-commit.sh` | `PreToolUse` on Bash | Intercepts `git commit` commands: checks for hardcoded magic numbers, malformed TODO format, invalid JSON in `assets/data/`, and missing required sections in GDD files |
| `validate-push.sh` | `PreToolUse` on Bash | Intercepts `git push` commands: warns when pushing directly to `main` or `master` |
| `validate-assets.sh` | `PostToolUse` on Write/Edit | After any file write inside `assets/`, validates naming conventions (`snake_case`) and JSON structure |
| `pre-compact.sh` | `PreCompact` | Before Claude compresses its context, reads `production/session-state/active.md` and prepends a progress note so context is not lost |
| `session-stop.sh` | `Stop` | Appends a timestamped accomplishment entry to `production/session-logs/` |
| `log-agent.sh` | `SubagentStart` | Writes a timestamped entry to the agent audit log every time a subagent is invoked |

---

### 6. `.claude/rules/` — 11 Path-Scoped Coding Standards

Rules are Markdown files that Claude Code loads automatically when the user edits a file matching the rule's path glob. They are *not* enforced by a linter — they are behavioral instructions that guide agent outputs.

| Rule File | Path Pattern | Key Constraints |
|---|---|---|
| `gameplay-code.md` | `src/gameplay/**` | No hardcoded values, always use delta time, no direct UI references |
| `engine-code.md` | `src/core/**` | Zero allocations in hot paths, thread-safe, stable public API |
| `ai-code.md` | `src/ai/**` | Performance budgets, data-driven parameters, debuggability |
| `network-code.md` | `src/networking/**` | Server-authoritative, versioned message schemas, security review required |
| `ui-code.md` | `src/ui/**` | No game state ownership, localization-ready, accessibility |
| `shader-code.md` | `src/shaders/**` | Performance-first, readable constants, comment complex math |
| `prototype-code.md` | `prototypes/**` | Relaxed standards, `README.md` required, hypothesis documented |
| `design-docs.md` | `design/gdd/**` | 8-section GDD required, formula format, edge cases covered |
| `test-standards.md` | `tests/**` | Naming conventions, coverage requirements, fixture patterns |
| `data-files.md` | `assets/data/**` | JSON structure standards, schema validation |
| `narrative.md` | `design/narrative/**` | Tone consistency, translation-ready strings, character voice |

---

### 7. `.claude/docs/` — Internal Framework Documentation

This directory documents the framework itself (not the user's game). Key files:

| File | Contents |
|---|---|
| `quick-start.md` | Practical usage guide: how to invoke agents, run skills, and navigate the workflow |
| `agent-roster.md` | Full table of all 48 agents with their domains, models, and capabilities |
| `agent-coordination-map.md` | Delegation and escalation paths between every agent pair |
| `setup-requirements.md` | Prerequisites (Claude Code, Git, optional `jq`/Python), platform notes |
| `technical-preferences.md` | Template for the user to fill in their engine, language, and performance targets |
| `coding-standards.md` | Index of all 11 rules with short summaries |
| `coordination-rules.md` | Formal delegation model (vertical, horizontal, escalation, propagation, domain) |
| `context-management.md` | How to use `production/session-state/active.md` to persist state across sessions |
| `hooks-reference.md` | Summary table of all 8 hooks |
| `rules-reference.md` | Summary table of all 11 rules |
| `skills-reference.md` | Index of all 37 skills with one-line descriptions |
| `review-workflow.md` | Step-by-step code review and design review workflow |
| `directory-structure.md` | Annotated project directory layout (imported by `CLAUDE.md`) |
| `CLAUDE-local-template.md` | Template for a local `CLAUDE.local.md` override (gitignored) |
| `settings-local-template.md` | Template for local settings overrides |

#### `docs/templates/` — 29 Document Templates

Ready-made Markdown templates for every production artifact:

- **Design**: `game-concept.md`, `game-pillars.md`, `game-design-document.md`, `technical-design-document.md`, `economy-model.md`, `level-design-document.md`, `faction-design.md`
- **Narrative**: `narrative-character-sheet.md`
- **Production**: `sprint-plan.md`, `milestone-definition.md`, `project-stage-report.md`, `risk-register-entry.md`, `systems-index.md`, `concept-doc-from-prototype.md`
- **Technical**: `architecture-decision-record.md`, `architecture-doc-from-code.md`, `design-doc-from-implementation.md`, `test-plan.md`
- **Release**: `release-checklist-template.md`, `release-notes.md`, `changelog-template.md`, `patch-notes.md`, `post-mortem.md`, `incident-response.md`, `pitch-document.md`
- **Agent Protocols**: `design-agent-protocol.md`, `implementation-agent-protocol.md`, `leadership-agent-protocol.md`
- **Art & Audio**: `art-bible.md`, `sound-bible.md`

---

### 8. `docs/` — Project-Level Documentation

| File / Directory | Contents |
|---|---|
| `COLLABORATIVE-DESIGN-PRINCIPLE.md` | The philosophical and practical foundation of the user-driven collaboration model |
| `WORKFLOW-GUIDE.md` | Comprehensive guide to every workflow pattern the framework supports |
| `examples/` | Annotated session transcripts showing real usage of `/brainstorm`, `/team-combat`, `/scope-check`, and `/reverse-document` |
| `engine-reference/godot/` | Godot 4 version pin, breaking changes, deprecated APIs, best practices, and module-level notes |
| `engine-reference/unity/` | Unity equivalent of the above, including DOTS/Cinemachine/Addressables plugin notes |
| `engine-reference/unreal/` | Unreal Engine 5 equivalent, including GAS/PCG/CommonUI plugin notes |

The engine reference directories are **not** auto-generated. They are manually curated snapshots intended to be updated when the pinned engine version changes (see `UPGRADING.md`).

---

### 9. `production/` — Runtime Session State

Two subdirectories (both gitignored):

- `production/session-state/active.md` — the *single source of truth* for what the current session is working on: active sprint, current epic, and current task. Written by the user or skills like `/sprint-plan`; read by `session-start.sh` and `pre-compact.sh`.
- `production/session-logs/` — append-only audit log written by `session-stop.sh` and `log-agent.sh`.

---

## How the Components Interact

```
User types "/team-combat"
        │
        ▼
Claude Code reads .claude/skills/team-combat/skill.md
        │   (skill prompt instructs which agents to invoke)
        │
        ▼
SessionStart hooks fire first (already ran at session open):
  session-start.sh  ──▶  prints sprint context
  detect-gaps.sh    ──▶  checks for missing docs
        │
        ▼
Claude invokes game-designer agent
  ┌─ reads .claude/agents/game-designer.md (system prompt)
  ├─ reads .claude/rules/design-docs.md (if editing design/gdd/**)
  └─ follows coordination-rules.md (asks questions, shows options, waits)
        │
        ▼
Claude delegates to gameplay-programmer
  ┌─ reads .claude/agents/gameplay-programmer.md
  ├─ reads .claude/rules/gameplay-code.md (if editing src/gameplay/**)
  └─ asks "May I write to src/gameplay/combat.gd?"
        │   (user approves)
        │
        ▼
PreToolUse hook fires on git commit:
  validate-commit.sh  ──▶  checks magic numbers, TODOs, JSON
        │
        ▼
PostToolUse hook fires after file write:
  validate-assets.sh  ──▶  validates asset naming/structure
        │
        ▼
log-agent.sh fires on SubagentStart  ──▶  writes audit trail entry
        │
        ▼
Session ends → session-stop.sh  ──▶  logs accomplishments
```

---

## Customization Guide

This is a template, not a locked framework. Common customizations:

| Goal | What to Change |
|---|---|
| Choose an engine | Fill in `CLAUDE.md` and `technical-preferences.md`; use the matching engine specialist agents |
| Remove unused agents | Delete files from `.claude/agents/` |
| Add a new specialist | Create a new `.md` file in `.claude/agents/` following the existing YAML front-matter format |
| Add a new workflow | Create a subdirectory in `.claude/skills/` with a `skill.md` prompt |
| Tighten/relax code rules | Edit the relevant file in `.claude/rules/` |
| Adjust hook strictness | Edit the relevant script in `.claude/hooks/` |
| Add project-specific context | Edit `CLAUDE.md` or create a `CLAUDE.local.md` (gitignored) |
| Override settings locally | Copy `settings-local-template.md` to `.claude/settings.local.json` |

---

## Key Design Principles

1. **User-driven, not autonomous** — every agent must ask before acting, show drafts, and wait for approval. No autonomous commits.
2. **Separation of concerns** — each agent owns exactly one domain; files outside that domain require explicit delegation.
3. **Graceful degradation** — all hooks work with or without optional tools (`jq`, Python). Missing tools skip the check silently.
4. **Template, not framework** — every file is meant to be edited or deleted. Nothing is locked.
5. **Grounded in theory** — agents apply MDA Framework, Self-Determination Theory, Flow State Design, and Bartle Player Types, not just intuition.
