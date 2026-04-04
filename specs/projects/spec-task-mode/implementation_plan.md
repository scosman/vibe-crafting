---
status: complete
---

# Implementation Plan: Task Mode

## Phases

- [x] Phase 1: Extract shared coding prompt sections
  - Create `references/shared/coding_role.md` and `references/shared/coding_workflow.md` from existing `coding_phase_prompt.md`
  - Refactor `coding_phase_prompt.md` to load shared files instead of inlining
  - No behavior change — pure extraction

- [x] Phase 2: Add task mode files and update existing references
  - Create `references/cmd_task.md` (command reference)
  - Create `references/task_coding_prompt.md` (loads shared files, adds task-specific context/commit)
  - Update `references/cr_agent_prompt.md` (spec compliance → spec/task compliance)
  - Update `references/cmd_continue.md` (handle `Current Task:` state)
  - Update `references/cmd_implement.md` (add note about `/spec task`)
  - Update `SKILL.md` (add task command, modes section, router update, state format)
