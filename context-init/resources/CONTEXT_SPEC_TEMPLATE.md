# CONTEXT_SPEC — Canonical Specification for Context Documents

> **IMMUTABLE.** This document defines the canonical structure of the project's context documents.
> Any change to `metaspec.md`, `index.md`, or `timeline.md` MUST follow this spec.
> Changing THIS document requires explicit user approval.
>
> Version: 2.1 | Created: {{CREATION_DATE}}

---

## Core Principles

Based on Anthropic's official CLAUDE.md guidelines, industry best practices for AI-Driven
Development (Google, Meta, Stripe), and token-efficient documentation patterns for LLMs.

### 1. Token Economy

Every line consumes context tokens. Lines that do not change the agent's behavior are cost
without return. Golden rule: **if removing the line causes no error, it should not exist.**

### 2. Single Ownership

Each piece of information lives in EXACTLY ONE document. Redundancy across files creates
inevitable divergence and wastes tokens.

### 3. Derivability

Information the agent can obtain by reading the code (via Glob, Grep, Read) must NOT be
documented. Document only what is NOT inferable: decisions, motivations, business rules,
and state that requires human context.

### 4. Anti-Frankenstein

Context documents are NOT logs. They represent the CURRENT STATE of the project.
Updates REPLACE information — they never accumulate. History belongs to git and to
`timeline.md`.

### 5. Progressive Disclosure (Tier 1/2/3)

The 3 context documents are Tier 1: short selectors that orient the agent.
Deep detail lives in Tier 2 (walkthroughs/plans) and Tier 3 (docs/).

- Tier 1 (always loaded): metaspec/index/timeline — short bullets, one fact each
- Tier 2 (on-demand): context/walkthroughs/, plans/ — implementation detail
- Tier 3 (referenced): docs/ — detailed technical reference

Rule: if a metaspec bullet describes WHY something works, it belongs to Tier 1. If it
describes HOW it was implemented, it belongs in the walkthrough.

---

## Ownership Map

Each type of information has exactly one owner. Violating ownership creates redundancy.

| Information | Owner | Do NOT place in |
|-------------|-------|-----------------|
| Identity, domain, purpose | `metaspec.md` | index, timeline |
| Stack and versions | `metaspec.md` | index, CLAUDE.md |
| Architecture and data flow | `metaspec.md` | docs/ |
| Current state (version, branch, health) | `metaspec.md` | index, timeline |
| Open technical debt | `metaspec.md` | timeline |
| Authentication and authorization | `metaspec.md` | index, timeline |
| Data sources and integrations | `metaspec.md` | index, timeline |
| Business rules (1-line summary + ref) | `metaspec.md` | timeline, index |
| Business rules (canonical detail) | `docs/RULES_*.md` (Tier 3) | metaspec, timeline |
| List of active artifacts | `index.md` | metaspec |
| Navigation between documents | `index.md` | metaspec, timeline |
| Inventory of critical files | `index.md` | metaspec |
| Test summary (status) | `index.md` | metaspec |
| Infrastructure and CI/CD | `index.md` | metaspec |
| Technical documentation (references) | `index.md` | metaspec |
| Evolutionary history by phase | `timeline.md` | metaspec, index |
| Accumulated historical metrics | `timeline.md` | metaspec, index |
| Dev commands (build, test, lint) | `CLAUDE.md` | metaspec, index |
| Code/workflow conventions | `CLAUDE.md` | metaspec |

---

## Format: metaspec.md

**Purpose:** Essential project context for the agent. Answers: "What is this project,
how does it work, and what do I need to know to avoid breaking anything?"

**Token budget:** ~4000 tokens. Mechanical proxy: `wc -w file.md` * 1.3. Soft cap — exceeding it raises a warning in Phase 6 of `/context-update`, it does not block. Lines: derived, not validated.

### Required Sections (in this order)

```
# MetaSpec — {ProjectName}
> Context for AI agents. Version: X.Y | Updated: YYYY-MM-DD

## IDENTITY
  - Name, domain, purpose, target users, language
  - Maximum: 5 bullets

## STACK
  - Code block with technologies and versions
  - One line per layer (e.g. runtime, framework, database, infra)
  - Do NOT list LOC — it changes every commit

## ARCHITECTURE
  - ASCII data-flow diagrams (maximum 2)
  - Layer tables: Layer | Directory | Responsibility
  - Do NOT list individual files (that belongs to index.md)
  - Do NOT list LOC

## CURRENT STATE (vX.Y — DD/MM/YYYY)
  - Current branch and overall health (1 line)
  - "Ready" — concise list of what works (bullets)
  - "Technical debt" — list with the action required
  - Do NOT include history of previous versions (that belongs to timeline)
  - Do NOT include "what was removed in vX" (that belongs to timeline)
  - Do NOT include "what changed in vX.Y" (that belongs to timeline)
```

### Conditional Sections (include only if applicable to the project)

