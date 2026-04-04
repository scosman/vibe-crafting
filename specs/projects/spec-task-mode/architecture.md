---
status: complete
---

# Architecture: Task Mode

## Overview

This project modifies markdown reference files in the spec skill (`/Users/scosman/.claude/skills/spec/`). There's no application code — the "architecture" is the file organization, prompt design, and how existing files change.

## File Changes

### New Files

#### `references/cmd_task.md`

The command reference for `/spec task`. Follows the same structure as `cmd_implement.md`. Sections:

1. **Invocation** — command syntax, aliases, handling bare `/spec task` with no description
2. **Clarify/Pushback Phase** — when to ask questions vs proceed, how to append notes to task file
3. **Task File Creation** — slug generation, file format, writing to `.specs_skill_state/tasks/`
4. **State Update** — writing `Current Task: tasks/[slug]` to `current_project.md`
5. **Implementation Flow** — identical to `cmd_implement.md`'s Single Phase Flow, but:
   - Uses the task coding prompt template instead of the project coding prompt
   - CR agent prompt references task file instead of spec artifacts
   - Commit prompt doesn't reference `implementation_plan.md` checkboxes
6. **Prompt Templates** — task-specific versions of Initial Coding, CR Feedback, Commit, and CR Agent prompts
7. **Escalation** — same as `cmd_implement.md`

The manager role is identical to project mode: orchestrate sub-agents, never code directly.

#### `references/task_coding_prompt.md`

A simplified version of `coding_phase_prompt.md` for task mode. Key differences from the project version:

- **Context Loading**: Reads the task file (`.specs_skill_state/tasks/[slug].md`) instead of spec artifacts. Both `## Request` and `## Notes` sections inform the work.
- **No Phase Plan**: No `phase_plans/phase_N.md` to write. The task file IS the plan.
- **No implementation_plan.md**: No checkboxes to mark complete.
- **Commit**: Commits with a descriptive message. Marks `status: complete` on the task file.
- **Shared sections**: Persona, quality bar, automated checks, CR feedback handling, non-interactive rule, and escalation are loaded from shared reference files (see Shared Prompt Sections below) — not duplicated.

Structure mirrors `coding_phase_prompt.md` with three invocation modes:
1. **Initial**: Read task file → implement → run checks → run tests → return summary
2. **CR Feedback**: Address feedback → run checks → run tests → return summary  
3. **Commit**: Commit all changes → mark task complete → return commit message

#### `references/shared/coding_role.md`

Extracted from the existing `coding_phase_prompt.md` "Your Role" section — the senior engineer persona, code quality expectations, and willingness to flag bad requirements.

#### `references/shared/coding_workflow.md`

Extracted from the existing `coding_phase_prompt.md` — the implementation steps that are identical across modes:
- Build → run automated checks → write tests → run tests → run checks again
- CR feedback invocation (read feedback, address issues, run checks/tests)
- Non-interactive rule and the one escalation exception
- Completion semantics (what "done" means per invocation mode)

Both `coding_phase_prompt.md` and `task_coding_prompt.md` load these shared files and add their mode-specific sections (context loading, phase plan vs task file, commit behavior).

### Modified Files

#### `SKILL.md`

Add to the Command Reference section, after `/spec implement`:

```markdown
### `/spec task`

Implement a one-off task without a full spec. Describe what you want inline and get the same
implement loop (coding agent, code review, commit) without planning artifacts.

→ Read [task command reference](references/cmd_task.md)
```

Add a section explaining the two modes:

```markdown
## Modes

The skill operates in two modes:

- **Project mode**: Full spec planning → phased implementation. Use for anything that benefits
  from upfront design — new features, complex changes, multi-phase work.
- **Task mode**: Inline description → single-pass implementation. Use for well-understood,
  small-to-medium changes — bug fixes, small features, refactors.

Both modes use the same implement loop (coding agent → code review → commit). Task mode
skips the spec planning steps.
```

Update the Router section to mention task state detection.

Update State Management to document the `Current Task:` format.

#### `references/cmd_continue.md`

Update the state-reading logic in "1. Read State" to handle both formats:

```
Current Project: /specs/projects/PROJECT_NAME
Current Task: tasks/[slug]
```

Update "3. Determine Current State" to add a task branch:

**If active work is a task:**
- Read the task file from `.specs_skill_state/tasks/[slug].md`
- If `status: active`: offer to resume implementation
- If `status: complete`: report done, suggest next action

Task resume enters the implementation flow directly — no clarification re-run.

#### `references/cr_agent_prompt.md`

Rename "Spec Compliance" section header to "Spec/Task Compliance". Update the content to handle both modes:

- **Project mode** (unchanged): Read spec artifacts, verify implementation matches spec
- **Task mode**: Read the task file (`.specs_skill_state/tasks/[slug].md`), verify implementation matches the request and any notes

The context loading section gains a conditional: if the prompt mentions a task file path, load that instead of spec artifacts.

#### `references/coding_phase_prompt.md`

Refactored to load shared sections from `references/shared/coding_role.md` and `references/shared/coding_workflow.md` instead of inlining them. The project-specific parts remain: context loading (spec artifacts), phase plan writing, and commit behavior (marking implementation_plan.md checkboxes). Behavior is identical — this is a pure extraction, no logic changes.

#### `references/cmd_implement.md`

Minimal change — add a note in the routing section:

> For one-off tasks without a full spec, use `/spec task` instead.

No structural changes needed since the implement command still only operates on projects.

## Prompt Template Design

The `cmd_task.md` prompt templates follow the same pattern as `cmd_implement.md` but reference task-specific files:

### Initial Coding Prompt (Task)

```
You are a coding agent implementing a task.

**Task file:** [.specs_skill_state/tasks/SLUG.md]

Read `references/task_coding_prompt.md` for your full instructions. Follow them precisely.

Return a short summary of what you built when implementation is complete and ready for code review.
```

### CR Feedback Prompt (Task)

Identical to project mode — the feedback format doesn't change.

### Commit Prompt (Task)

```
Your code has passed review. Commit all changes with a descriptive message summarizing the work done.
Mark the task status as complete in the task file.

Return the commit message you used.
```

### CR Agent Prompt (Task)

```
Review code changes for the task described in [.specs_skill_state/tasks/SLUG.md].

Read `references/cr_agent_prompt.md` for your full review instructions. Follow them precisely.
```

## Slug Generation

Simple rules for converting task description to filesystem slug:
- Lowercase
- Replace spaces and special characters with hyphens
- Collapse multiple hyphens
- Truncate to ~50 characters at a word boundary
- On collision, append `-2`, `-3`, etc.

## No Component Designs Needed

This project is small enough that everything fits in this architecture doc. All files are markdown with well-defined structures — no complex interactions requiring separate component docs.
