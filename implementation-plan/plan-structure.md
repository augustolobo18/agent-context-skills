# Plan Structure

**Component of:** implementation-plan
**Purpose:** Define the exact format and sections of an implementation plan

## Required Sections

Every implementation plan MUST contain these sections in order (Section 0 first):

### Header Block

```markdown
# Implementation Plan: [Feature Name]

**Context:** [1-2 sentences describing the problem and why this is needed]
**Tech Stack:** [Technologies, frameworks, libraries involved]
```

### Section 0: Context to Load (Required Reading)

```markdown
## 0. Context to Load

> Read EVERY item below BEFORE starting execution. These are the files that hold the
> conventions, domain rules, and patterns needed to implement this plan without
> hallucinating. Read the CURRENT version on disk — do not trust memory or assumption.

**Context docs (conventions and rules):**
- [ ] `CLAUDE.md` — agent guidelines (if it exists)
- [ ] `context/metaspec.md` — stack, architecture, state
- [ ] `context/index.md` → section [X] — [rule/domain relevant to this feature]
- [ ] [other technical docs from index.md relevant to the files being touched]

**Reference code (patterns to follow):**
- [ ] `path/to/similar_file.ext` — [type] pattern to imitate
- [ ] [neighboring files that establish the convention]

**Files to modify (read the current state before altering):**
- [ ] `path/to/file_to_modify.ext` — will be changed in task [N.N]
```

Rules for this section:
- List ONLY paths (pointers), NEVER paste file contents. The cost of embedding content
  (output) outweighs the savings; snapshots also go stale. The executor reads the source
  of truth live.
- Derive the paths directly from the exploration done in the Discovery Phase (the files
  you have already read/identified as relevant).
- Every file with a `[MODIFY: ...]` or `[DELETE: ...]` tag in the phases MUST appear in
  "Files to modify".
- Point to specific sections (`index.md → section X`) when the doc is large, so the
  executor does not load more than it needs.

### Section 1: Goals & Scope

```markdown
## 1. Goals & Scope

### 1.1. Goals

* **Goals:** [The objectives of this plan]

### 1.2. Scope

* **Inputs:** [What inputs/data the feature receives]
* **Outputs:** [What outputs/results the feature produces]
* **In-Scope:** [What the agent WILL do]
* **Out-of-Scope:** [An action the agent will NOT take — starts with a verb: "Do not edit X"]
* **Out-of-Scope:** [Another action the agent will NOT take]
* **Constraint:** [A property of the system/result that MUST stay true — asserts state: "X must remain Y"]
* **Constraint:** [Additional invariant or fixed rule as needed]
```

Rules for this section (how to tell Out-of-Scope from Constraint):

- **Out-of-Scope is about the AGENT; Constraint is about the SYSTEM.** If the line
  describes an ACTION the agent will not take, it is Out-of-Scope. If it describes a
  PROPERTY of the system/result that must stay true, it is a Constraint.
- Violation test: breaking an Out-of-Scope = "I did extra work" (scope creep); breaking a
  Constraint = "I did the work wrong / broke something" (regression).
- Out-of-Scope starts with an action verb ("Do not edit X", "Do not include Y") and lists
  ONLY exclusions someone could reasonably assume were included (adjacent to the work). Do
  not enumerate the universe of things not done.
- Constraint asserts a state/invariant ("X must stay Y", "the change must not break Z").
  It covers invariants to preserve AND fixed rules (business, stack, contract) the
  solution must respect.
- When a "do not touch X" exists because the effect must be achieved another way, write it
  as a Constraint — the "do not touch" is a consequence, not a separate rule.
- If a line carries both natures, split it in two.

Minimum: 1 In-Scope, 2 Out-of-Scope, 1 Constraint, 1 Input, 1 Output

### Section 2: Technical Design

```markdown
## 2. Technical Design

### Data Flow
1. **[Step Name]:** [Description of the data transformation]
2. **[Step Name]:** [Next step in the flow]

### Data Structures (Draft)
[Pseudocode or schema — NOT actual implementation code]
```

Keep pseudocode minimal — just enough to communicate intent.

### Section 3: Phased Execution

```markdown
## 3. Phased Execution

### Phase 1: [Phase Name] ([Category])
- [ ] **1.1: [Task Title]** [ADD: path/to/new/file.ext]
    - [Description of what to implement]
    - *Verification:* [How to verify this works]
- [ ] **1.2: [Task Title]** [MODIFY: path/to/existing/file.ext]
    - [Description of the changes]
    - *Verification:* [Verification method]

### Phase 2: [Phase Name]
- [ ] **2.1: [Task Title]** [ADD/MODIFY/DELETE: path]
    ...
```

Rules:
- Maximum 5 phases
- Each task MUST have a file operation tag
- Each task SHOULD have a *Verification:* line
- Tasks numbered as [Phase].[Task] (e.g. 1.1, 1.2, 2.1)

### Section 4: Test Strategy

```markdown
## 4. Test Strategy

- [ ] **Unit:** [What unit tests to create]
- [ ] **Integration:** [What integration tests to create]
- [ ] **[Other]:** [Performance, E2E, etc. as needed]
```

Minimum: 2 test categories

### Section 5: Rollback & Risks

```markdown
## 5. Rollback & Risks

- **Risk:** [What could go wrong]
    - *Mitigation:* [How to prevent or handle it]
- **Risk:** [Another potential issue]
    - *Mitigation:* [Prevention strategy]
- **Rollback:** [How to undo the changes if needed]
```

Minimum: 2 risks with mitigations

### Footer

The plan file does **NOT** include an approval block. Approval happens via
`AskUserQuestion` after the plan is saved and presented to the user (see
`approval-workflow.md`).

The plan ends after Section 5 (Rollback & Risks).

## File Operation Tags

ALWAYS use these tags, even for minor changes:

| Tag | Meaning | Example |
|-----|---------|---------|
| `[ADD: ./path]` | Create a new file | `[ADD: ./src/services/auth.py]` |
| `[MODIFY: ./path]` | Modify an existing file | `[MODIFY: ./src/config.py]` |
| `[DELETE: ./path]` | Remove a file | `[DELETE: ./src/legacy/old_auth.py]` |

## Phase Categories

Common phase categories (use in parentheses):

- **(Core Domain)** — Business logic, models, schemas
- **(Infrastructure)** — Database, external services
- **(API Layer)** — Endpoints, controllers
- **(Integration)** — Connecting components
- **(Testing)** — Test implementation
- **(Cleanup)** — Refactoring, documentation
