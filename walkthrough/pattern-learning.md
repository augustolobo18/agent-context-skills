# Pattern Learning Component

**Component of:** walkthrough
**Purpose:** Analyze existing walkthroughs to learn the project's pattern before generating new documents.

## Concept

The skill must ALWAYS analyze existing walkthroughs before generating new ones. This ensures consistency with the pattern established by the project and avoids style divergences.

## Implementation

### Step 1: Discover Existing Walkthroughs

```bash
# Detect the project's walkthrough location (in priority order)
# 1. Check CLAUDE.md for a specific configuration
# 2. Search common locations:
Glob("**/walkthroughs/*.md")
Glob("**/context/walkthroughs/*.md")
Glob("**/docs/walkthroughs/*.md")
Glob("**/*walkthrough*.md")  # Fallback: any file with "walkthrough" in the name
```

### Step 2: Select Samples

Read at least 3 walkthroughs, prioritizing:
1. The most recent ones (by date in the filename)
2. A variety of types (refactoring, bugfix, feature)
3. Different sizes (concise vs detailed)

### Step 3: Extract Patterns

When reading each walkthrough, identify:

**Header Structure:**
- Title format (with or without "Technical Walkthrough")
- Header fields (Date, Status, Roadmap Item, Author)
- Use of separators (---)

**Sections Found:**
- Which sections appear consistently (mandatory)
- Which sections appear occasionally (contextual)
- Section order
- Numbering (1, 2, 3... or 1.1, 1.2...)

**Formatting:**
- Table style (pipes, alignment)
- Code block style (language specified)
- Use of lists (bullet vs numbered)
- Use of emojis (limited to status)

**Writing Conventions:**
- Active vs passive voice
- Level of technical detail
- Technical terms preserved as-is

## Common Patterns (Default Template)

If no existing walkthrough is found, use these patterns as a base:

### Default Header
```markdown
# Technical Walkthrough - [Descriptive Title]

**Date:** YYYY-MM-DD
**Status:** Completed | In Progress
**Roadmap Item:** [R.X.X] (when applicable)
**Author:** Claude Code (optional)

---
```

### Mandatory Sections
1. Summary (or "Implementation Summary")
2. Changes Made (with Created/Modified/Deleted subsections)
3. Commit Suggestion

### Common Contextual Sections
- Architecture (with ASCII or mermaid diagrams)
- Test Results
- Attention Points / Known Limitations
- Technical Debt Resolved
- How to Test / Verification
- References
- Architectural Decision (for design changes)

### Table Style
```markdown
| Column 1 | Column 2 |
|----------|----------|
| value    | value    |
```

### Code Block Style
```markdown
```python
# Specify the language
code_here()
```
```

### Conventions
- Paths relative to the project (no C:\Users\...)
- Prefix each file with its path (e.g. `backend/services/file.py`)
- Metrics in tabular format when there are several
- Commits following conventional commits

## Best Practices

1. **Always analyze before generating** - Never skip this step
2. **Adapt to the context** - If the project has a specific style, follow it
3. **Prioritize consistency** - Better to follow the existing pattern than to "improve" it
4. **Document divergences** - If the pattern does not cover a case, document the decision

## Edge Cases

**No existing walkthrough:**
- Use the default template from report-template.md
- Inform the user that you are establishing a new pattern

**Walkthroughs inconsistent with each other:**
- Prioritize the most recent
- Identify the majority pattern
- Ask the user if necessary

**Walkthrough in a different style:**
- Adapt to the project's existing structure
- Keep technical terms as-is
