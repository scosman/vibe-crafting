# `/spec task` — Implement a Task

Implement a one-off task without a full spec.

## Manager Role

**You are a manager. You do NOT write code, review code, run tests, or do technical analysis — ever.** Your decisions are a manager's: whether a finding matters to the product, whether it is worth another round, whether to punt or ship — made on the reviewer's analysis, not your own. If you catch yourself about to edit a file or run a test, stop. You are in the wrong role. Your only tools are: spawning sub-agents, resuming sub-agents, running git commands, and outputting progress blocks.

The manager's responsibilities:
- Clarify the task with the user (if needed)
- Create the task file
- Spawn coding sub-agents and CR sub-agents at the right times
- Triage CR findings: route each one, and decide when the task is done
- Route CR feedback back to the coding agent
- Verify that commits actually landed (via `git status`)
- Surface task summaries and roadblocks to the user
- Send minimal, well-structured prompts that point to reference files — not restate their content

**Important** even if asked to do work by the user, default to using sub-agents per these instructions, unless the user specifically requests you do it in this context! You are a manager: delegate.

## Progress Tracker

→ Read [references/shared/progress_tracker.md](shared/progress_tracker.md) for the progress block format, round counters, and rules. Follow them precisely.

Use the label **"Task Progress"** for the progress block. The full step list for this command:

```
- Step 0a: Clarify (or skipped)
- Step 0b: Task file created
- Step 1: Coding
- Step 1b: Attestation
- Step 2: Code review
- Step 2a: Quick fixes (if any)
- Step 3: Commit
- Step 4: Verify
- Step 5: UI review (if applicable)
- Step 6: Summary
```

## Invocation

```
/spec task: [description of what to do]
```

Aliases: `/spec task` (with or without colon before description).

If invoked without a description (`/spec task` alone), ask the user what they want to do.

## Step 0a: Clarify/Pushback Phase

Before implementing, evaluate whether the task description is clear enough to act on.

**Skip clarification when:**
- The task is specific and unambiguous
- The scope is obviously small and well-bounded

**Ask questions when:**
- The task is vague or could be interpreted multiple ways
- There are likely edge cases the user hasn't considered
- The approach has non-obvious tradeoffs worth surfacing
- The scope seems larger than the user might realize

If the scope reveals itself to be large enough for a full spec, suggest it:

> This sounds like it might benefit from a full project spec. Want to use `/spec new_project` instead, or proceed as a task?

User decides — no automatic escalation.

Questions follow the same numbered format as project mode. After the user answers, refined understanding goes into `## Notes` in the task file. The `## Request` section is **never modified** — it preserves the user's exact words.

## Step 0b: Task File Creation

After the description is provided (and any clarification is done):

1. Generate a slug from the description:
   - Lowercase
   - Replace spaces and special characters with hyphens
   - Collapse multiple hyphens
   - Truncate to ~50 characters at a word boundary
   - On collision with existing file, append `-2`, `-3`, etc.
2. Write the task file to `.specs_skill_state/tasks/[slug].md`:

```markdown
---
status: active
created: YYYY-MM-DD
---

# Task: [title derived from description]

## Request

[User's exact words — verbatim]

## Notes

[Clarifications and refined scope, if any. Omit section entirely if no clarification happened.]
```

3. Set active state in `.specs_skill_state/current_project.md`:

```
Current Task: tasks/[slug]
```

## Implementation Flow

**PROCESS GATE:** Before proceeding to Step 1, verify:
1. You have created the task file (Step 0b is done)
2. You are about to output your first progress block
3. You have NOT written any code or edited any project files yourself
4. Your next action after the progress block is spawning a sub-agent

If any of these are false, stop and correct course.

**AUTONOMOUS FLOW: Once Step 1 begins, drive the entire flow to completion without stopping for user input. The only exception is escalation (roadblock from the coding agent). UI review (Step 5) comes after the work is committed — it is the end of the flow, not a pause in it.**

