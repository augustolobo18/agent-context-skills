# Example: Generic Briefing

**Scenario**: TypeScript/React project "task-tracker" with a complete context structure.

## Phase 0: Discovery

```
context/metaspec.md  — FOUND
context/index.md     — FOUND
context/timeline.md  — FOUND

Result: 3/3 docs exist. Proceeding to load.
```

## Phases 1-4: Loading

```
Phase 1 — Core docs:
  CLAUDE.md               — read (42 lines)
  context/CONTEXT_SPEC.md — read (300 lines)
  context/metaspec.md     — read (87 lines)
  context/index.md        — read (62 lines)
  context/timeline.md     — read (58 lines)
  Project name: task-tracker (from the metaspec header)

Phase 2 — Active artifacts:
  context/analysis/     — 1 file (report-2026-03.md)
  context/walkthroughs/ — 2 files (settings-page.md, auth-flow.md)
  plans/                — 1 file (admin-panel.md)

Phase 3 — Technical docs (from index.md):
  docs/API_REFERENCE.md — listed (not read)
  docs/DEPLOY_GUIDE.md  — listed (not read)

Phase 4 — Memory:
  MEMORY.md             — read (8 lines, 3 references)
  project_stack.md      — read
  feedback_testing.md   — read
  user_role.md          — read
```

> Note: Per Phase 2's policy, only the most recent walkthrough (settings-page.md) is read
> in full. The analysis report is listed but not read. Technical docs (Tier 3) are listed
> only — they are read on demand if the session touches the subject.

## Phase 5: Expected Briefing

```
CONTEXT [task-tracker] LOADED
Files read: 12 | ~38k tokens estimated

State: v2.3, branch main. Full CRUD + working admin panel.
Last implementation: settings-page.md — settings page in the admin panel
Technical debt: ~3 components without an error boundary, rate limiting pending

Active plans:
- admin-panel.md — in progress (70%)

Technical docs available (on demand):
- docs/API_REFERENCE.md — REST API reference
- docs/DEPLOY_GUIDE.md — deploy guide

Analysis reports available (on demand):
- report-2026-03.md — 2026-03

Memory: user is a senior TS/React dev, prefers bundled PRs for refactors

Ready to work.
```

## Variation: Project with no artifacts

If there are no walkthroughs, plans, or analysis reports:

```
CONTEXT [data-pipeline] LOADED
Files read: 5 | ~12k tokens estimated

State: v1.0, branch main. Working pipeline with 3 workers.
Last implementation: no walkthrough found
Technical debt: none identified

Active plans:
  none

Technical docs available (on demand): none

Analysis reports available (on demand): none

Memory: no items recorded

Ready to work.
```

## Variation: Missing docs (invokes context-init)

```
context/metaspec.md  — FOUND
context/index.md     — NOT FOUND
context/timeline.md  — NOT FOUND

Result: 1/3 docs exist. Invoking /context-init...
[context-init creates index.md and timeline.md, invokes context-update]
[return] Proceeding to load.
```
