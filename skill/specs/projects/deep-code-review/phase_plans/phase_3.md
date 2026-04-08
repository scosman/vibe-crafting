---
status: complete
---

# Phase 3: Create Manager Command Reference and Update SKILL.md

## What This Phase Builds

1. `references/cmd_deep_code_review.md` -- the manager command reference for `/spec deep cr`
2. Updated `SKILL.md` with the new command entry

## Plan

### 1. Create `references/cmd_deep_code_review.md`

Follow `cmd_implement.md` structure closely:

- **Manager Role** section: same "never reviews code" constraint, with exception that manager writes cr_plan.md and cr_summary.md directly
- **Progress Tracker** section: reference shared/progress_tracker.md, label "Deep CR Progress"
- **Step list**: Step 0 (Context & diff), Step 1 (Plan), Step 2 (Phase reviews with sub-steps 2a/2b/2c), Step 3 (Summary), Step 4 (Present)
- **Detailed step instructions** for each step:
  - Step 0: diff algorithm from functional spec, git context, spec/task context, review name generation, fork-point confirmation edge case
  - Step 1: phase planning (project-specific + reusable template phases with trigger conditions), write cr_plan.md
  - Step 2: spawn sub-agents per phase using two prompt templates (project-specific and reusable), update progress after each return
  - Step 3: read all phase_N_feedback.md files, write cr_summary.md using functional spec format
  - Step 4: present results (review folder link, issues overview table, critical issues highlighted)
- **Prompt Templates** section: verbatim templates for both project-specific and reusable template phase prompts
- **Autonomous flow**: no user interaction once Step 1 begins (only exception: fork-point confirmation in Step 0)
- **Non-Interactive** guidance: adapted from coding_workflow.md for review context

### 2. Update `SKILL.md`

Add `/spec deep cr` entry between `/spec cr` and the "Bare `/spec` -- Router" section, matching the exact text from architecture.md.

## Files

| Action | File |
|--------|------|
| Create | `references/cmd_deep_code_review.md` |
| Modify | `SKILL.md` |