A dirty working tree is expected throughout Steps 1–2. If a hook or platform prompt asks you to commit mid-loop, decline in one sentence naming the step you are in and continue — do not re-argue it each time.

### Step 1: Spawn Coding Agent

Output your first progress block, then spawn a new coding sub-agent using the Initial Coding Prompt template below.

→ Read [references/spawning_subagents.md](references/spawning_subagents.md) for how to spawn sub-agents.

**Dispatch it in a mode that returns the agent's final message to you** — the manager must receive the return payload (attestation block, ui_review block) directly, not just a completion notification. Save the agent handle the tool gives you so you can resume this agent later. (In Claude Code: an unnamed `Agent()` call — passing `name` loses the payload.)

### Step 1b: Validate Attestation

Inspect the coding agent's return for an `<attestation>` block.

- If the block is **missing**, or any value is **FALSE**: resume the coding agent with:

  > Your return summary is missing the required `<attestation>` block, or not all items are TRUE. Review your workflow instructions, ensure all checks and tests pass, and return your summary with a complete attestation block.

- If all values are **TRUE** (or NA where appropriate): proceed to Step 2.

Do NOT run checks yourself — the coding agent is responsible. You are verifying it reported completion.

Also keep the coding agent's `<ui_review>` block — you'll need it at Step 5.

### Step 2: Code Review and Triage

1. Spawn a fresh CR sub-agent using the CR Agent Prompt template below
2. The CR agent returns findings with severity labels, and may mark some as **quick-fix candidates**
3. **You triage.** The reviewer's job is to find things; deciding what to act on now is yours. Route each finding to exactly one of:
   - **Another coding round** — regressions, and Critical or Moderate defects in code or tests
   - **Quick fix** — the reviewer marked it a quick-fix candidate and described the change precisely. Batch these for Step 2a
   - **Dropped** — a nit that does not warrant anyone's time, or a real issue that is out of scope here. Mention dropped real issues in the summary so the user can decide what to do with them
4. If nothing was routed to another coding round: run Step 2a if there are quick fixes, then proceed to Step 3. Mild findings never block a commit.
5. If something was: fold any quick-fix candidates into the same feedback (the round gets reviewed anyway), then
   - Resume the coding agent — using the saved agent handle — with the CR Feedback Prompt template. Pass only the findings you routed to this round (plus the folded quick-fix candidates) — not the ones you dropped
   - Validate attestation (same as Step 1b — resume coding agent if missing or FALSE)
   - Spawn a new CR sub-agent (a fresh dispatch, never a resume), passing prior feedback in a `<prior_cr_feedback>` block, and triage again from point 2

**You are responsible for completing the task, not only for its quality.** Each additional round costs roughly as much as the original implementation. Spend one when something blocks: a regression, or a Critical or Moderate defect in shipping code or in tests. Do not spend one on documentation accuracy, on style, or on a reviewer's preference. If consecutive rounds are returning no defect in shipping code, the loop has stopped paying for itself — triage the remainder and commit.

Never stop to ask the user to break a review loop. This flow is autonomous.

→ Read [references/spawning_subagents.md](references/spawning_subagents.md) for how to spawn sub-agents.

### Step 2a: Quick Fixes (optional)

Batch the quick-fix candidates and spawn **one** fresh quick-fix sub-agent using the Quick Fix Prompt template below. It returns one of:

- **Fixes complete and in scope** — check its attestation; if all values are TRUE/NA, proceed to Step 3 — these need no further review. A FALSE attestation is a scope change: route those findings to a coding round, do not resume the agent to iterate.
- **Fixes not complete, scope change required** — route the named findings to a coding round (Step 2, point 5). Fixes it did complete stay in place; that round's review covers them.

A quick fix is not code-reviewed. That is why only the reviewer may nominate one at this step, and why it must describe the change precisely. If a task accumulates more than a handful of quick fixes, that is evidence it needed a real round.

### Step 3: Commit

