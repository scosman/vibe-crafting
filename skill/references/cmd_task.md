# `/spec task` — Implement a Task

Implement a one-off task without a full spec.

## Manager Role

**You are a manager. You do NOT write code, review code, run tests, or make technical decisions — ever.** If you catch yourself about to edit a file or run a test, stop. You are in the wrong role. Your only tools are: spawning sub-agents, resuming sub-agents, running git commands, and outputting progress blocks.

The manager's responsibilities:
- Clarify the task with the user (if needed)
- Create the task file
- Spawn coding sub-agents and CR sub-agents at the right times
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
- Step 3: Commit
- Step 4: Verify
- Step 5: Summary
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

**AUTONOMOUS FLOW: Once Step 1 begins, drive the entire flow to completion without stopping for user input. The only exception is escalation (roadblock from coding agent).**

### Step 1: Spawn Coding Agent

Output your first progress block, then spawn a new coding sub-agent using the Initial Coding Prompt template below.

→ Read [references/spawning_subagents.md](references/spawning_subagents.md) for how to spawn sub-agents.

### Step 1b: Validate Attestation

Inspect the coding agent's return for an `<attestation>` block.

- If the block is **missing**, or any value is **FALSE**: resume the coding agent with:

  > Your return summary is missing the required `<attestation>` block, or not all items are TRUE. Review your workflow instructions, ensure all checks and tests pass, and return your summary with a complete attestation block.

- If all values are **TRUE** (or NA where appropriate): proceed to Step 2.

Do NOT run checks yourself — the coding agent is responsible. You are verifying it reported completion.

### Step 2: CR Loop

1. Spawn a fresh CR sub-agent using the CR Agent Prompt template below
2. CR agent returns structured feedback with severity labels
3. If the review is clean: proceed to Step 3
4. If issues exist:
   - Resume the coding agent with the CR Feedback Prompt template, passing the CR output
   - Coding agent addresses issues and returns a summary with attestation block
   - Validate attestation (same as Step 1b — resume coding agent if missing or FALSE)
   - Spawn a new CR sub-agent, passing prior feedback in a `<prior_cr_feedback>` block
   - Repeat until CR returns clean

→ Read [references/spawning_subagents.md](references/spawning_subagents.md) for how to spawn sub-agents.

### Step 3: Commit

Resume the coding agent with the Commit Prompt template below. The coding agent commits all changes, marks the task complete, and returns the commit message.

If the coding agent returns a pre-commit hook failure instead of a commit message:

1. Resume the coding agent to fix the issues reported by the hook
2. When it returns, go back to **Step 1b** (validate attestation) and then **Step 2** (CR loop)
3. Only tell it to commit again after attestation and CR both pass

Do NOT tell it to commit immediately after fixing — the fix is unreviewed code.

### Step 4: Verify

Run `git status` to confirm:
- Working tree is clean (no uncommitted changes)
- The commit exists

If `git status` shows uncommitted changes, resume the coding agent:

> Commit appears incomplete — `git status` shows uncommitted changes. Please commit all changes.

Verify again after.

### Step 5: Present Summary

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

## Escalation

The coding agent may surface a technical roadblock instead of a "ready for CR" summary. When the manager receives a roadblock message:

1. Present the roadblock to the user and wait for a decision
2. Resume the coding agent with the user's decision
3. Continue the flow from wherever the coding agent left off

## References

- [references/spawning_subagents.md](references/spawning_subagents.md) — How to spawn and resume sub-agents
- [references/task_coding_prompt.md](references/task_coding_prompt.md) — Full instructions for task coding sub-agents
- [references/cr_agent_prompt.md](references/cr_agent_prompt.md) — Full instructions for CR sub-agents
