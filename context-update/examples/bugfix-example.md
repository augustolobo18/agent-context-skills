# Example: Simple Bug Fix

## Scenario

Fixing a broken import in a dashboard component.

## Input (git log)

```
a1b2c3d fix: correct import in PerformanceChart
```

```
git diff --stat HEAD~1:
  src/components/PerformanceChart.tsx | 2 +-
  1 file changed, 1 insertion(+), 1 deletion(-)
```

## Skill Decision

| Doc | Compliance | Action | Reason |
|-----|------------|--------|--------|
| metaspec.md | compliant | UPDATE | Remove the resolved technical debt |
| index.md | compliant | SKIP | No file created/deleted |
| timeline.md | compliant | SKIP | A simple bug fix is not significant progress |

## Edit Applied

**metaspec.md** — CURRENT STATE section, technical debt:

Before:
```markdown
## CURRENT STATE (v2.3 — 28/03/2026)
- Admin panel with settings management (CRUD). Dashboard stable.

Technical debt:
- Broken import in PerformanceChart causes a build error
- ~3 components without an error boundary
```

After:
```markdown
## CURRENT STATE (v2.3 — 30/03/2026)
- Admin panel with settings management (CRUD). Dashboard stable.

Technical debt:
- ~3 components without an error boundary
```

Header updated: `Updated: 2026-03-30`

## Terminal Output

```
CONTEXT UPDATED — task-tracker

Compliance:
  metaspec.md: compliant
  index.md: compliant
  timeline.md: compliant

Updates:
  metaspec.md: removed resolved technical debt (PerformanceChart import)
  index.md: no changes
  timeline.md: no changes (simple bug fix)

Validation: OK (metaspec ~3.7k, index ~2.1k, timeline ~4.2k tokens)
```

## Key Points

- Simple bug fixes affect ONLY the metaspec (technical debt).
- The timeline only receives changes that represent progress in the current phase.
- The CURRENT STATE section does not need to change for a one-off fix.
