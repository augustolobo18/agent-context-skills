# Example: Full Initialization (Full Init)

**Scenario**: A TypeScript/React project with no pre-existing context structure.

## Simulated Project

- Name: `task-tracker`
- Stack: TypeScript, React, Node.js, PostgreSQL
- Structure: monorepo with `src/`, `api/`, `web/`
- Git: 45 commits, 3 months of history

## Phase 0: Inventory

```
Checking existing structure...
  context/CONTEXT_SPEC.md  — NOT found
  context/metaspec.md      — NOT found
  context/index.md         — NOT found
  context/timeline.md      — NOT found
  context/analysis/        — NOT found
  context/walkthroughs/    — NOT found
  plans/                   — NOT found

Result: 0/4 docs exist. Running full init.
```

## Phase 1: Scan

```
Identity: task-tracker (from package.json)
Stack: TypeScript 5.3, React 18.2, Express 4.18, PostgreSQL 15
Architecture: SPA + REST API, 3 layers (web, api, shared)
Git: 45 commits, branch main, 3 phases identified
Tests: jest (web/__tests__/, api/tests/)
CI: .github/workflows/ci.yml
Patterns (Phase 1g):
  AUTH: jwt detected in dependencies → AUTH section included
  DATA: PostgreSQL detected → DATA section included
  BUSINESS RULES: no domain/rules/policies directory → omitted
```

## Phases 2-6: Creation

All 4 docs and 3 directories created.

## Phase 7: Expected Output

```
CONTEXT INITIALIZED — task-tracker

Created:
  + context/CONTEXT_SPEC.md (canonical template v2.1)
  + context/metaspec.md (~2.3k tokens)
  + context/index.md (~1.6k tokens)
  + context/timeline.md (~1.5k tokens)
  + context/analysis/ (directory)
  + context/walkthroughs/ (directory)
  + plans/ (directory)

Already existed:
  (none)

Stack detected: TypeScript 5.3 + React 18.2 + Express 4.18 + PostgreSQL 15
Phases identified: 3 (Jan/2026 - Mar/2026)

Ready for /context-loader.
```

## Phase 8: Invoke /context-update

After the Phase 7 output, context-init automatically invokes `/context-update`:

```
Invoking /context-update to validate compliance and update docs...
```

context-update validates that the 4 newly created docs follow the CONTEXT_SPEC and are up
to date with the real state of the codebase. Since they were created by context-init
following the spec, the expected result is:

```
CONTEXT VALIDATED — task-tracker
All docs comply with the CONTEXT_SPEC. No updates needed.
Validation: OK (metaspec ~2.3k, index ~1.6k, timeline ~1.5k tokens)
```

## Generated Docs (summary)

### metaspec.md (~2.3k tokens)
- IDENTITY: task-tracker, task management, B2B SaaS
- STACK: TypeScript, React, Express, PostgreSQL with versions
- ARCHITECTURE: SPA → API → DB diagram, 3-layer table
- CURRENT STATE: v0.3, branch main, working CRUD, auth pending
- DATA: local PostgreSQL, no external integrations yet (detected via Phase 1g)
- AUTH: JWT (detected via Phase 1g)

### index.md (~1.6k tokens)
- Quick Navigation: 4 context docs
- Active Artifacts: all empty (init)
- Critical Files: grouped into Web (App.tsx, routes.tsx), API (server.ts, routes/), Config (package.json, tsconfig.json)
- Tests: jest, 2 suites, passing
- Infrastructure: ci.yml

### timeline.md (~1.5k tokens)
- Phase 1: Initial Setup (Jan/2026) — scaffolding, dependencies, base config
- Phase 2: CRUD (Feb/2026) — models, routes, listing components
- Phase 3: UI Polish (Mar/2026) — form validation, error handling, responsiveness
- Metrics Snapshot: TS, ~15 deps, 3 phases
