# Pressure Test Scenarios — context-loader v2.0

---

## Scenario 1: Project without a context/ folder (invokes context-init)

### Setup
The `context/` directory does not exist. No metaspec, index, or timeline.

### Expected Behavior
- Phase 0 detects 0/3 docs
- Invokes `/context-init` (full init)
- context-init creates everything + invokes context-update
- On return, proceeds to Phase 1 and loads normally
- Briefing displayed with data from the newly created docs

### Validation
- /context-init was invoked
- All docs were loaded after creation
- Briefing displayed normally

### Status
VALIDATED

---

## Scenario 2: Complete structure (direct load)

### Setup
Project with context/metaspec.md, index.md, timeline.md, CONTEXT_SPEC.md, CLAUDE.md, artifacts, and docs.

### Expected Behavior
- Phase 0 detects 3/3
- Does NOT invoke /context-init
- Phases 1-4 load everything
- Complete briefing

### Validation
- No external skill invoked
- All files loaded
- Briefing with all sections filled in

### Status
VALIDATED

---

## Scenario 3: Partial (1/3 docs exist)

### Setup
Only context/metaspec.md exists. index.md and timeline.md are missing.

### Expected Behavior
- Phase 0 detects 1/3
- Invokes `/context-init` (partial: creates index + timeline)
- context-init invokes context-update (validates compliance)
- On return, loads everything

### Validation
- /context-init invoked in partial mode
- Missing docs created
- Full load after init

### Status
VALIDATED

---

## Scenario 4: Empty MEMORY.md

### Setup
MEMORY.md exists but is empty (0 bytes).

### Expected Behavior
- Phase 4 reads the file and detects it is empty
- No referenced file is looked up
- Briefing: "Memory: no items recorded"

### Validation
- Does not try to read non-existent files
- Briefing generated normally

### Status
VALIDATED

---

## Scenario 5: MEMORY.md does not exist

### Setup
The project's memory directory does not exist.

### Expected Behavior
- Read returns a file-not-found error
- Logs "MEMORY.md not found" and continues
- Briefing: "Memory: memory file not found"

### Validation
- The skill does not stop execution
- All other phases complete normally

### Status
VALIDATED

---

## Scenario 6: Many active plans (10+)

### Setup
The `plans/` directory contains 12 active .md files.

### Expected Behavior
- Glob returns 12 files
- Read ALL 12
- Briefing lists them all under "Active plans"

### Validation
- All 12 plans loaded
- Token estimate reflects the volume
- Briefing not truncated

### Status
VALIDATED

---

## Scenario 7: old/ folder with many files

### Setup
`context/walkthroughs/old/` contains 30 files. `plans/old/` contains 20.

### Expected Behavior
- The `context/walkthroughs/*.md` and `plans/*.md` glob patterns return ONLY the root
- NO file from `old/` is read
- The count reflects only the active artifacts

### Validation
- Zero files from old/ in the results
- Token estimate not inflated

### Status
VALIDATED

---

## Scenario 8: No active plans

### Setup
`plans/` exists but is empty (all moved to `old/`).

### Expected Behavior
- Glob returns an empty list
- Briefing: "Active plans: none"

### Validation
- No empty section or broken formatting
- Briefing is readable

### Status
VALIDATED

---

## Scenario 9: Technical docs referenced in index.md

### Setup
index.md has a `## Technical Documentation` section listing 3 docs. 2 exist, 1 was deleted.

### Expected Behavior
- Phase 3 extracts the 3 paths from index.md
- Lists them in the briefing as available on demand
- Does NOT read them (Tier 3 is listing-only)
- The deleted file's absence surfaces only if the agent later tries to read it

### Validation
- 3 paths listed in the briefing
- Zero Read calls for docs/ during Phase 3
- No error, since no read is attempted

### Status
VALIDATED

---

## Scenario 10: index.md without a Technical Documentation section

### Setup
index.md exists and is compliant, but has no `## Technical Documentation` section (project without docs/).

### Expected Behavior
- Phase 3 detects the missing section
- SKIPS the entire phase
- No technical doc listed
- Proceeds to Phase 4

### Validation
- No error raised
- Phase 3 skipped gracefully

### Status
VALIDATED

---

## Scenario 11: Read permissions denied

### Setup
A file in `context/analysis/` has restricted permissions.

### Expected Behavior
- Read returns a permission error
- Logs "Error reading [file]: permission denied"
- All other files read normally

### Validation
- The skill does not stop
- Briefing generated with partial data

### Status
VALIDATED

---

## Scenario 12: No walkthrough or analysis report

### Setup
`context/walkthroughs/` and `context/analysis/` exist but are empty.

### Expected Behavior
- Glob returns empty lists
- Briefing: "Last implementation: no walkthrough found"
- Briefing: "Analysis reports available (on demand): none"

### Validation
- The skill does not fail on empty lists
- Briefing correctly formatted

### Status
VALIDATED

---

## Scenario 13: Project name detected from the metaspec

### Setup
metaspec.md has the header `# MetaSpec — data-pipeline`.

### Expected Behavior
- Phase 1 extracts "data-pipeline" from the header
- Briefing uses "CONTEXT [data-pipeline] LOADED"

### Validation
- Name is not hardcoded
- Name comes from the metaspec, not from the directory

### Status
VALIDATED

---

## Scenario 14: MEMORY.md with broken references

### Setup
MEMORY.md references `[Project X](project_x.md)` but the file does not exist.

### Expected Behavior
- Identifies the reference, tries to read it
- Read returns an error
- Logs "project_x.md not found" and continues
- Briefing includes the readable items from MEMORY.md

### Validation
- Does not enter a loop
- Partial briefing generated

### Status
VALIDATED
