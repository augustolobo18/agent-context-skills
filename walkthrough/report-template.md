# Report Template Component

**Component of:** walkthrough
**Purpose:** Define the structure and template of the walkthrough to be generated.

## Base Template

```markdown
# Technical Walkthrough - [Descriptive Title]

**Date:** YYYY-MM-DD
**Status:** Completed
**Roadmap Item:** [R.X.X] (if applicable)
**Author:** Claude Code

---

## 1. Summary

[Executive paragraph of 2-4 sentences describing what was implemented and why.
It should answer: What? Why? What is the impact?]

---

## 2. Changes Made

### 2.1 Created Files

| File | Description |
|------|-------------|
| `path/to/new_file.ext` | Purpose of the file |

### 2.2 Modified Files

| File | Change |
|------|--------|
| `path/to/modified.ext` | Description of the change |

**Metrics:**
- Lines added: X
- Lines removed: Y
- Net change: +/-Z lines

### 2.3 Deleted Files

| File | Reason |
|------|--------|
| `path/to/deleted.ext` | Reason for removal |

---

## 3. [Contextual Section - Architecture/Solution/Approach]

[Include when there are significant architectural changes]

### Diagram/Flow

[ASCII diagram, mermaid, or comparison table]

### Technical Description

[Explanation of the technical approach taken]

---

## 4. Test Results

```
[Real output from the test run]
```

**Summary:**
- Total tests: X
- Passing: Y
- Failing: Z (if any, explain)

---

## 5. [Contextual Section - Attention Points]

[Include when there are limitations or important considerations]

### Known Limitations

1. **[Limitation 1]:** Description and reason
2. **[Limitation 2]:** Description and reason

### Maintenance Considerations

- [Important point for future developers]

---

## 6. [Contextual Section - Technical Debt]

[Include when resolving or creating technical debt]

**Resolved:**
| ID | Description | Status |
|----|-------------|--------|
| TD.XX | Description of the debt | RESOLVED |

**Created:**
[List if any debt was created as a trade-off]

---

## 7. [Contextual Section - How to Test]

[Include when manual verification is needed]

### Verification Steps

1. [Step 1]
2. [Step 2]
3. [Step 3]

### Commands

```bash
# Command to verify
command_here
```

---

## 8. [Contextual Section - References]

[Include when there are related documents]

- [Document Name](relative/path.md)
- [External Link](https://example.com)

---

## N. Commit Suggestion

**Type:** feat | fix | refactor | docs | test | chore

```
type(scope): short imperative description

- Detail of change 1
- Detail of change 2
- Detail of change 3

Resolves: [TD.XX or Issue] (if applicable)
Ref: [R.X.X] (if applicable)
```

---

*End of Walkthrough*
```

## Mandatory vs Contextual Sections

### Always Include
1. **Header** - Title, Date, Status
2. **Summary** - Executive overview
3. **Changes Made** - File list
4. **Commit Suggestion** - Always at the end

### Include When Relevant

| Section | When to Include |
|---------|-----------------|
| Architecture/Solution | Structural changes, new patterns |
| Test Results | Tests were run |
| Attention Points | There are limitations or considerations |
| Technical Debt | Resolved or created debt |
| How to Test | Manual verification needed |
| References | There are related docs |

## Formatting Rules

### Headings
- H1 (`#`) only for the main title
- H2 (`##`) for main sections
- H3 (`###`) for subsections

### Tables
```markdown
| Column 1 | Column 2 |
|----------|----------|
| value    | value    |
```

### Code Blocks
```markdown
```python
# Always specify the language
code_here()
```
```

### Lists
- Bullets (`-`) for unordered items
- Numbers (`1.`) for sequences

### File Paths
- Use paths relative to the project
- Format: `folder/subfolder/file.ext`
- Wrap in backticks

### Commits
- Follow conventional commits
- Types: feat, fix, refactor, docs, test, chore
- Scope optional but recommended
- Body with a list of changes

## Level of Detail by Type

### Bugfix (50-100 lines)
- Summary focused on problem/solution
- Simple file table
- Tests if available
- Commit

### Medium Feature (100-300 lines)
- Full summary
- Architecture if relevant
- Detailed tests
- Attention points
- Commit

### Complex Implementation (300+ lines)
- All sections
- Multiple diagrams
- Detailed metrics
- Cross-references
- Detailed commit

## Best Practices

1. **Self-contained** - The reader should understand without seeing the code
2. **Specific** - Avoid vague language
3. **Quantitative** - Include metrics when possible
4. **Technical** - Use correct terminology
5. **Consistent** - Follow the project's pattern
