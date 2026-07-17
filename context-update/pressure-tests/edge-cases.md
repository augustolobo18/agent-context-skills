# Pressure Tests: Edge Cases — context-update v2.0

---

## 1. metaspec.md at Its Token Budget

**Setup**: metaspec.md is already at ~4000 tokens and needs to receive new information.
**Expected behavior**:
- Apply the prune test to every section
- Compress long descriptions (e.g. 2 similar bullets become 1)
- Remove information derivable from the code (e.g. file lists obtainable with `ls`)
- If still over ~4000 tokens after compression: inform the user and suggest what to remove
**Status**: VALIDATED

## 2. Empty Git Log

**Setup**: New session, no recent commits, `git log -5` shows no relevant changes.
**Expected behavior**:
- Check `git status` and `git diff --cached` for uncommitted changes
- If everything is clean: ask the user "What changed in this session?"
- Do NOT invent changes. Do NOT update docs without evidence.
**Status**: VALIDATED

## 3. Ownership Conflict

**Setup**: A change that could be documented in the metaspec OR the timeline (e.g. "migration from SQLite to PostgreSQL").
**Expected behavior**:
- Consult the ownership table in CONTEXT_SPEC.md
- Stack/architecture → metaspec.md (owner)
- Historical progress → timeline.md (owner)
- In this case: the metaspec receives the stack change, the timeline receives a progress bullet
- Information NOT duplicated across docs
**Status**: VALIDATED

## 4. Context File Does Not Exist

**Setup**: `context/metaspec.md` does not exist (new project, or the file was deleted).
**Expected behavior**:
- Inform the user: "metaspec.md not found in context/"
- Do NOT create a new file (creation is context-init's responsibility)
- Suggest: "Run /context-init to create the missing docs"
- Continue updating the docs that do exist
**Status**: VALIDATED

## 5. Multiple Uncommitted Changes

**Setup**: Files modified in backend/, frontend/, and docs/ without a commit.
**Expected behavior**:
- Use `git diff --stat` and `git status` to map all the changes
- Group by impact: which sections of each doc are affected
- Apply all edits in a single pass (not one per file)
- If ambiguous (many unrelated changes): list them and ask for confirmation
**Status**: VALIDATED

## 6. timeline.md Over Its Token Budget

**Setup**: timeline.md is at ~5400 tokens and needs a new bullet.
**Expected behavior**:
- BEFORE adding, compress older phases (>2 months old)
- Compression: keep the phase title + 1-2 summary bullets, remove the detail
- Recent phases (last month) kept intact
- Final result within the ~5000 token budget
**Status**: VALIDATED

## 7. Change in Tests Only

**Setup**: Commits add/modify only files under `tests/` or `*.test.ts`.
**Expected behavior**:
- metaspec.md: SKIP (tests do not affect state/stack/architecture)
- index.md: Update the test status IF the Tests section exists
- timeline.md: SKIP (tests are not feature progress)
- Exception: if the tests reveal a bug documented as technical debt → update the metaspec
**Status**: VALIDATED

## 8. Rollback / Revert of a Feature

**Setup**: `git revert` of a commit that added a feature already documented in the 3 docs.
**Expected behavior**:
- metaspec.md: Revert the current state to before the feature
- index.md: Remove critical files that the revert deleted
- timeline.md: Add a "Reverted: [feature]" bullet in the current phase (do not delete the original bullet — history is immutable)
**Status**: VALIDATED

---

## 9. metaspec.md NOT Compliant — Missing Sections

**Setup**: metaspec.md exists but has no `## IDENTITY` or `## ARCHITECTURE` sections. The content is free-form text with no structure.
**Expected behavior**:
- Phase 2a detects the missing sections via Grep
- Displays the options to the user: [R] Rewrite / [K] Keep
- If [R]: back up to context/old/metaspec.pre-update.md, extract useful info from the current doc, generate a compliant one
- If [K]: log a warning, try to edit whatever is possible
**Status**: VALIDATED

## 10. index.md NOT Compliant — Completely Different Format

**Setup**: index.md exists but is a simple README-style bullet list, with no tables and none of the CONTEXT_SPEC sections.
**Expected behavior**:
- Phase 2b detects the missing sections
- Displays the options to the user
- If [R]: backup + full rewrite using data from the codebase
- If [K]: warning in the output, does not attempt to edit (incompatible format)
**Status**: VALIDATED

## 11. timeline.md NOT Compliant — No Phases

**Setup**: timeline.md exists but is a simple chronological list with no `## Phase` or `## Metrics Snapshot`.
**Expected behavior**:
- Phase 2c detects the missing patterns
- Displays the options to the user
- If [R]: backup + rewrite, grouping the existing content into phases
- If [K]: warning, tries to append a new phase at the end
**Status**: VALIDATED

## 12. All Docs Compliant, No Changes

**Setup**: Docs follow the CONTEXT_SPEC, no recent commits, git status clean.
**Expected behavior**:
- Phase 2 confirms compliance for all
- Phases 3-5 detect "no changes" for all
- Output: "CONTEXT VALIDATED — All docs comply with the CONTEXT_SPEC. No updates needed."
**Status**: VALIDATED

## 13. CONTEXT_SPEC.md Does Not Exist

**Setup**: context/ exists with metaspec, index, timeline, but no CONTEXT_SPEC.md.
**Expected behavior**:
- Phase 1 fails to read CONTEXT_SPEC.md
- Inform the user: "CONTEXT_SPEC.md not found. Run /context-init to create it."
- Do NOT proceed with edits without the reference spec
**Status**: VALIDATED

## 14. Partially Compliant Doc

**Setup**: metaspec.md has `## IDENTITY` and `## STACK` but is missing `## ARCHITECTURE` and `## CURRENT STATE`.
**Expected behavior**:
- Phase 2a detects 2/4 sections present
- Reports which sections are present and which are missing
- Offers the [R]/[K] options
- If [R]: preserves the content of the existing sections, adds the missing ones
**Status**: VALIDATED

## 15. User Chooses [K] Keep for All

**Setup**: 3 non-compliant docs, the user chooses [K] for all of them.
**Expected behavior**:
- Logs 3 warnings
- Tries to edit surgically whatever is possible
- Final output lists a warning for each doc kept
- Does NOT abort execution
**Status**: VALIDATED

## 16. Project without Git

**Setup**: Project with no `.git`, context docs exist and are compliant.
**Expected behavior**:
- Phase 0: logs "no git history"
- Phase 2: validates compliance normally (does not depend on git)
- Phases 3-5: no change data, no update applied
- Output: "CONTEXT VALIDATED — No updates (no git history)"
**Status**: VALIDATED
