---
name: context-init
version: 1.0
status: production
description: |
  Initializes the context documentation structure for any software project.
allowed-tools: Read, Glob, Grep, Bash, Write
requires:
  - Read
  - Glob
  - Grep
  - Bash
  - Write
token_budget: 5000
metrics:
  execution_time: ~45s
  avg_docs_generated: 3-4
  success_rate: "99%"
components:
  - resources/CONTEXT_SPEC_TEMPLATE.md
  - examples/init-full-example.md
  - examples/init-partial-example.md
  - pressure-tests/test-scenarios.md
---

You are a context initialization agent. You create the context documentation structure for
ANY project, detecting what already exists and creating only what is missing. The skill is
generic — it works with Python, TypeScript, Go, Rust, Java, or any stack.

## <constraints>
- **Paths**: ALWAYS use the current working directory (cwd) as the project root.
- **Environment**: Bash on Windows (git, ls, mkdir work). Avoid PowerShell.
- **Never overwrite**: If a doc already exists, do NOT touch it. Create only what is missing.
- **Canonical template**: CONTEXT_SPEC.md comes EXCLUSIVELY from the template at `~/.claude/skills/context-init/resources/CONTEXT_SPEC_TEMPLATE.md`. Never generate it from scratch.
- **Token budgets**: metaspec ~4000 tokens, index ~3000, timeline ~5000 (proxy `wc -w * 1.3`).
- **Density**: concise bullets (~200 chars max, 1 fact each). Tables/code blocks are exempt.
- **Dynamic detection**: Never hardcode project names. Everything is inferred from the codebase.
- **Resilience**: If a file/command fails, log it and continue with the rest.
</constraints>

## <phases>

### Phase 0: Inventory — What already exists?

Run in parallel (Glob for each):
1. `context/CONTEXT_SPEC.md`
2. `context/metaspec.md`
3. `context/index.md`
4. `context/timeline.md`
5. Directories: `context/analysis/`, `context/walkthroughs/`, `plans/`

Build two lists:
- **Existing**: docs/dirs already in the project (they will be kept intact)
- **To create**: missing docs/dirs (they will be generated in the following phases)

**If EVERYTHING exists**: skip Phases 1-6 (nothing to create). Go straight to Phase 7 and Phase 8.

### Phase 1: Deep Project Scan

Run ONLY if there are docs to create. If everything already exists, skip to Phase 7. Collect in parallel:

**1a. Project identity**
Detect the name in this priority order (use the first one found).
If multiple manifests of the same type exist (e.g. a monorepo), prioritize the one at the project ROOT.
For scoped npm packages (`@org/name`), use the full name as the identity.
- `package.json` → `name` field
- `pyproject.toml` → `name` field under `[project]` or `[tool.poetry]`
- `setup.py` → `name` field in `setup()`
- `Cargo.toml` → `name` field under `[package]`
- `go.mod` → module path (last segment)
- `pom.xml` → `<artifactId>`
- `.git/config` → remote origin URL (last segment without .git)
- **Fallback**: current directory name

**1b. Stack and dependencies**
- Glob `**/*.py`, `**/*.ts`, `**/*.js`, `**/*.go`, `**/*.rs`, `**/*.java`, `**/*.tsx`, `**/*.jsx` (count occurrences per extension to identify the predominant languages)
- Read the dependency manifest: `package.json`, `requirements.txt`, `pyproject.toml`, `Pipfile`, `Cargo.toml`, `go.mod`, `pom.xml` (whichever exists)
- Detect the runtime version in the relevant ecosystem files:
  - Python: `.python-version`, `runtime.txt`
  - Node: `.node-version`, `.tool-versions`
  - Rust: `Cargo.toml` → `[package].edition`
  - Java: `pom.xml` → `<java.version>` or `<maven.compiler.source>`
- Detect frameworks from imports and dependencies (React, Next.js, FastAPI, Django, Express, Gin, Actix, Spring, etc.)

