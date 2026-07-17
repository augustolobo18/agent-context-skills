# Example: Partial Initialization (Partial Init)

**Scenario**: A Python/FastAPI project that already has metaspec.md and index.md created manually, but is missing CONTEXT_SPEC.md, timeline.md, and the directories.

## Simulated Project

- Name: `data-pipeline`
- Stack: Python 3.12, FastAPI, Celery, Redis, PostgreSQL
- Structure: `src/`, `workers/`, `api/`
- Git: 120 commits, 6 months of history

## Phase 0: Inventory

```
Checking existing structure...
  context/CONTEXT_SPEC.md  — NOT found
  context/metaspec.md      — FOUND (kept intact)
  context/index.md         — FOUND (kept intact)
  context/timeline.md      — NOT found
  context/analysis/        — NOT found
  context/walkthroughs/    — FOUND
  plans/                   — NOT found

Result: 2/4 docs exist. Creating only the missing ones.
```

## Phase 1: Scan

Runs normally (required to generate timeline.md and CONTEXT_SPEC.md).

```
Identity: data-pipeline (from pyproject.toml)
Stack: Python 3.12, FastAPI 0.109, Celery 5.3, Redis 7, PostgreSQL 15
Git: 120 commits, branch main, 5 phases identified
Patterns (Phase 1g):
  AUTH: none detected → omitted
  DATA: Redis + PostgreSQL detected → DATA section (if the metaspec were being created)
  BUSINESS RULES: no domain/rules/policies directory → omitted
```

> Note: Since metaspec.md already existed, the Phase 1g data was not used in this run. The scan runs anyway because timeline.md needs the git data.

## Phases 2-6: Selective Creation

- CONTEXT_SPEC.md: **CREATED** (from the template)
- metaspec.md: **SKIPPED** (already exists)
- index.md: **SKIPPED** (already exists)
- timeline.md: **CREATED** (from the scan)
- context/analysis/: **CREATED**
- context/walkthroughs/: **SKIPPED** (already exists)
- plans/: **CREATED**

## Phase 7: Expected Output

```
CONTEXT INITIALIZED — data-pipeline

Created:
  + context/CONTEXT_SPEC.md (canonical template v2.1)
  + context/timeline.md (~2.4k tokens)
  + context/analysis/ (directory)
  + plans/ (directory)

Already existed:
  — context/metaspec.md (kept intact)
  — context/index.md (kept intact)
  — context/walkthroughs/ (kept intact)

Stack detected: Python 3.12 + FastAPI 0.109 + Celery 5.3 + Redis 7 + PostgreSQL 15
Phases identified: 5 (Oct/2025 - Mar/2026)

Ready for /context-loader.
```

## Phase 8: Invoke /context-update

After the Phase 7 output, context-init automatically invokes `/context-update`:

```
Invoking /context-update to validate compliance and update docs...
```

context-update validates the compliance of ALL docs (including metaspec.md and index.md,
which already existed and were not touched by context-init). If an existing doc does not
follow the CONTEXT_SPEC, the user is asked whether to rewrite it.

In this case, assuming the existing docs comply:

```
CONTEXT UPDATED — data-pipeline

Compliance:
  metaspec.md: compliant
  index.md: compliant
  timeline.md: compliant

Updates:
  metaspec.md: no changes
  index.md: no changes
  timeline.md: no changes

Validation: OK (metaspec ~3.8k, index ~1.8k, timeline ~2.4k tokens)
```

## Notes

1. The existing docs (metaspec.md, index.md) were NOT modified by context-init.
2. The scan ran because timeline.md needed to be created.
3. Phase 8 invoked /context-update, which VALIDATED the existing docs (compliance OK).
4. If the existing docs did not follow the CONTEXT_SPEC, context-update would have offered [R]ewrite or [K]eep.
