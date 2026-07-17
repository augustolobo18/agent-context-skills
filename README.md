# Agent Context Skills

A collection of **Agent Skills** for managing project context and planning implementation work. These skills give an AI coding agent a durable, structured memory of a codebase and a disciplined workflow for planning changes before writing any code.

They are **agent-agnostic** — designed to work with any CLI coding agent (Claude Code, Codex, Antigravity, and others). Each skill is plain Markdown with YAML frontmatter, so it can be loaded by any harness that supports skills or dropped in as instructions.

## What's inside

| Skill | Version | What it does |
|-------|---------|--------------|
| [`context-init`](context-init/SKILL.md) | 1.0 | Initializes the context-documentation structure for any software project. |
| [`context-loader`](context-loader/SKILL.md) | 2.0 | Loads all of a project's context documentation into the context window at the start of a session. |
| [`context-update`](context-update/SKILL.md) | 2.0 | Validates and updates the context documentation (metaspec, index, timeline) after changes. |
| [`implementation-plan`](implementation-plan/SKILL.md) | 4.0 | Generates detailed implementation plans — with impact analysis, phases, and diagrams — *before* any code is written. |
| [`walkthrough`](walkthrough/SKILL.md) | 3.1 | Generates technical documentation of a completed implementation — with diagrams, metrics, and automatic project detection — before a commit or review. |

Plus a supporting subagent:

- [`agents/plan-executor.md`](agents/plan-executor.md) — executes an already-approved implementation plan in an isolated context.

## How they fit together

```
context-init      →  set up context docs for a new project
      │
      ▼
context-loader    →  load project state at the start of each session
      │
      ▼
implementation-plan  →  plan a change (discovery → plan → approval)
      │                         │
      │                         ▼
      │                 plan-executor  →  execute the approved plan
      ▼
walkthrough       →  document the completed implementation before commit/review
      │
      ▼
context-update    →  record what changed back into the context docs
```

## Repository layout

```
.
├── agents/
│   └── plan-executor.md
├── context-init/
│   ├── SKILL.md
│   ├── examples/
│   ├── pressure-tests/
│   └── resources/
├── context-loader/
│   ├── SKILL.md
│   ├── examples/
│   ├── pressure-tests/
│   └── resources/
├── context-update/
│   ├── SKILL.md
│   ├── examples/
│   └── pressure-tests/
├── implementation-plan/
│   ├── SKILL.md
│   ├── discovery-phase.md
│   ├── approval-workflow.md
│   ├── plan-structure.md
│   ├── visual-templates.md
│   ├── visual-templates-detailed.md
│   ├── examples/
│   ├── pressure-tests/
│   └── resources/
└── walkthrough/
    ├── SKILL.md
    ├── data-collection.md
    ├── mermaid-guidelines.md
    ├── pattern-learning.md
    ├── report-template.md
    ├── visual-templates.md
    ├── examples/
    ├── pressure-tests/
    └── resources/
```

Each skill directory contains a `SKILL.md` with YAML frontmatter (name, version, description, allowed tools, token budget) followed by the skill instructions, alongside examples, pressure tests, and resource templates.

## Usage

Each skill is a self-contained Markdown file (`SKILL.md`) plus its supporting resources. To use them, make the skill directories available to your coding agent however it discovers skills or instruction files. A few common patterns:

- **Claude Code** — place the skill directories under `~/.claude/skills/` (user scope) or `.claude/skills/` (project scope), and `agents/plan-executor.md` under `~/.claude/agents/`.
- **Other CLI agents (Codex, Antigravity, etc.)** — point the agent at the relevant `SKILL.md`, or include its contents in the agent's instruction/context files.

Because the skills are just structured Markdown, they degrade gracefully: even an agent with no formal skill system can read a `SKILL.md` and follow it as a procedure.

## License

Released under the [MIT License](LICENSE).
