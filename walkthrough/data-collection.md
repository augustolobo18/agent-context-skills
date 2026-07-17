# Data Collection Component

**Component of:** walkthrough
**Purpose:** Collect real implementation data to document in the walkthrough.

## Concept

A walkthrough must document REAL data, never invented. This component defines how to collect implementation information systematically.

## Implementation

### Phase 1: Questions to the User

Ask the user for context that cannot be inferred from the code:

```
I need some information to generate the walkthrough:

1. **Motivation:** What problem does this implementation solve?
   (e.g. "Bug where a direct search did not navigate", "Refactor to improve testability")

2. **Scope:** Which files were modified/created?
   (I can detect this via git if available, or list them manually)

3. **Tests:** Did the tests pass? Do you want me to run them and capture the output?

4. **Roadmap:** Is this implementation tied to a roadmap item?
   (e.g. "R.2.2", "10.1.1", or leave blank)

5. **Attention Points:** Is there anything future developers should know?
   (Limitations, accepted trade-offs, technical debt created/resolved)
```

### Phase 2: Automatic Collection (Git)

If the project uses git, collect automatically:

```bash
# Check whether this is a git repository
git rev-parse --is-inside-work-tree

# Modified files (uncommitted)
git status --porcelain

# Files in the last commit
git diff HEAD~1 --name-status

# Change statistics
git diff HEAD~1 --stat

# Recent commit messages (for context)
git log -5 --oneline

# Detailed diff (for analysis)
git diff HEAD~1
```

### Phase 3: Quantitative Metrics

Collect metrics whenever possible:

**Line count:**
```bash
# Lines in a specific file
wc -l path/to/file.py

# Lines before vs after (if available in the diff)
git diff HEAD~1 --numstat
```

**File count:**
- Created files: `A` in git status
- Modified files: `M` in git status
- Deleted files: `D` in git status

**Test results:**
```bash
# Python/pytest
pytest -v 2>&1 | tail -20

# JavaScript/vitest
npm test -- --reporter=verbose 2>&1 | tail -30

# JavaScript/jest
npm test -- --verbose 2>&1 | tail -30
```

### Phase 4: Code Analysis

Read the modified files to understand:

1. **What changed:**
   - New functions/classes
   - Signature changes
   - Logic changes

2. **Architectural impact:**
   - New dependencies
   - Folder structure changes
   - Public API changes

3. **Applied patterns:**
   - Design patterns used
   - Conventions followed
   - Refactors performed

## Collected Data Structure

```yaml
implementation:
  title: "Short description"
  motivation: "Problem solved"
  roadmap_item: "R.X.X" | null

files:
  created:
    - path: "path/file.ts"
      description: "Purpose of the file"
      lines: 150
  modified:
    - path: "path/other.py"
      change: "Description of the change"
      lines_before: 200
      lines_after: 180
  deleted:
    - path: "path/removed.js"
      reason: "Reason for removal"

tests:
  executed: true | false
  output: "Full test output"
  passed: 72
  failed: 0
  skipped: 2

metrics:
  total_files: 5
  lines_added: 320
  lines_removed: 45
  net_change: +275

context:
  attention_points:
    - "Description of point 1"
    - "Description of point 2"
  technical_debt:
    resolved: ["TD.07"]
    created: []
  references:
    - "Link or path to related doc"
```

## Best Practices

1. **Never invent data** - If you cannot collect it, mark it as "Not available"
2. **Prefer automatic data** - Git is more reliable than memory
3. **Validate with the user** - Confirm that collected data is correct
4. **Sanitize outputs** - Remove credentials, tokens, sensitive data

## Edge Cases

**Git not available:**
- Ask the user for the list of files
- Request metrics manually
- Note in the walkthrough that metrics are approximate

**Tests not run:**
- Document "Manual verification performed"
- List the verification steps
- Mark it as an attention point

**Partial implementation:**
- Mark status as "In Progress"
- Document what was completed
- List the pending next steps

**Very long test output:**
- Include a summary (passed/failed/skipped)
- Truncate the output with "..."
- Mention that the full output is available

---

## Massive Refactor (50+ files)

When the implementation involves many files (>20), group by category:

### Grouping Logic

