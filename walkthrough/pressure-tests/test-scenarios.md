# Pressure Test Scenarios

Test scenarios to validate the walkthrough generation skill.

---

## Scenario 1: Simple Bugfix

### Setup
- User fixed a bug in 1 file
- ~10-line change
- Tests passing

### Expected Behavior
- Concise walkthrough (50-100 lines)
- Focus on: Problem, Diagnosis, Solution, Verification
- Minimal sections: Summary, Changes, Tests, Commit
- Commit type: `fix`

### Validation
- [ ] Walkthrough < 100 lines
- [ ] Problem clearly described
- [ ] Solution explained
- [ ] Commit type `fix`

### Status
PASSED

---

## Scenario 2: New Feature (Medium)

### Setup
- User implemented a new feature
- 5-10 files created/modified
- Includes unit tests
- Tied to a roadmap item

### Expected Behavior
- Standard walkthrough (100-300 lines)
- All mandatory sections + Architecture + References
- Diagram if there is a new flow
- Roadmap reference in the commit

### Validation
- [ ] Walkthrough 100-300 lines
- [ ] Architecture section present
- [ ] Diagram included
- [ ] Ref: [R.X.X] in the commit

### Status
PASSED

---

## Scenario 3: Massive Refactor

### Setup
- User refactored an entire module
- 50+ files modified
- Significant architectural change

### Expected Behavior
- Extensive walkthrough (300+ lines)
- Files grouped by category
- Summarized metrics ("32 test files updated")
- Multiple before/after diagrams
- Architecture comparison table

### Validation
- [ ] Files grouped (not listed individually)
- [ ] Summarized metrics
- [ ] Before/after diagram
- [ ] Comparison table present

### Status
PASSED

### Implementation
- Grouping logic documented in data-collection.md
- Template for summarized metrics included
- Grouping by directory and by file type

---

## Scenario 4: No Git Available

### Setup
- Project does not use git
- User describes changes manually

### Expected Behavior
- Ask the user for the list of files
- Request approximate metrics
- Generate the walkthrough based on manual input
- Note: "Approximate metrics (no git)"

### Validation
- [ ] Questions asked to the user
- [ ] Walkthrough generated with manual data
- [ ] Note about approximation included

### Status
PASSED

---

## Scenario 5: Tests Not Run

### Setup
- Implementation completed
- User did not run the tests
- Verification was manual

### Expected Behavior
- "How to Test" section instead of "Test Results"
- List the manual verification steps
- Attention point: "Automated tests not run"

### Validation
- [ ] "How to Test" section present
- [ ] Verification steps listed
- [ ] Attention point recorded

### Status
PASSED

---

## Scenario 6: Sensitive Data in the Output

### Setup
- Test output contains credentials
- Logs show API tokens

### Expected Behavior
- Detect sensitive patterns (API_KEY, token, password)
- Replace with [REDACTED]
- Keep useful context from the output

### Validation
- [ ] Credentials do not appear in the walkthrough
- [ ] [REDACTED] used appropriately
- [ ] Output still useful for understanding the result

### Status
PASSED

### Implementation
- Regex patterns documented in data-collection.md
- Sanitization function defined with patterns for:
  - API keys, tokens, passwords
  - Connection strings, AWS keys, private keys
- Post-sanitization validation included

---

## Scenario 7: No Reference Walkthrough

### Setup
- Walkthroughs folder empty or nonexistent
- First walkthrough of the project

### Expected Behavior
- Use the default template from report-template.md
- Inform the user that you are establishing the pattern
- Generate a complete walkthrough anyway

### Validation
- [ ] Default template used
- [ ] Informative message to the user
- [ ] Valid walkthrough generated

### Status
PASSED

---

## Scenario 8: Partial Implementation

### Setup
- User wants to document work in progress
- Implementation ~60% complete

### Expected Behavior
- Status: "In Progress"
- Document what was completed
- "Next Steps" section with pending items
- Do not generate a final commit suggestion

### Validation
- [ ] Status marked as "In Progress"
- [ ] Next steps section
- [ ] Commit marked as WIP or omitted

### Status
PASSED

---

## Scenario 9: Multiple Change Types

### Setup
- Commit contains: bugfix + refactor + new feature
- Hard to classify the commit type

### Expected Behavior
- Ask the user for the main focus
- Or use the most encompassing type (feat if it includes a feature)
- Document all aspects in the commit body

### Validation
- [ ] User asked about the focus
- [ ] Appropriate commit type selected
- [ ] Commit body lists all changes

### Status
PASSED

### Implementation
- Multiple-type detection logic in data-collection.md
- Indicators per commit type defined
- Prompt for the user to choose the main focus
- Multi-type commit format documented

---

## Scenario 10: Error During Test Execution

### Setup
- User asks to run the tests
- Tests fail during execution

### Expected Behavior
- Include the failure output in the walkthrough
- Mark the tests as failing
- Suggest the implementation may not be complete
- Status: "In Progress" or "Blocked"

### Validation
- [ ] Failure output included
- [ ] Appropriate status
- [ ] Suggested action

### Status
PASSED

---

## Scenario 11: Project with a Different Walkthrough Pattern

### Setup
- Existing walkthroughs use a format different from the template
- e.g. no numbering, different section names

### Expected Behavior
- Adapt to the project's existing pattern
- Do not force the default template
- Keep consistency with previous walkthroughs

### Validation
- [ ] Format adapted to the project
- [ ] Consistent with existing walkthroughs
- [ ] Logical structure maintained

### Status
PASSED

---

## Summary

| Scenario | Status | Notes |
|----------|--------|-------|
| 1. Simple Bugfix | PASSED | - |
| 2. New Feature | PASSED | - |
| 3. Massive Refactor | PASSED | Grouping logic in data-collection.md |
| 4. No Git | PASSED | - |
| 5. No Tests | PASSED | - |
| 6. Sensitive Data | PASSED | Sanitization patterns in data-collection.md |
| 7. No References | PASSED | - |
| 8. Partial | PASSED | - |
| 9. Multiple Types | PASSED | User prompt logic in data-collection.md |
| 10. Tests Fail | PASSED | - |
| 11. Different Pattern | PASSED | - |

**Pass Rate:** 11/11 (100%)
**All scenarios implemented and validated.**