```
## DATA
  - Include if: the project integrates with external sources or stores data
  - External sources (APIs, databases, queues, streams)
  - Local storage (caches, TTLs, file storage)
  - Configs that affect behavior

## AUTH
  - Include if: the project has authentication/authorization
  - Single table: Aspect | Detail
  - Provider, algorithm, roles, access flow

## CRITICAL BUSINESS RULES
  - Include if: the domain has rules the agent MUST know to avoid breaking logic
  - Format: one subsection per domain with ONLY a 1-line summary + reference
    e.g. "- {domain}: <1-line summary of the rule set>. Details: docs/RULES_{domain}.md"
  - Canonical detail (hardcoded lists, tables, formulas, edge cases) lives in
    docs/RULES_*.md (Tier 3). The metaspec holds only the selector + reference.
  - Status badges / small rules (~3 bullets) may stay inline (the overhead of
    creating a dedicated doc > the tokens saved)
  - ONLY non-obvious rules that affect implementation
```

### Sections FORBIDDEN in metaspec

- `REFERENCES` — the index already does this
- `CONVENTIONS` — CLAUDE.md already does this
- Version history ("Removed in v11", "Changed in v12.1")
- LOC of individual files
- Cross-references to other context documents

### Update Rule

When updating `metaspec.md`, the agent MUST:
1. **Replace** the relevant section with the new state (do not append to the existing one)
2. Stay within the token budget (~4000 tokens) and respect density (~200 chars/bullet)
3. Update the version and date in the header
4. If information migrated to another document, REMOVE it from the metaspec

---

## Format: index.md

**Purpose:** Navigation map. Answers: "Where do I find X?"

**Token budget:** ~3000 tokens. Proxy: `wc -w * 1.3`. Soft cap (warning, not a block). Lines: derived.

### Required Sections (in this order)

```
# Context Index — {ProjectName}
> Artifact map. Updated: YYYY-MM-DD

## Quick Navigation
  - 4-5 row table: Artifact | Path | Description
  - Core context documents only

## Active Artifacts
  ### Analyses — context/analysis/
  ### Walkthroughs — context/walkthroughs/
  ### Plans — plans/
  - Table: File | Date | Description | Status
  - ONLY files at the root (not old/)
  - Implemented plans should be moved to old/ — not listed here

## Critical Files
  - Group by project layer (free-form, adaptable categories)
  - Example categories: "Core", "API", "UI", "CLI", "Config", "Workers"
  - Table: File | Responsibility
  - ONLY files the agent needs to know about to avoid mistakes
  - Do NOT list LOC (it changes constantly)
  - Do NOT list EVERY file (the agent knows how to use Glob)
```

### Conditional Sections (include only if applicable to the project)

```
## Tests
  - Include if: the project has test suites
  - Summary table: Layer | Directory | Status
  - Do NOT list exact assertion counts
  - Use "passing" / "N failing" as status

## Infrastructure
  - Include if: the project has CI/CD or deploy configs
  - Table: File | Description
  - Only CI configs, deploy scripts, .env locations

## Technical Documentation
  - Include if: docs exist under docs/
  - Table: File | Description
  - References to the project's detailed technical documentation
```

### Sections FORBIDDEN in index

- Exact counts of LOC, assertions, or files
- Historical notes ("Removed in vX", "Was Y, now is Z")
- `old/` subsections with counts of historical artifacts
- Duplication of metaspec information (stack, config details)

### Update Rule

When updating `index.md`, the agent MUST:
1. **Add** newly created artifacts
2. **Remove** artifacts moved to `old/` or deleted
3. NEVER increment counters ("now there are 42 tests") — use qualitative status
4. Stay within the token budget (~3000 tokens)
5. Update the date in the header

---

## Format: timeline.md

**Purpose:** The project's evolutionary history. Answers: "How did we get here, and why?"

**Token budget:** ~5000 tokens. Proxy: `wc -w * 1.3`. Soft cap (warning, not a block). Lines: derived. Older phases should be compressed when the budget is exceeded.

### Required Sections (in this order)

```
# Timeline — {ProjectName}
> Evolutionary history. {N} phases | {period}.

## Phase N: {Name} ({period})
  - Commits: start_hash -> end_hash
  - 3-5 bullets describing WHAT changed and WHY
  - Do NOT include "Milestone:" — it is redundant with the bullets
  - Do NOT list individual walkthroughs — consult old/ if needed

## Metrics Snapshot ({date})
  - Summary table with APPROXIMATE metrics (use "~")
  - Do NOT promise precision that decays (exact counts become lies)
```

### Compressing Older Phases

When `timeline.md` exceeds its token budget:
1. Compress older phases (>3 months) into a 2-line format:
   ```
   ## Phase 0: Foundation (Dec/2025) — Initial setup + working API
   ## Phase 1: Refactor (Jan/2026) — Layered architecture
   ```
2. Keep recent phases (<3 months) in the full format
3. Details of compressed phases remain accessible via `git log`

### Sections FORBIDDEN in timeline

