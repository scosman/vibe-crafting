---
status: complete
---

# Task Mode for Spec Skill

## Problem

The `/spec implement` workflow (manager → coding agent → CR loop → commit) produces high-quality code. But it requires a full spec project — project overview, functional spec, architecture, implementation plan — before any code gets written. For small, well-understood changes ("fix the auth timeout bug", "add a loading spinner to the dashboard"), that overhead isn't worth it.

Users want to say `/spec task: fix the auth timeout bug` and get the same implement loop (code agent, CR, fix, commit) without creating a spec first.

## Solution

Add a new **task mode** to the spec skill. A task is a lightweight alternative to a project: the user describes what they want in plain text, and the skill runs the implement loop against that description directly.

### New Command: `/spec task`

**Invocation:** `/spec task: [description of what to do]`

The description is inline — no multi-step spec planning. The manager:

1. Writes the user's description **verbatim** to `.specs_skill_state/tasks/[slug].md`
2. Sets it as active in `.specs_skill_state/current_project.md` (using a `Current Task:` prefix to distinguish from projects)
3. **Clarify/pushback phase:** Before implementing, the manager considers whether the task needs clarification — ambiguous scope, missing details, potential pitfalls. If the task is clear and well-scoped, skip straight to implementation. If not, ask questions first. Clarifications and refined instructions are appended as notes below the original description (the original wording is never modified).
4. Runs the single-phase implement flow: spawn coding agent → CR loop → commit
5. Marks the task complete when done

### Task File Format

Stored at `.specs_skill_state/tasks/[slug].md`:

```markdown
---
status: active | complete
created: YYYY-MM-DD
---

# Task: [title]

## Request

[User's description — verbatim, never modified after initial write]

## Notes

[Clarifications, refined scope, or decisions from pushback phase. Added below the original request, never editing it. This section is omitted if there were no clarifications.]
```

The slug is auto-generated from the description (e.g., "fix auth timeout" → `fix-auth-timeout`).

**Important:** The `## Request` section preserves the user's exact words. This is the source of truth. If the manager's clarify/pushback phase surfaces refinements, those go in `## Notes` as additive context — never rewriting the original request. This prevents early misinterpretation from silently propagating through the coding and CR agents.

### State Integration

`current_project.md` gains a new format:

```
Current Task: tasks/fix-auth-timeout
```

This lets `/spec continue` detect whether the active work is a task or project and route accordingly. Tasks don't have spec artifacts, so continue just checks if the task is complete or still active.

### What Changes

- **New file:** `references/cmd_task.md` — task command reference
- **New file:** `references/task_coding_prompt.md` — coding agent prompt for task mode (simpler than phase prompt — no spec artifacts to read, just the task description)
- **Updated:** `SKILL.md` — add task mode to command reference and explain the two modes
- **Updated:** `references/cmd_continue.md` — handle `Current Task:` prefix, route to task resume
- **Updated:** `references/cmd_implement.md` — mention that task mode exists as an alternative (or at minimum, don't confuse task state with project state)
- **Updated:** `references/cr_agent_prompt.md` — update "spec compliance" section to be "spec/task compliance" so it works for both modes (review against task description in task mode, spec artifacts in project mode)
- **New file:** `references/task_coding_prompt.md` — separate, simpler coding prompt for task mode (no spec artifacts, no phase plan, no implementation_plan.md checkboxes — just the task description and notes)

### What Stays the Same

- The implement loop mechanics (manager, coding agent, CR agent, commit flow)
- Sub-agent spawning pattern
- The full project workflow — task mode is additive, not a replacement

## Scope

This is the spec skill only (files under `/Users/scosman/.claude/skills/spec/`). No changes to the host repo or other skills.