```yaml
grouping_rules:
  by_directory:
    - src/components/     # "UI Components"
    - src/services/       # "Services"
    - src/utils/          # "Utilities"
    - tests/              # "Tests"

  by_type:
    - "*.test.ts"         # "Test files"
    - "*.stories.tsx"     # "Storybook"
    - "*.types.ts"        # "Type definitions"
```

### Grouped Output Format

```markdown
### 2.1 Modified Files (47 files)

**UI Components (12 files)**
| Directory | Files | Main Change |
|-----------|-------|-------------|
| src/components/Button/ | 3 | Props refactor |
| src/components/Modal/ | 4 | New state system |
| src/components/Form/ | 5 | Unified validation |

**Services (8 files)**
| Directory | Files | Main Change |
|-----------|-------|-------------|
| src/services/api/ | 5 | New cache layer |
| src/services/auth/ | 3 | Token refresh |

**Updated Tests: 27 files**
- Tests adapted to the new interfaces
- No change in test logic
```

---

## Sanitizing Sensitive Data

Before including any output in the walkthrough, sanitize sensitive data:

### Patterns to Detect

```yaml
sensitive_patterns:
  api_keys:
    - regex: "(api[_-]?key|apikey)['\"]?\\s*[:=]\\s*['\"]?[a-zA-Z0-9_-]{20,}"
    - replace: "[API_KEY_REDACTED]"

  tokens:
    - regex: "(bearer|token|jwt)['\"]?\\s*[:=]\\s*['\"]?[a-zA-Z0-9._-]{20,}"
    - replace: "[TOKEN_REDACTED]"

  passwords:
    - regex: "(password|passwd|pwd|secret)['\"]?\\s*[:=]\\s*['\"]?[^\\s'\"]{4,}"
    - replace: "[PASSWORD_REDACTED]"

  connection_strings:
    - regex: "(mongodb|postgres|mysql|redis)://[^\\s]+"
    - replace: "[CONNECTION_STRING_REDACTED]"

  aws_keys:
    - regex: "AKIA[0-9A-Z]{16}"
    - replace: "[AWS_KEY_REDACTED]"

  private_keys:
    - regex: "-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY-----"
    - replace: "[PRIVATE_KEY_REDACTED]"
```

### Sanitization Function

```python
def sanitize_output(text: str) -> str:
    """Remove sensitive data while keeping useful context."""
    for pattern_name, pattern_config in SENSITIVE_PATTERNS.items():
        text = re.sub(
            pattern_config['regex'],
            pattern_config['replace'],
            text,
            flags=re.IGNORECASE
        )
    return text
```

### Post-Sanitization Validation

Verify that the sanitized output:
- Contains no remaining sensitive patterns
- Still provides useful context
- Clearly indicates where data was removed

---

## Multiple Change Types

When a commit contains multiple change types (feat + fix + refactor):

### Automatic Detection

```yaml
commit_type_indicators:
  feat:
    - "new|add|adds|implements|creates"
    - new files under src/features/ or src/components/
  fix:
    - "fix|fixes|bug|error|issue"
    - small changes in existing files
  refactor:
    - "refactor|reorganize|move|rename"
    - many modified files with no behavior change
  docs:
    - "docs|readme|changelog"
    - modified .md files
  test:
    - "test|spec"
    - files under tests/ or *.test.*
```

### Prompt for the User

When multiple types are detected:

```
I detected multiple change types in this implementation:
- [feat] New UI components added
- [fix] Bug fix in the form
- [refactor] Import reorganization

What is the main focus? (this defines the commit type)

1. feat - New functionality (recommended if it includes a feature)
2. fix - Bug fix
3. refactor - Code improvement

Or I can generate a commit with the main type + a mention of the others in the body.
```

### Multi-type Commit Format

```
feat(ui): add form components with validation

- New components: Input, Select, Checkbox
- Fix bug in the submit handler
- Refactor imports to barrel exports

Includes: fix(form), refactor(imports)
Ref: R.2.3
```

---

## Technical Terms

Keep design pattern names and standard technical terms as-is:
- Design pattern names (Repository, Factory, etc.)
- Commit types (feat, fix, refactor)
- Framework terms (component, hook, service)
- Acronyms (API, UI, DB, etc.)
