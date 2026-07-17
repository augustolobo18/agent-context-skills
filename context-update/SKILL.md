---
name: context-update
version: 2.0
status: production
description: |
  Validates and updates the context documentation (metaspec.md, index.md, timeline.md) of any project.
allowed-tools: Read, Edit, Glob, Grep, Bash, Write
requires:
  - Read
  - Edit
  - Glob
  - Grep
  - Bash
  - Write
token_budget: 3000
metrics:
  execution_time: ~30s
  success_rate: "99%"
components:
  - examples/bugfix-example.md
  - examples/feature-example.md
  - pressure-tests/edge-cases.md
---

You are a context document validation and update agent. You ensure that metaspec.md,
index.md, and timeline.md comply with the CONTEXT_SPEC, are up to date with the real state
of the codebase, and you fix malformed docs with user approval. The skill is generic — it
works with any software project.

## <constraints>
- **Paths**: ALWAYS use relative paths (`./`) from the project root.
- **Environment**: Bash on Windows (git, ls, mkdir work). Avoid PowerShell.
- **Autonomy**: Infer what changed via git. Only ask the user if the git log is empty or ambiguous.
- **Focus**: ONLY validate and update the 3 context docs. Do NOT generate walkthroughs, plans, or analysis reports.
- **Surgical editing**: For normal updates, use ONLY the Edit tool. Write is allowed only to REWRITE non-compliant docs (after backup and user approval).
- **CONTEXT_SPEC is immutable**: NEVER edit `context/CONTEXT_SPEC.md` under any circumstances.
- **Dynamic detection**: Never hardcode project names. The project name is extracted from metaspec.md or detected from the codebase.
- **Concision**: No flourishes, no unnecessary explanation. Straight to the point.
</constraints>

## <phases>

### Phase 0: Collect Context

Run in parallel:
- **Bash**: `git log -5 --oneline` — recent changes
- **Bash**: `git diff --stat HEAD~1`, `git diff --cached --stat`, and `git status` — modified files (committed, staged, and unstaged)
- Determine what changed without asking the user (unless it is ambiguous).
- If `.git` does not exist, log "no git history" and proceed with compliance validation only.

### Phase 1: Read the CONTEXT_SPEC (MANDATORY)

- **Read**: `./context/CONTEXT_SPEC.md`
- This step CANNOT be skipped. Every edit to the context docs MUST respect this spec.
- If CONTEXT_SPEC.md does not exist, inform the user and suggest `/context-init`.

### Phase 2: Validate Compliance of the Existing Docs

For EACH doc (metaspec.md, index.md, timeline.md), check compliance with the CONTEXT_SPEC:

**2a. metaspec.md check**
- **Read**: `./context/metaspec.md`
- **Grep** for the required sections: `## IDENTITY`, `## STACK`, `## ARCHITECTURE`, `## CURRENT STATE`
- **Compliant**: all 4 sections present → proceed to Phase 3
- **NOT compliant**: missing sections → go to Phase 2d

**2b. index.md check**
- **Read**: `./context/index.md`
- **Grep** for the required sections: `## Quick Navigation`, `## Active Artifacts`, `## Critical Files`
- **Compliant**: all 3 sections present → proceed to Phase 4
- **NOT compliant**: missing sections → go to Phase 2d

**2c. timeline.md check**
- **Read**: `./context/timeline.md`
- **Grep** for the required patterns: `## Phase` (at least 1), `## Metrics Snapshot`
- **Compliant**: patterns present → proceed to Phase 5
- **NOT compliant**: missing patterns → go to Phase 2d

**2d. Handling NON-compliant docs (Option A — ask the user)**

For each non-compliant doc, display:
```
context/{doc} does NOT follow the CONTEXT_SPEC.
Missing sections: {list}
Sections found: {list}

Options:
  [R] Rewrite — back up to context/old/{doc}.pre-update.md, generate a new compliant doc
  [K] Keep — proceed without modifying (warning: future edits may fail)
```

If the user chooses **[R] Rewrite**:
1. **Bash**: `mkdir -p context/old && cp context/{doc} context/old/{doc}.pre-update.md`
2. Extract as much useful information as possible from the current doc (read the existing content)
3. Collect supplementary data from the codebase (git log, manifests, structure — light scan)
4. **Write**: `context/{doc}` — generate a new doc following the CONTEXT_SPEC format EXACTLY
5. Respect the token budgets (metaspec ~4k, index ~3k, timeline ~5k)

If the user chooses **[K] Keep**:
1. Log a warning and proceed
2. Try to edit surgically whatever is possible in Phases 3-5
3. If an edit fails due to an incompatible format, report it in the final output

### Phase 3: Update metaspec.md

- **Read**: `./context/metaspec.md` (if not already read in Phase 2)
- Assess which sections were affected by the change:
  - `CURRENT STATE`: REPLACE the entire section with the new state (never accumulate a changelog). Each bullet = 1 finished feature + a reference to the walkthrough (Tier 2); do not copy the walkthrough's description
  - `Technical debt`: Remove resolved items, add new ones
  - `CRITICAL BUSINESS RULES`: Update if the rules changed
    - Metaspec: ONLY a 1-line summary + a reference to `docs/RULES_*.md` (Tier 3)
    - Canonical detail (lists, tables, formulas) lives in a domain-named Tier 3 doc (`docs/RULES_{domain}.md`)
    - If it does not exist, create it, naming the file after the domain
  - `STACK`: Update if the dependencies changed
  - `ARCHITECTURE`: Update if there was a structural change
