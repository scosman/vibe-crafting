---
status: complete
---

# Functional Spec: Task Mode

## Overview

Task mode adds a lightweight path through the spec skill's implement loop. Users describe what they want inline, and the manager runs the same coding agent → CR → commit flow without requiring spec artifacts.

Two modes now exist:
- **Project mode** (existing): Full spec planning → phased implementation
- **Task mode** (new): Inline description → single-pass implementation

## Command: `/spec task`

### Invocation

```
/spec task: [description of what to do]
```

Aliases: `/spec task` (with colon or without).

If invoked without a description (`/spec task` alone), ask the user what they want to do.

### Clarify/Pushback Phase

Before implementing, the manager evaluates whether the task description is clear enough to act on.

**Skip clarification when:**
- The task is specific and unambiguous ("fix the off-by-one error in `parse_date` that crashes on leap years")
- The scope is obviously small and well-bounded

**Ask questions when:**
- The task is vague or could be interpreted multiple ways
- There are likely edge cases the user hasn't considered
- The approach has non-obvious tradeoffs worth surfacing
- The scope seems larger than the user might realize

Questions follow the same numbered format as project mode. After the user answers, refined understanding goes into `## Notes` in the task file — the `## Request` section is never modified.

### Task File Creation

After the description is provided (and any clarification is done), the manager:

1. Generates a slug from the description (e.g., "fix auth timeout" → `fix-auth-timeout`)
2. Writes the task file to `.specs_skill_state/tasks/[slug].md`
3. Sets active state in `.specs_skill_state/current_project.md`

### Implementation Flow

Same as project mode's single-phase flow:

1. Spawn coding agent with task-specific prompt (reads task file, not spec artifacts)
2. CR loop: spawn CR agent → if issues, resume coding agent → repeat
3. Commit: resume coding agent to commit
4. Verify: `git status` check
5. Mark task `status: complete`

### Escalation

Same as project mode — coding agent can surface roadblocks, manager presents to user, resumes with decision.

## Task File Format

Path: `.specs_skill_state/tasks/[slug].md`

```markdown
---
status: active | complete
created: YYYY-MM-DD
---

# Task: [title derived from description]

## Request

[User's exact words — verbatim, never modified]

## Notes

[Clarifications, scope decisions, and refined understanding from pushback phase.
Only present if clarification happened. Additive only — never rewrites the request.]
```

## State Management

### current_project.md Format

Task mode uses a different prefix:

```
Current Task: tasks/[slug]
```

vs project mode's existing:

```
Current Project: /specs/projects/[name]
```

The path in `Current Task` is relative to `.specs_skill_state/`.

### `/spec continue` Integration

When `current_project.md` contains a `Current Task:` line:

- If task status is `active`: offer to resume implementation (re-enter the implement flow)
- If task status is `complete`: report done, suggest next action

Continue does NOT re-run clarification — that only happens during initial `/spec task` invocation.

### `/spec` Router Integration

The bare `/spec` command should detect task state and present it alongside project state:

- If active task exists: show task name and status, offer to continue
- If no active task or project: suggest `/spec task` as an option alongside `/spec new_project`

## CR Agent in Task Mode

The CR agent reviews against:

1. **Task compliance** — does the implementation match what the task description asked for? (Same role as "spec compliance" in project mode, but reading the task file instead of spec artifacts)
2. **Code quality** — same as project mode
3. **Consistency** — same as project mode
4. **Project-specific standards** — same as project mode

The CR agent prompt's "Spec Compliance" section becomes "Spec/Task Compliance" and handles both modes.

## Edge Cases

### Name Collisions

If a task slug already exists in `.specs_skill_state/tasks/`, append a numeric suffix: `fix-auth-timeout-2`.

### Switching Between Modes

Starting a new task while a project is active (or vice versa) replaces the active pointer in `current_project.md`. The previous project/task files remain on disk — nothing is deleted.

### Task Scope Creep

If during clarification the task reveals itself to be large enough for a full spec, the manager should suggest:

> This sounds like it might benefit from a full project spec. Want to use `/spec new_project` instead, or proceed as a task?

User decides. No automatic escalation.

## Out of Scope

- Task history/listing (no `/spec tasks` command to list past tasks)
- Task dependencies or ordering
- Multi-phase tasks (a task is always a single implementation pass)
- Converting a task into a project after the fact
