# Approval Workflow

**Component of:** implementation-plan
**Purpose:** Guarantee explicit user approval via AskUserQuestion before any implementation

## Concept

The plan is a **CONTRACT** between the assistant and the user. No code change happens until
the user explicitly approves. Approval uses `AskUserQuestion` to present clear options in a
multiple-choice interface.

## Full Flow

```
Plan saved in plans/ → Bash `start ""` to open the file → AskUserQuestion (approval)
  │
  ├─ "Approved" → Invoke the plan-executor subagent (Sonnet) with the plan path
  ├─ "Revise"   → User gives feedback → Revise the plan → New AskUserQuestion
  └─ "Cancel"   → End with no changes
```

## The Approval Call

After saving and opening the plan with `Bash` (`start "" "plans/..."`), use EXACTLY this call:

```json
{
  "questions": [
    {
      "question": "Review the plan above. How would you like to proceed?",
      "header": "Approval",
      "multiSelect": false,
      "options": [
        {
          "label": "Approved",
          "description": "Start the implementation, following the plan exactly as presented."
        },
        {
          "label": "Revise",
          "description": "Request changes to the plan before implementing. Describe the adjustments you want."
        },
        {
          "label": "Cancel",
          "description": "Abandon this plan. No change will be made to the code."
        }
      ]
    }
  ]
}
```

**Note:** The user may also select "Other" (automatic in AskUserQuestion) to give free-form feedback.

## Handling Responses

### Response: "Approved"

Do NOT implement in the main agent. Instead, **delegate execution to the `plan-executor`
subagent** (which runs on Sonnet, in an isolated context, bootstrapping from the plan's
Section 0). This keeps the main session clean and execution cheap, without you having to
switch models manually.

Steps:

1. Confirm to the user:
   ```
   Approved. Delegating execution to the plan-executor subagent (Sonnet)...
   ```
2. Invoke the Agent tool with `subagent_type: "plan-executor"` and a prompt that passes the
   **exact path of the saved plan**, for example:
   ```
   Execute the approved implementation plan at `plans/YYYY-MM-DD_Plan_Name.md`.
   Perform the mandatory bootstrap by reading Section "0. Context to Load" before
   writing any code. Implement all phases and run the verifications.
   ```
3. When the subagent returns, **relay its summary leanly** — this is the operational
   handoff (files changed, verifications, open items), the only channel through which the
   result of the isolated execution reaches the conversation. Keep it short: do NOT turn it
   into technical documentation (diagrams, narrative, metrics) — that is `/walkthrough`'s
   job, which runs later, on demand. If the subagent flags an open item/decision, resolve it
   with the user and, if needed, invoke the subagent again to continue.

> Why delegate: execution is the long, expensive phase. Running it on an isolated Sonnet
> (instead of in the main agent on Opus) cuts cost without losing fidelity — the bootstrap
> guarantees the subagent has the project's conventions and rules.

### Response: "Revise"

Selecting "Revise" includes no text — it is only a signal of intent. You need to collect the
feedback:

1. Ask the user what they want to change (plain text in the chat, e.g. "What would you like to adjust in the plan?")
2. Wait for the response
3. Revise the relevant sections based on the feedback
4. Save the updated plan (overwrite the file in `plans/`)
5. Open it again with `Bash` (`start "" "plans/..."`)
6. Use `AskUserQuestion` again with the same approval options

### Response: "Cancel"

```
Understood. Plan cancelled. No changes were made.

If you like, I can:
- Start over with a different approach
- Discuss the requirements before planning
- Work on something else
```

### Response: "Other" (free-form feedback)

Treat it as "Revise" — the typed text is the feedback. Follow the same revision flow.

## Edge Cases

### User wants to skip planning ("just do it already")

```
I understand the urgency. But for a change of this scope, the plan avoids:
- Forgetting files that need to change
- Problems that only surface mid-implementation
- Rework caused by misalignment

It takes just a few seconds. May I proceed with the planning?
```

### User asks to implement during planning

```
I will finish the plan first, before any change to the code.
That way you can review the full approach before I start.
```

### Partial approval ("approve phase 1 only")

```
Understood. I will delegate only Phase 1 to the executor and pause for your review before Phase 2.
```

When invoking `plan-executor`, restrict the scope in the prompt (e.g. "Execute ONLY Phase 1
of the plan at `plans/...`; stop when it is done"). Track which phases have already run and
re-invoke the subagent for the following phases only after fresh approval.

### Requirements change after approval

1. Pause the current implementation
2. Revise the plan with the changes
3. Save, open with `Bash` (`start "" "plans/..."`), and request fresh approval via `AskUserQuestion`

## Post-Approval Behavior

The implementation happens **inside the `plan-executor` subagent**, not in the main agent.
The execution rules (follow the plan exactly, bootstrap, run verifications, stop and report
instead of guessing) are defined in `plan-executor.md` itself and are its responsibility.

The main agent's role after delegating is only:

1. **Wait for the return** of the subagent.
2. **Relay the lean summary** to the user (operational handoff, see step 3 above).
3. **Handle open items** — if the subagent stopped asking for a decision, resolve it with
   the user and re-invoke the subagent to continue from where it stopped.
4. **Do not re-execute** or re-document what the subagent has already done.
