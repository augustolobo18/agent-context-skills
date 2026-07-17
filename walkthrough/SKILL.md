---
skill_id: walkthrough
name: walkthrough
version: 3.1
status: production
description: |
  Generates technical documentation of completed implementations with support for diagrams, metrics, and automatic project detection.
  Use when an implementation is finished and needs to be documented technically before a commit or review.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
disable-model-invocation: false
token_budget: 10000
metrics:
  execution_time: "2-5 min"
  avg_doc_size: "100-300 lines"
  success_rate: "99%"
components:
  - data-collection.md
  - mermaid-guidelines.md
  - pattern-learning.md
  - visual-templates.md
  - report-template.md
  - resources/section-keywords.yaml
  - examples/refactoring-walkthrough.md
  - pressure-tests/test-scenarios.md
---

You are a generator of analytical technical walkthroughs. Your job is to document code implementations completely and in a structured way, adapting to any repository.

## <strategy>
1. **Argument interpretation**: The user may call the command with parameters. Analyze the prompt that triggered this skill:
   - `--visual-level=minimal` (tables only)
   - `--visual-level=standard` (tables + ASCII tree + simple Mermaid diagrams. This is the **DEFAULT** if unspecified)
   - `--visual-level=detailed` (all visual elements, pie charts, sequence/state diagrams)
2. **Terrain recognition**: Discover where the current project saves its documentation and the context of the latest changes.
3. **Collection and generation**: Use Git to extract the real data of the implementation and generate a rich, structured Markdown file.
</strategy>

## <configuration>
| Parameter | Values | Default | Description |
|-----------|--------|---------|-------------|
| `--visual-level` | `minimal`, `standard`, `detailed` | `standard` | Level of visual elements (tables, ASCII tree, Mermaid) |
</configuration>

## <constraints>
- **Paths**: ALWAYS use relative paths (`./`) from the project root. NEVER use absolute paths such as `C:\Users\...`.
- **Environment**: The terminal is compatible with Bash commands running on Windows (git, npm, pytest work). Avoid native PowerShell syntax.
- **Autonomy vs interaction**: If the git log is clear about what was just done, do not ask questions — generate the document. If it is too confusing or empty, quickly ask the user which implementation should be documented.
- **Fidelity**: Test results and metrics must be REAL, extracted from the tools. Do not invent data.
</constraints>

## <phases>
### Phase 1: Setup & Pattern Learning (1-2 min)
Run in parallel:
- **Glob**: Search for `./context/walkthroughs/*.md`, `./docs/walkthroughs/*.md`, `./documentation/walkthroughs/*.md`, or `./walkthroughs/*.md`.
  - The first directory that returns results is set as `[OUTPUT_DIR]`.
  - Read 2 files from that directory (if any exist) to imitate the project's tone and structure.
  - If no directory exists, set `[OUTPUT_DIR]` to `./context/walkthroughs/` and create the folder using bash.
- **Bash**: `git log -5 --oneline` (to pick up the latest changes if the user did not specify what to document).
- **Bash**: `git diff HEAD~1 --stat` or `git status` (to map the modified files).

**Expected output:** `[OUTPUT_DIR]` set, tone/structure patterns learned, git context loaded.

### Phase 2: Data Collection & Test Execution (1-2 min)
If the user asks, or if a standard test ecosystem is detected (e.g. `package.json` with a `test` script, or `pytest`):
- Run the tests and capture the last 30 lines of output. Example: `npm test 2>&1 | tail -30` or `pytest -v --tb=short 2>&1 | tail -30`.

### Phase 3: Visual Elements Generation
Based on `--visual-level` (default is *standard* if the user does not specify anything), prepare the following elements in Markdown/Mermaid:
- **Minimal**: Tables of modified/created files and basic line metrics.
- **Standard**: Tables + structured ASCII tree of the affected directories + 1 Mermaid diagram (before/after flowchart or a simple architecture diagram).
- **Detailed**: Tables + ASCII tree + Sequence Diagram (interactions) + State Diagram (if applicable) + Pie Chart (distribution of modified files/languages generated in Mermaid).

### Phase 4: Document Generation & Output
1. Structure the document with the following sections (adjust according to the template learned in Phase 1):
   - Header (Title, Date YYYY-MM-DD, Status)
   - 1. Implementation Summary
   - 2. Changes Made (including the visualizations generated in Phase 3)
   - 3. Real Test Results
   - 4. Attention Points / Limitations / Technical Debt
   - 5. Commit Suggestion (Conventional Commits standard)
2. **Write**: Save the file to `[OUTPUT_DIR]/[YYYY-MM-DD]_Walkthrough_[Short_Name].md`.
3. **Bash**: Open the finished file using `start "" "[OUTPUT_DIR]/[YYYY-MM-DD]_Walkthrough_[Short_Name].md"`.

### Phase 5: Update Context Documents (Mandatory)

Invoke `/context-update` to update the project's context documentation.
</phases>

## <quality-checks>
- Was the final file saved to a valid relative path?
- Does the Mermaid visual level match the requested `--visual-level`?
- Was the file opened at the end of the process?
- Was `/context-update` invoked at the end (Phase 5)?
</quality-checks>
