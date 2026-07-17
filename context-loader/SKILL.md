---
name: context-loader
version: "2.0"
status: production
description: |
  Loads ALL of a project's context documentation into the context window.
  Use at the start of a session to load the project state before starting work.
allowed-tools: Read, Glob
requires:
  - Read
  - Glob
token_budget: 2000
metrics:
  execution_time: ~30s
  avg_files_loaded: 10-20
  success_rate: "99%"
components:
  - examples/briefing-example.md
  - pressure-tests/test-scenarios.md
  - resources/file-priorities.yaml
---

You are a context loading agent. Your job is to read ALL of the project's context
documentation into the context window and display a short briefing in the terminal at the
end. The skill is generic — it works with any software project. Do NOT generate any file —
the summary is displayed only as text output in the conversation.

## <constraints>
- **Paths**: ALWAYS use the current working directory (cwd) as the project root.
- **Read-only**: Do NOT create, modify, or delete any file. Do NOT use Write, Edit, or Bash with redirection.
- **Resilience**: If a file does not exist, log it and continue with the rest.
- **Maximize parallelism**: Use multiple Read calls in parallel whenever possible.
- **Read WHOLE files**: Do not summarize, do not skip. The goal is to load everything into the context window.
- **Do NOT read old/ folders**: Files in `old/` are historical. Root glob patterns only.
- **Dynamic detection**: Never hardcode project names or technical doc paths.
</constraints>

## <phases>

### Phase 0: Discovery — Does the context structure exist?

Run in parallel (Glob for each):
1. `context/metaspec.md`
2. `context/index.md`
3. `context/timeline.md`

Evaluate the result:

- **3/3 exist** → proceed to Phase 1
- **1-2/3 exist** → invoke `/context-init` (it will create the missing ones + validate the existing ones via context-update) → once done, proceed to Phase 1
- **0/3 exist** → invoke `/context-init` (full init) → once done, proceed to Phase 1

> context-init creates the missing docs and ALWAYS invokes context-update at the end to validate compliance. On return, all docs will be present and compliant.

### Phase 1: Core Docs (parallel)

Read in parallel. If one does not exist, log it and continue.

1. `CLAUDE.md` — agent guidelines (if it exists)
2. `context/CONTEXT_SPEC.md` — canonical format (if it exists)
3. `context/metaspec.md` — identity, stack, architecture, state
4. `context/index.md` — artifact index and critical files
5. `context/timeline.md` — evolutionary history

**Detect the project name**: Extract it from the metaspec.md header (`# MetaSpec — {ProjectName}`).

### Phase 2: Active Artifacts (Tier 2 — selective)

Per `CONTEXT_SPEC.md` (Progressive Disclosure), Tier 2 is on-demand. The default load should
bring in only what defines the **current state** and the **next actions**, not the history.

Use Glob to discover (root only, do NOT include `old/`):

```
context/analysis/*.md
context/walkthroughs/*.md
plans/*.md
```

Reading policy:
- **Plans** → **Read ALL** found at the root. (Implemented ones have already been moved to `plans/old/` by project hygiene.) They define the next actions.
- **Walkthroughs** → **Read ONLY the most recent** (highest date in the filename, or mtime). It reflects the state after the last implementation. The earlier ones are already summarized in `timeline.md`.
- **Analysis reports** → **Do NOT read by default**. Only LIST them in the briefing (name + date). They are point-in-time diagnostics; the agent reads them on demand if the session touches the subject.

If no artifact is found in a category, log it and continue.

### Phase 3: Technical Documentation (Tier 3 — listing only)

Per `CONTEXT_SPEC.md`, Tier 3 (`docs/`) is **referenced**, not eager-loaded.

1. Identify the `## Technical Documentation` section in the index.md content (already loaded in Phase 1)
2. Extract the paths + descriptions from the table
3. Do **NOT** read the files. Only record the list (path + short description) to include in the briefing as "docs available on demand"

The agent reads these docs with Read when the session requires it (e.g. when the work touches a documented business rule, it reads that domain's Tier 3 rules doc listed in index.md at that moment).

If the section does not exist in index.md, skip this phase.

### Phase 4: Project Memory

1. Identify the active project's memory directory in Claude Code
2. **Read**: `MEMORY.md` from the memory directory
3. Identify all .md files referenced in MEMORY.md (markdown links)
4. **Read**: each referenced file from the same memory directory

If MEMORY.md does not exist or is empty, log it and continue.

### Phase 5: Terminal Briefing

After reading all the files, display directly in the terminal (as text in the conversation,
NOT as a file) a SHORT briefing:

```
CONTEXT [{PROJECT_NAME}] LOADED
Files read: XX | ~XXXk tokens estimated

State: [extracted from metaspec.md, CURRENT STATE section]
Last implementation: [most recent walkthrough, 1 line]
Technical debt: [from metaspec.md, top 2-3]

Active plans:
- [Plan name] — [status, 1 line]

Technical docs available (on demand):
- [file] — [description from index.md]

Analysis reports available (on demand):
- [file] — [date]

Memory: [relevant items from MEMORY.md, 1-2 lines]

Ready to work.
```

If some information is not available (e.g. no walkthrough), use "none" or "N/A".

### Token Estimation

To estimate tokens, use the formula: `total_characters / 4`. This gives a reasonable approximation.

### Priority Order

If for any reason there is a context limit, prioritize:
1. CLAUDE.md + CONTEXT_SPEC.md + metaspec.md + index.md (critical)
2. Active plans (next actions)
3. Recent walkthroughs (current state of the code)
4. Analysis reports (diagnostics)
5. Referenced technical docs (reference)
6. Memory (long-term context)

</phases>

## FORBIDDEN

This skill is READ-ONLY. The following actions are strictly forbidden:
- Creating, modifying, or deleting any file (do NOT use Write, Edit, or Bash with redirection)
- Using the `start` command to open files
- Generating plans, walkthroughs, analysis reports, or any artifact
- Saving the briefing as a file (it must be ONLY text in the conversation)

Any violation of these constraints invalidates the skill's execution.

## <quality-checks>
- Did Phase 0 check the 3 core docs BEFORE loading?
- Was /context-init invoked when docs were missing?
- Was the project name detected dynamically from metaspec.md?
- Were the technical docs read from index.md (not hardcoded)?
- Was no file in old/ read?
- Was the briefing displayed as text in the conversation (not as a file)?
- Was no file created or modified?

Edge cases are documented in `pressure-tests/test-scenarios.md`.
Briefing example in `examples/briefing-example.md`.
</quality-checks>
