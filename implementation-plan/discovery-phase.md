# Discovery Phase

**Component of:** implementation-plan
**Purpose:** Deep iterative discovery using AskUserQuestion before generating the plan

## Concept

Discovery happens **inline in the conversation**, with no mode transitions. The goal is to
reach zero ambiguity using silent codebase exploration + multiple-choice questions via
`AskUserQuestion`.

## Flow

```
User request
  │
  ├─ Step 1: Silent codebase exploration (Glob/Grep/Read)
  │    └─ Understand the stack, patterns, relevant files
  │
  ├─ Step 2: First round of questions (AskUserQuestion)
  │    └─ Clarify scope, constraints, preferences
  │
  ├─ Step N: Follow-up questions (if needed)
  │    └─ Refine based on the previous answers
  │
  └─ Full confidence → Plan Generation Phase
```

## Step 1: Understand the terrain

Before asking the user anything, explore silently:

```
# Identify the stack and structure
Glob: package.json, requirements.txt, Cargo.toml, go.mod, etc.
Read: The main configuration file

# Find code related to the request
Grep: Key terms from the user's request
Read: The most relevant files found

# Map dependencies and patterns
Glob: Test, config, and service directories
Read: Existing similar implementations
```

**Goal:** Have enough context to ask intelligent questions, not generic ones.

## Step 2: Iterative questioning via AskUserQuestion

### Tool capabilities

`AskUserQuestion` offers a rich multiple-choice interface:
- **1 to 4 questions per call** — group related questions into a single call
- **2 to 4 options per question** — plus an automatic "Other" for free-form answers
- **header** — short chip/tag (max 12 chars) such as "Scope", "Architecture", "Priority"
- **description** — explains each option's trade-offs
- **preview** — shows mockups, code, or diagrams side by side (single-select only)
- **multiSelect** — allows selecting multiple options when they are not mutually exclusive

### Questioning rules

1. **Ask about decisions, not facts about the code** — facts you read, decisions you ask about.
2. **Group related questions** — use up to 4 questions per `AskUserQuestion` call to reduce back-and-forth.
3. **Offer concrete options** — do not ask "how do you want it?"; offer options with pros and cons in the descriptions.
4. **Use previews when visual** — for layouts, directory structures, and schemas, use the `preview` field.
5. **Stop when you know enough** — do not ask for the sake of asking. If the answer does not change the plan, do not ask.
6. **Recommend when you have an opinion** — put the recommended option first with "(Recommended)" in the label.

### Example of a well-structured call

```json
{
  "questions": [
    {
      "question": "What should the authentication strategy be?",
      "header": "Auth",
      "multiSelect": false,
      "options": [
        {
          "label": "Stateless JWT (Recommended)",
          "description": "Self-contained tokens, no server-side session. Better for APIs and horizontal scaling."
        },
        {
          "label": "Session-based",
          "description": "Server-side session with a cookie. Simpler, but requires shared storage across multiple instances."
        }
      ]
    },
    {
      "question": "Does the scope include refresh tokens?",
      "header": "Scope",
      "multiSelect": false,
      "options": [
        {
          "label": "Yes, with rotation",
          "description": "Refresh token rotated on each use. More secure against token theft."
        },
        {
          "label": "No, access token only",
          "description": "Simpler implementation. The user re-authenticates when the token expires."
        }
      ]
    }
  ]
}
```

### Valid question categories

| Category | Example | When to ask |
|----------|---------|-------------|
| **Scope** | "Does feature X include scenario Y, or only Z?" | Whenever there is an ambiguous boundary |
| **Architecture** | "Materialized view or regular view?" | When there are real trade-offs |
| **Priority** | "If performance and simplicity conflict, which wins?" | When the trade-off is unavoidable |
| **Constraints** | "Is there a table/API that must not be changed?" | When the impact is destructive |
| **Preference** | "Name the service X or Y?" | When both options are valid |

### INVALID question categories

- "Can I use Glob to look for files?" → Do it, do not ask.
- "Does the project use React?" → Read package.json.
- "What is the directory structure?" → Explore with Glob.
- "Should I create tests?" → Yes, always. Do not ask.

## Step 3: Completion criteria

Before moving on to plan generation, validate mentally:

- [ ] I know exactly what the user wants (scope defined)
- [ ] I know which files will be impacted (I explored the codebase)
- [ ] All architectural decisions have been made (by the user, or by me based on the patterns)
- [ ] I have no assumptions — I have facts or answers

If any item fails, do another round of questions via `AskUserQuestion`.

## Transition to Generation

When discovery is complete, proceed directly to plan generation:

1. Start generating the plan in the format defined in `plan-structure.md` (sections 0-5)
2. Save to `plans/YYYY-MM-DD_Plan_Name.md`
3. Open with `Bash` (`start "" "plans/..."`) for the user to review
4. Request approval via `AskUserQuestion` (see `approval-workflow.md`)

**No mode transitions. No pauses. Continuous flow.**

## Edge Cases

### Trivial request (1 file, < 10 lines)
- Do just 1 quick round of exploration
- If everything is obvious, proceed without questions
- Generate a minimal plan (may use `--visual-level=minimal`)

### Very vague request ("improve the system")
- Explore the codebase broadly first
- Ask scoping questions before anything else:
  - "Improve along which dimension? Performance, UX, maintainability?"
  - "Is there a specific part causing problems?"
- Do not try to cover everything — help the user focus

### Impatient user ("just do it")
- Acknowledge the urgency
- Explain briefly: "I need 2 answers so the plan is not wrong"
- Ask only the most critical questions
- Document assumptions explicitly in the plan

### Large codebase (20+ impacted files)
- Group by module/feature during exploration
- Ask about priority between modules
- Consider suggesting a split into multiple plans