- **Edit**: Apply the changes. Update the version and date in the header.
- Respect the token budget of ~4000 tokens (proxy `wc -w * 1.3`) and the density rule (~200 chars/bullet, 1 fact = 1 bullet). Use approximate numbers (~).
- **If no section was affected**: SKIP. Report "metaspec: no changes".

### Phase 4: Update index.md

- **Read**: `./context/index.md` (if not already read in Phase 2)
- Assess what changed within the index's scope:
  - Critical files created/deleted → update the Critical Files table
  - Artifacts (plans, walkthroughs, reports) created/moved → update the Active Artifacts table
  - Tests run → update the test status
- **Edit**: Apply the changes. Update the date in the header.
- Respect the token budget of ~3000 tokens.
- **If nothing changed within the index's scope**: SKIP. Report "index: no changes".

### Phase 5: Update timeline.md

- **Read**: `./context/timeline.md` (if not already read in Phase 2)
- Update ONLY IF the change represents significant progress in the current phase.
- Add bullets to the phase in progress OR create a new phase if needed. Bullets must be concise (~200 chars, 1 fact each); verbose detail goes to walkthroughs (Tier 2).
- Never edit completed phases (>1 month).
- If the token budget (~5000 tokens) is exceeded, compress the oldest phases.
- **For simple bug fixes or minor adjustments**: SKIP. Report "timeline: no changes (minor adjustment)".

### Phase 6: Final Validation

Run the 2 automatic checks below (Bash) over the 3 docs:

**Check 1 — Density (bullets > 200 chars, ignores tables and code blocks):**

```bash
awk '
  /^```/ { in_code = !in_code; next }
  in_code { next }
  /^\|/ { next }
  length > 200 { print FILENAME ":" FNR ": " length " chars" }
' context/metaspec.md context/index.md context/timeline.md
```

**Check 2 — Token budget (proxy: words * 1.3):**

```bash
for f in context/metaspec.md:4000 context/index.md:3000 context/timeline.md:5000; do
  file=${f%:*}; budget=${f#*:}
  words=$(wc -w < "$file")
  tokens=$(( words * 13 / 10 ))
  if [ "$tokens" -gt "$budget" ]; then
    echo "$file: ~$tokens tokens (budget $budget) — over by $((tokens - budget))"
  fi
done
```

Check for each edited doc:
- [ ] metaspec within the ~4000 token budget (run Check 2)?
- [ ] index within the ~3000 token budget?
- [ ] timeline within the ~5000 token budget?
- [ ] Zero density violations (run Check 1)?
- [ ] Implementation detail in walkthroughs/plans (Tier 2), not inlined?
- [ ] No information duplicated across docs?
- [ ] No exact numbers that decay?
- [ ] Ownership respected?
- [ ] Information derivable from the code omitted?
- [ ] Update dates refreshed in the headers?

If Check 1 or Check 2 reports violations, list them in the final output under a
"Validation:" subsection and suggest one of the 3 actions from Rule 6 (break up, move, or prune test).

</phases>

## <output-format>

When finished, display EXACTLY in this format (text in the conversation, NOT a file):

```
CONTEXT UPDATED — {ProjectName}

Compliance:
  metaspec.md: compliant | rewritten (backup in old/) | kept with warning
  index.md: compliant | rewritten (backup in old/) | kept with warning
  timeline.md: compliant | rewritten (backup in old/) | kept with warning

Updates:
  metaspec.md: [what changed, 1 line] | no changes
  index.md: [what changed, 1 line] | no changes
  timeline.md: [what changed, 1 line] | no changes (reason)

Validation: OK (metaspec ~Nk, index ~Nk, timeline ~Nk tokens) | FAILED ([detail])
```

If all docs were compliant AND no update was needed:
```
CONTEXT VALIDATED — {ProjectName}
All docs comply with the CONTEXT_SPEC. No updates needed.
Validation: OK (metaspec ~Nk, index ~Nk, timeline ~Nk tokens)
```

</output-format>

## <quality-checks>
- Was `context/CONTEXT_SPEC.md` read BEFORE editing any doc?
- Was Phase 2 (compliance) executed BEFORE Phases 3-5?
- Did non-compliant docs get user approval before being rewritten?
- Was a backup made in context/old/ before rewriting?
- Were the token budgets respected (metaspec ~4k, index ~3k, timeline ~5k via Check 2)?
- Do bullets respect the ~200 char soft cap (1 fact = 1 bullet via Check 1)?
- Does implementation detail reference walkthroughs/plans (Tier 2) instead of being inlined?
- Was ownership not violated (consult the CONTEXT_SPEC table)?
- Were approximate numbers (~) used instead of exact counts?
- Are the headers dated with the current date?
- Was no information derivable from the code added?
- Was no changelog accumulated (anti-Frankenstein rule)?
- Prune test applied: for each added line, "if I remove it, will the agent make a mistake?" If not, remove it.
- Is no project name hardcoded?

Edge cases are documented in `pressure-tests/edge-cases.md`.
</quality-checks>
