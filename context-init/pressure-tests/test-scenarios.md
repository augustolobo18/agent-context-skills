# Pressure Test Scenarios — context-init

Edge case scenarios to validate the skill's robustness.

---

## Scenario 1: Empty Project (no files)

### Setup
Empty directory, no .git, no source files at all.

### Expected Behavior
- Phase 0: nothing exists, full init
- Phase 1: scan returns empty for everything; name = directory name
- metaspec: minimal sections with "to be defined"
- timeline: "no git history" format
- index: only Quick Navigation and Active Artifacts (empty)
- Do NOT fail or abort

### Validation
- 4 docs created without errors
- No remaining `{{...}}` placeholder in any field
- Token budgets respected

### Status
VALIDATED

---

## Scenario 2: Complete Structure (everything already exists)

### Setup
Project with context/CONTEXT_SPEC.md, metaspec.md, index.md, timeline.md, and all directories.

### Expected Behavior
- Phase 0 detects everything as existing
- Phases 1-6 SKIPPED (nothing to create)
- Phase 7 displays "CONTEXT ALREADY EXISTS — {ProjectName}. All docs and directories are already present."
- Phase 8 invokes /context-update to validate compliance of the existing docs
- Does NOT create any file or directory

### Validation
- No Write call executed
- No Bash mkdir executed
- /context-update WAS invoked (Phase 8 always runs)
- Output includes the delegation message to context-update

### Status
VALIDATED

---

## Scenario 3: Monorepo Project (multiple package.json)

### Setup
Monorepo with `package.json` at the root and in `packages/web/package.json`, `packages/api/package.json`.

### Expected Behavior
- Use the ROOT `package.json` for the project name (not the sub-packages)
- Stack must list technologies from all relevant sub-packages
- Architecture must reflect the monorepo structure

### Validation
- Name comes from the root package.json
- ASCII diagram shows the monorepo structure
- Critical files grouped by package

### Status
VALIDATED

---

## Scenario 4: Python Project without pyproject.toml

### Setup
Python project with only `requirements.txt` and `setup.py`. No pyproject.toml, no Pipfile.

### Expected Behavior
- Name detected from `setup.py` (name field), or the .git remote, or the directory name
- Stack detected via .py extensions and requirements.txt
- Python version inferred from `runtime.txt`, `.python-version`, or not reported

### Validation
- Name is not "." or an empty string
- Stack lists Python and the requirements.txt dependencies
- Does not fail due to the absence of pyproject.toml

### Status
VALIDATED

---

## Scenario 5: Project without Git

### Setup
Project with code but no `.git` directory (not initialized, or .git deleted).

### Expected Behavior
- Phase 1d: log "no git history" and continue
- timeline.md: minimal "no git history" format (see Phase 6 of the skill)
- metaspec CURRENT STATE: no branch, no hash
- All other phases work normally

### Validation
- No error caused by the missing git
- timeline created in the minimal format
- metaspec has no empty or "null" branch field

### Status
VALIDATED

---

## Scenario 6: Partial — Only timeline.md is missing

### Setup
Project with CONTEXT_SPEC.md, metaspec.md, index.md, and all directories. Only timeline.md is missing.

### Expected Behavior
- Phase 0: 3/4 docs exist, 1 to create
- Phase 1: scan runs (required for the timeline)
- Phases 3-5: SKIPPED (docs already exist)
- Phase 6: timeline.md created
- Output lists 1 created and 3+3 already existing

### Validation
- Only 1 Write call (timeline.md)
- No existing doc touched
- Report is correct

### Status
VALIDATED

---

## Scenario 7: Go Project with go.mod

### Setup
Go project with `go.mod`, `cmd/main.go`, `internal/`, `pkg/`.

### Expected Behavior
- Name extracted from `go.mod` (last segment of the module path)
- Stack: Go + version from go.mod
- Architecture: standard Go layout (cmd/, internal/, pkg/)
- Dependencies from go.mod listed

### Validation
- Name is correct (not the full module path)
- Stack does not list "Node.js" or "Python"
- Go directory structure reflected in the architecture

### Status
VALIDATED

---

## Scenario 8: Rust Project with Cargo.toml

### Setup
Rust project with `Cargo.toml`, `src/main.rs`, `src/lib.rs`.

### Expected Behavior
- Name from `[package].name` in Cargo.toml
- Stack: Rust + edition from Cargo.toml
- Dependencies from `[dependencies]` listed
- Entry point detected as src/main.rs

### Validation
- Name is correct
- Stack includes the Rust edition
- Architecture reflects the src/ structure

### Status
VALIDATED

---

## Scenario 9: Java Project with Maven (pom.xml)

### Setup
Java project with `pom.xml`, `src/main/java/`, `src/test/java/`.

### Expected Behavior
- Name from `<artifactId>` in pom.xml
- Stack: Java + version from `<java.version>` or `<maven.compiler.source>`
- Framework detected from dependencies (Spring Boot, etc.)
- Maven structure reflected in the architecture

### Validation
- Name is correct (artifactId, not groupId)
- Stack includes the Java version
- Tests detected under src/test/

### Status
VALIDATED

---

## Scenario 10: Permissions — context/ directory is not writable

### Setup
The `context/` directory exists but is not writable (chmod 444 on Linux, readonly on Windows).

### Expected Behavior
- mkdir -p may fail
- Write may fail
- The skill must report the error clearly without aborting entirely
- Docs that could be created are reported; those that failed are listed with the error

### Validation
- Does not raise an unhandled exception
- Output indicates which docs failed
- Successfully created docs are reported normally

### Status
VALIDATED

---

## Scenario 11: Project with a very large README.md (>1000 lines)

### Setup
Project with a 2000-line README.md (extensive documentation).

### Expected Behavior
- Read only the first ~100 lines of the README to extract purpose and description
- Do NOT load the entire README into the context window
- Use the Read tool's `limit` parameter

### Validation
- Read called with a limit for the README
- Purpose correctly extracted from the first lines
- Does not consume excessive tokens

### Status
VALIDATED

---

## Scenario 12: Project with special characters in the name

### Setup
Project whose package.json name is `@my-org/super-app` (scoped npm package).

### Expected Behavior
- Full name `@my-org/super-app` used as the identity (per SKILL.md Phase 1a)
- Do not break paths or YAML because of the @
- Doc headers use the full name: `# MetaSpec — @my-org/super-app`

### Validation
- Name is `@my-org/super-app` (full name, not just `super-app`)
- No doc with broken characters
- Doc headers correctly formatted

### Status
VALIDATED

---

## Scenario 13: Mixed project (Python + TypeScript)

### Setup
Project with a Python backend (FastAPI) and a TypeScript frontend (React). Structure: `backend/`, `frontend/`.

### Expected Behavior
- Stack lists BOTH languages and frameworks
- Architecture reflects the backend/frontend separation
- Dependencies from BOTH manifests (requirements.txt + package.json)
- Critical files grouped by layer (Backend, Frontend)

### Validation
- Both languages appear in STACK
- ASCII diagram shows the Frontend -> Backend -> DB flow
- index.md has sections for both layers

### Status
VALIDATED
