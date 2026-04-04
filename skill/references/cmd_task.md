# `/spec task` — Implement a Task

Implement a one-off task without a full spec. The top-level agent acts as a strict manager/coordinator — it orchestrates sub-agents but never writes code or reviews it.

## Invocation

```
/spec task: [description of what to do]
```

Aliases: `/spec task` (with or without colon before description).

If invoked without a description (`/spec task` alone), ask the user what they want to do.

## Clarify/Pushback Phase

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

## Task File Creation

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

Once the clarify/pushback phase is complete and the task file is written, **run the entire implementation flow without stopping for user input.** The manager drives coding agent → CR loop → commit to completion autonomously. The only exception is escalation (see below).

Same as project mode's Single Phase Flow, adapted for tasks.

### Step 1: Spawn Coding Agent

Spawn a new coding sub-agent using the Initial Coding Prompt template below.

→ Read [references/spawning_subagents.md](references/spawning_subagents.md) for how to spawn sub-agents.

### Step 2: CR Loop

1. Spawn a fresh CR sub-agent using the CR Agent Prompt template below
2. CR agent returns structured feedback with severity labels
3. If the review is clean: proceed to Step 3
4. If issues exist:
   - Resume the coding agent with the CR Feedback Prompt template, passing the CR output
   - Coding agent addresses issues and returns a summary
   - Spawn a new CR sub-agent, passing prior feedback in a `<prior_cr_feedback>` block
   - Repeat until CR returns clean

→ Read [references/spawning_subagents.md](references/spawning_subagents.md) for how to spawn sub-agents.

### Step 3: Commit

Resume the coding agent with the Commit Prompt template below. The coding agent commits all changes, marks the task complete, and returns the commit message.

### Step 4: Verify

Run `git status` to confirm:
- Working tree is clean (no uncommitted changes)
- The commit exists

If `git status` shows uncommitted changes, resume the coding agent:

> Commit appears incomplete — `git status` shows uncommitted changes. Please commit all changes.

Verify again after.

### Step 5: Present Summary

Show the task summary to the user.

## Manager Role

The manager orchestrates the implementation process. It does NOT code, review code, run tests, or make technical decisions.

The manager's responsibilities:
- Spawn coding sub-agents and CR sub-agents at the right times
- Route CR feedback back to the coding agent
- Verify that commits actually landed (via `git status`)
- Surface task summaries and roadblocks to the user
- Send minimal, well-structured prompts that point to reference files — not restate their content

**Important** even if asked to do work by the user, default to using sub-agents per these instructions, unless the user specifically requests you do it in this context! You are a manager: delegate.

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