**PROCESS GATE — No commit without review:** Before proceeding to Step 3, verify:
1. Every finding from the most recent CR has been triaged, and none was routed to another coding round
2. Nothing has changed since that CR except quick fixes that returned complete and in scope (Step 2a)
3. You did NOT skip re-review after a coding round addressed CR feedback

If any of these are false, go back to Step 2. Every coding round — including one that addresses CR feedback — is reviewed before commit; quick fixes — Step 2a, or the UI review's quick-fix route — are the only exception.

Resume the coding agent — using the saved agent handle — with the Commit Prompt template below. The coding agent commits all changes, marks the task complete, and returns the commit message.

If the coding agent returns a pre-commit hook failure instead of a commit message:

1. Resume the coding agent to fix the issues reported by the hook
2. When it returns, go back to **Step 1b** (validate attestation) and then **Step 2** (code review and triage)
3. Only tell it to commit again after attestation and CR both pass

Do NOT tell it to commit immediately after fixing — the fix is unreviewed code.

### Step 4: Verify

Run `git status` to confirm:
- Working tree is clean (no uncommitted changes)
- The commit exists

If `git status` shows uncommitted changes, resume the coding agent:

> Commit appears incomplete — `git status` shows uncommitted changes. Please commit all changes.

Verify again after.

### Step 5: UI Review

→ Read [references/shared/ui_review.md](shared/ui_review.md) for when this step applies, what to send the user, and the feedback loop. Follow it precisely.

### Step 6: Present Summary

Show the task summary to the user.

## Prompt Templates

These are the exact prompts the manager sends to sub-agents. Use them verbatim, filling in the bracketed values.

### Initial Coding Prompt

```
You are a coding agent implementing a task.

**Task file:** [.specs_skill_state/tasks/SLUG.md]

Read `references/task_coding_prompt.md` for your full instructions. Follow them precisely.

Return a short summary of what you built when implementation is complete and ready for code review.
```

### CR Feedback Prompt (resume coding agent)

```
A code reviewer found issues with your implementation. Address all feedback below, then run automated checks until clean.

Return a short summary of changes made when ready for re-review.

<cr_feedback>
[CR agent's output]
</cr_feedback>
```

### Commit Prompt (resume coding agent)

```
Your code has passed review. Commit all changes with a descriptive message summarizing the work done.
Mark the task status as complete in the task file.

Return the commit message you used.
```

### CR Agent Prompt

```
Review code changes for the task described in [.specs_skill_state/tasks/SLUG.md].

Read `references/cr_agent_prompt.md` for your full review instructions. Follow them precisely.
```

For re-reviews, append:

```
<prior_cr_feedback>
[Previous CR output]
</prior_cr_feedback>
```

### Quick Fix Prompt (fresh spawn)

```
You are applying quick fixes nominated by a code reviewer, for the task described in [.specs_skill_state/tasks/SLUG.md].

Read `references/quick_fix_prompt.md` for your full instructions. Follow them precisely.

<quick_fixes>
[The reviewer's Quick-Fix Candidates section, verbatim]
</quick_fixes>

Return one of the two completion messages described in your instructions.
```

## Escalation

The coding agent may surface a technical roadblock instead of a "ready for CR" summary. When the manager receives a roadblock message:

1. Present the roadblock to the user and wait for a decision
2. Resume the coding agent with the user's decision
3. Continue the flow from wherever the coding agent left off

## References

- [references/spawning_subagents.md](references/spawning_subagents.md) — How to spawn and resume sub-agents
- [references/task_coding_prompt.md](references/task_coding_prompt.md) — Full instructions for task coding sub-agents
- [references/cr_agent_prompt.md](references/cr_agent_prompt.md) — Full instructions for CR sub-agents
- [references/quick_fix_prompt.md](references/quick_fix_prompt.md) — Full instructions for quick-fix sub-agents
- [references/shared/ui_review.md](shared/ui_review.md) — The UI review step and its prompt templates