**1c. Architecture**
- Read entry points: `main.*`, `app.*`, `index.*`, `src/main.*`, `src/app.*`, `src/index.*`, `src/main/java/**/Main.java`, `src/main/java/**/Application.java`, `cmd/main.go`
- Map the structure: `ls` of top-level directories and `src/` if it exists
- Read configs: `docker-compose.yml`, `Dockerfile`, `.env.example`, `tsconfig.json`, `webpack.config.*`, `vite.config.*`

**1d. Git history**
- `git log --oneline -30` to extract phases
- `git log --format="%H %ad %s" --date=short -30` for dates and messages
- `git log --format="%H" --reverse -1` for the first commit hash
- `git rev-parse --short HEAD` for the current hash
- `git branch --show-current` for the current branch
- If `.git` does not exist, log "no git history" and continue

**1e. Tests and CI/CD**
- Glob: `test/`, `tests/`, `__tests__/`, `spec/`, `**/*.test.*`, `**/*.spec.*`
- Glob: `.github/workflows/*.yml`, `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/config.yml`

**1f. Existing docs**
- Glob: `README.md`, `docs/*.md`, `docs/**/*.md`
- Read `README.md` if it exists (first 100 lines, use the `limit` parameter). Extract purpose and description.

**1g. Pattern detection for conditional sections**
- AUTH: Grep for `passport|jwt|oauth|bcrypt|auth0|nextauth|clerk|supabase/auth` in the collected dependencies
- DATA: Grep for `redis|rabbitmq|kafka|sqs|s3|firebase|prisma|typeorm|sqlalchemy|mongoose|sequelize` in the collected dependencies
- BUSINESS RULES: Presence of directories such as `domain/`, `rules/`, `policies/`, `validators/`, `business/`

### Phase 2: Create directories (if they do not exist)

```bash
mkdir -p context/analysis context/walkthroughs plans
```

Record which directories were created vs already existed.

### Phase 3: Generate CONTEXT_SPEC.md (if it does not exist)

1. **Read**: `~/.claude/skills/context-init/resources/CONTEXT_SPEC_TEMPLATE.md`
2. Replace `{{CREATION_DATE}}` with the current date (YYYY-MM-DD format)
3. **Write**: `context/CONTEXT_SPEC.md`

Do NOT alter any other part of the template. It is the canonical spec.

### Phase 4: Generate metaspec.md (if it does not exist)

Use the data collected in Phase 1. The format MUST follow the `## Format: metaspec.md` section of the CONTEXT_SPEC EXACTLY (see `resources/CONTEXT_SPEC_TEMPLATE.md`).

Fill it with data inferred from the scan:
- **IDENTITY**: name, domain, purpose (from the README/manifest), target users, code language
- **STACK**: detected technologies and versions, one per layer
- **ARCHITECTURE**: ASCII flow diagram (based on entry points and structure) + Layer | Directory | Responsibility table
- **CURRENT STATE**: branch, what works (from recent git log), technical debt (inferred or empty)

**Conditional sections** (include ONLY if detected in Phase 1g):
- `## DATA` — if Phase 1g detected integrations with APIs/databases/queues
- `## AUTH` — if Phase 1g detected auth dependencies
- `## CRITICAL BUSINESS RULES` — if Phase 1g detected domain/rules directories
  - Content in the metaspec: ONLY a 1-line summary per domain + a reference to `docs/RULES_*.md`
  - If the project has complex domain rules (hardcoded lists, tables, formulas),
    create/maintain a Tier 3 doc named after the domain (`docs/RULES_{domain}.md`)
  - Small rules (~3 bullets) may stay inline in the metaspec without a reference

**Token budget**: ~4000 tokens (proxy `wc -w * 1.3`). Concise bullets (1 fact each, ~200 chars). Implementation detail goes to walkthroughs/plans (Tier 2). Omit empty sections.

### Phase 5: Generate index.md (if it does not exist)

Use the data collected in Phase 1. The format MUST follow the `## Format: index.md` section of the CONTEXT_SPEC EXACTLY (see `resources/CONTEXT_SPEC_TEMPLATE.md`).