- Lists of walkthroughs per phase (they exist in `old/`)
- "Milestone:" as a subsection (redundant)
- LOC before/after refactors
- Exact commit counts per phase (use "~N")

### Update Rule

When updating `timeline.md`, the agent MUST:
1. **Add** a new phase OR update the phase in progress
2. NEVER edit completed phases (>1 month) except to compress them
3. Update the metrics snapshot with approximate values
4. Respect density (~200 chars/bullet); verbose detail goes to walkthroughs (Tier 2)
5. If the token budget (~5000 tokens) is exceeded, compress the oldest phases

---

## Anti-Frankenstein Rules

These rules prevent cruft from accumulating in the context documents.

### 1. Update = Replace

When the project state changes, the corresponding section is REWRITTEN with the new state.
Never add "Updated DD/MM: X is now Y" — that is a log, not state.

**Wrong:**
```
## CURRENT STATE (v2.0)
- Working REST API
- JWT auth

### What changed in v2.1
- Added rate limiting
- Fixed refresh token bug
```

**Right:**
```
## CURRENT STATE (v2.1 — 30/03/2026)
- Working REST API with rate limiting
- JWT auth with working refresh token
```

### 2. One Fact, One Place

If the test count appears in `metaspec.md` AND in `index.md`, which is correct when they
diverge? Answer: neither — because the count should not be in either. The agent knows how
to count on its own.

### 3. Exact Numbers Decay

Exact counts (661 assertions, 484 LOC, 418 commits) go stale after the next commit. Use:
- "~160 tests" instead of "162"
- "~500 assertions" instead of "499"
- Or omit numbers and use status: "tests passing", "2 tests failing"

### 4. History Belongs to Git

"What was removed in v2" is not current state — it is history. If it must be recorded, it
goes in `timeline.md` under the corresponding phase. If it is already in the timeline, do
NOT duplicate it in the metaspec.

### 5. Prune Test

Before saving any update, ask of each added line:
> "If I remove this line, will the agent make a mistake?"

If the answer is "no", the line should not exist.

### 6. One Fact, One Bullet (Density)

Each bullet contains ONE fact. Soft cap ~200 characters per line.

Paragraph-bullets (stacking changes/causes/effects into a single bullet to game a size
limit) violate the spirit of SNR: the doc looks compact but the real token cost is the
same as a poorly organized doc 3x its size.

When a bullet exceeds ~200 chars, choose ONE of 3 actions:
1. Break it into sub-bullets (one fact each)
2. Move detail to a referenced walkthrough/plan (explicit path) — see Principle 5
3. Apply the prune test and cut information derivable from the code

Exceptions: lines in markdown tables (`|`) and code blocks (` ``` `).

**Wrong** (1 bullet, ~600 chars, 5 facts mixed together):
```
- New feature X (DD/MM): refactors component Y, adds prop Z, fixes a bug
  in case W, migrates from helper A to B, and updates tests. Validated in
  prod by user Q. Plan in plans/old/2026-X. Walkthrough in
  context/walkthroughs/2026-X. Cases covered: 5 cross-side scenarios
  with edge cases, including regression in the component's pre-existing
  tests. Build OK, 600 vitest passing.
```

**Right** (3 short sub-bullets + reference):
```
- Feature X (DD/MM): refactors component Y; new prop Z; fixes bug in case W
  - Migrates helper A -> B (5 cross-side scenarios covered)
  - Details: context/walkthroughs/2026-X.md
```

---

## Validation Checklist

Use this checklist when updating any context document:

- [ ] Is the document within its token budget (metaspec ~4k, index ~3k, timeline ~5k)?
- [ ] Does no bullet exceed ~200 characters (excluding markdown tables and code blocks)?
- [ ] Has implementation detail been moved to walkthroughs/plans (Tier 2)?
- [ ] Is no information duplicated in another context document?
- [ ] Are there no exact numbers that will decay on the next commit?
- [ ] Is there no history/changelog section in the metaspec?
- [ ] Is there no "what changed in vX.Y" section (except in the timeline)?
- [ ] Is ownership respected per the table above?
- [ ] Has information derivable from the code been omitted?
- [ ] Is the update date current in the header?

---

## References

This spec was built from:

- **Anthropic CLAUDE.md Best Practices**: target <200 lines, prune test, conditional loading
- **Anthropic Prompting Guide**: XML tags, declarative > descriptive, primacy effect
- **Google AI Documentation Standards**: structured context, single source of truth
- **Stripe Engineering Practices**: minimal viable documentation, ownership model
- **Meta AI-Assisted Development**: token-efficient context, derivability principle
- **Community Consensus** (Cursor Rules, Windsurf Rules, Cline Rules): short, specific, verifiable
- **Documented anti-patterns**: Frankenstein docs, kitchen sink sessions, governance theater
- **Anthropic principle**: "If your CLAUDE.md is too long, Claude ignores half of it"
- **Token-efficient documentation**: maximize signal-to-noise ratio in limited context
