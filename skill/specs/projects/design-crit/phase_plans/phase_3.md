---
status: complete
---

# Phase 3: Manager Command Reference and SKILL.md

## Overview

Create the main manager command reference file (`references/cmd_design_crit.md`) that defines the full step-by-step flow for the `/spec design crit` command. This is the most complex file in the project -- it orchestrates everything built in phases 1 and 2. Also update `SKILL.md` with a command entry.

## Steps

1. Create `references/cmd_design_crit.md` with:
   - Manager role declaration (never reviews specs, only orchestrates)
   - Progress tracker reference with "Design Crit Progress" label
   - Step 0: Scope identification (read current project or argument, scan spec files, confirm scope with user)
   - Step 1: Plan (design review phases -- spec-specific + reusable template phases, write crit_plan.md, get user approval)
   - Step 2: Phase reviews (spawn sub-agents, update progress, autonomous flow)
   - Step 3: Summary (read phase feedback, write crit_summary.md with Issue Queue using DC-* IDs)
   - Step 4: Present (show results, offer resolution)
   - Step 5: Resolution phase (5a-5f: load queue, present groups, process decisions, write refinement files, verify completeness, fold into specs)
   - Autonomous flow rules (Steps 2-4 no user input; Step 5 fully interactive)
   - Two prompt templates (spec-specific and reusable template phases)
   - References section

2. Update `SKILL.md`: Add `/spec design crit` command entry after the `/spec deep cr` entry, following the pattern from the architecture doc.

## Tests

- NA (markdown/prompt-writing project, no automated tests)