Fill it with data inferred from the scan:
- **Quick Navigation**: table of the 4 context docs (CONTEXT_SPEC, MetaSpec, Index, Timeline)
- **Active Artifacts**: empty sections ("No artifacts.") for Analyses, Walkthroughs, Plans
- **Critical Files**: group by detected layer (free-form categories), File | Responsibility table. ONLY critical files, do NOT list them all.

**Conditional sections** (include ONLY if detected):
- `## Tests` — if Phase 1e found test suites
- `## Infrastructure` — if Phase 1e found CI/CD configs
- `## Technical Documentation` — if Phase 1f found docs under docs/

**Token budget**: ~3000 tokens. Concise bullets (~200 chars).

### Phase 6: Generate timeline.md (if it does not exist)

Use the `git log` from Phase 1. The format MUST follow the `## Format: timeline.md` section of the CONTEXT_SPEC EXACTLY (see `resources/CONTEXT_SPEC_TEMPLATE.md`).

Fill it with data from git:
- Analyze commit messages by period (weeks/months) and group them into thematic phases
- 3-5 bullets per phase describing WHAT changed and WHY
- Metrics Snapshot with APPROXIMATE values (~)

**Without git**: If the project has no git history, generate a minimal timeline with 1 "Initialization" phase and basic metrics (detected language, "git history not available").

**Token budget**: ~5000 tokens. Concise bullets (~200 chars); detail goes to walkthroughs (Tier 2).

### Phase 7: Output — Terminal Report

Display EXACTLY in this format (text in the conversation, NOT as a file):

```
CONTEXT INITIALIZED — {ProjectName}

Created:
  + context/CONTEXT_SPEC.md (canonical template v2.1)
  + context/metaspec.md (~N tokens)
  + context/index.md (~N tokens)
  + context/timeline.md (~N tokens)
  + context/analysis/ (directory)
  + context/walkthroughs/ (directory)
  + plans/ (directory)

Already existed:
  — context/metaspec.md (kept intact)

Stack detected: {1-line summary}
Phases identified: {N} ({period})

Failures: (if any)
  ! context/metaspec.md — permission error (details)

Ready for /context-loader.
```

List under "Created" ONLY what was created in this run.
List under "Already existed" ONLY what was found in Phase 0.
List under "Failures" ONLY docs/dirs that failed to be created (with the reason). Omit the section if there are no failures.
If nothing was created (everything existed), display:
```
CONTEXT ALREADY EXISTS — {ProjectName}
All docs and directories are already present. No files created.
Invoking /context-update to validate compliance...
```

### Phase 8: Invoke /context-update (ALWAYS)

This phase runs ALWAYS, regardless of how many docs were created (0, 1, 2, 3, or 4).

Invoke the `/context-update` skill to:
1. **Validate compliance** of the existing docs (those not created in this run)
2. **Update** docs that are out of date relative to the current codebase
3. **Fix** malformed docs that do not follow the CONTEXT_SPEC (with user approval)

> Docs just created by context-init already follow the CONTEXT_SPEC, but context-update
> will review them all the same for consistency.

</phases>

## <quality-checks>
- Was Phase 0 executed BEFORE creating anything?
- Was no existing doc overwritten?
- Did CONTEXT_SPEC.md come from the canonical template (not generated from scratch)?
- Is the metaspec ~4k tokens? index ~3k tokens? timeline ~5k tokens (proxy `wc -w * 1.3`)?
- Do bullets respect ~200 chars (1 fact = 1 bullet)?
- Was Progressive Disclosure applied (detail lives in walkthroughs/plans, Tier 2)?
- Were project names detected dynamically (nothing hardcoded)?
- Was CONTEXT_SPEC ownership respected in each generated doc?
- Were approximate numbers (~) used instead of exact counts?
- Were the analysis/, walkthroughs/, plans/ directories created?
- Was Phase 8 (/context-update) invoked at the end?

Edge cases are documented in `pressure-tests/test-scenarios.md`.
Output examples in `examples/init-full-example.md` and `examples/init-partial-example.md`.
</quality-checks>
