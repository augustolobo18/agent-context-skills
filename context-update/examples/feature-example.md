# Example: New Feature

## Scenario

Adding a settings page to the admin panel, with a new component and a new route.

## Input (git log)

```
f4e5d6c feat: add SettingsPage to the admin panel
b7a8c9d feat: PUT /api/settings endpoint for editing configs
```

```
git diff --stat HEAD~2:
  src/api/routes/settings.ts                    |  28 ++++
  src/domain/settings.ts                        |  15 +++
  src/components/admin/SettingsPage.tsx         | 142 +++++++++++++++++
  src/pages/AdminPage.tsx                       |   8 +-
  4 files changed, 190 insertions(+), 3 deletions(-)
```

## Skill Decision

| Doc | Compliance | Action | Reason |
|-----|------------|--------|--------|
| metaspec.md | compliant | UPDATE | Current state changed (new admin capability) |
| index.md | compliant | UPDATE | New critical file (SettingsPage.tsx) |
| timeline.md | compliant | UPDATE | Significant progress in the current phase |

## Edits Applied

**metaspec.md** — CURRENT STATE section (REPLACE, do not accumulate):

```markdown
## CURRENT STATE (v2.3 — 30/03/2026)
- Admin panel with settings management (CRUD). Dashboard stable.
- Working REST API with rate limiting and JWT auth.
```

Header updated: `Updated: 2026-03-30`

**index.md** — Critical Files table, UI section:

Added:
```markdown
| `src/components/admin/SettingsPage.tsx` | Settings CRUD in the admin panel |
```

**timeline.md** — bullet in the current phase:

```markdown
## Phase 3: Admin and Settings (Mar/2026)
- Admin panel with settings management (SettingsPage + PUT endpoint)
```

## Terminal Output

```
CONTEXT UPDATED — task-tracker

Compliance:
  metaspec.md: compliant
  index.md: compliant
  timeline.md: compliant

Updates:
  metaspec.md: current state updated (settings management in the admin panel)
  index.md: added SettingsPage.tsx to the critical files table
  timeline.md: new bullet in Phase 3 (settings management)

Validation: OK (metaspec ~3.9k, index ~2.2k, timeline ~4.4k tokens)
```

## Key Points

- New features typically affect all 3 docs.
- CURRENT STATE is REPLACED wholesale (anti-Frankenstein) — it does not receive an append.
- The index only receives files that are entry points or architectural decisions.
- The timeline receives 1 concise bullet per feature, not 1 per commit.
