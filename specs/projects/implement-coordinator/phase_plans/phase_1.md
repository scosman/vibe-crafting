---
status: complete
---

# Phase 1: Core Restructuring — Manager Spec and Three-Invocation Coding Prompt

## Overview

Rewrite the two core files that define the implement workflow. `cmd_implement.md` becomes a pure manager specification (orchestrate, don't code). `coding_phase_prompt.md` is restructured around three invocation modes (initial, CR fix, commit) instead of a monolithic flow that self-spawns CR agents.

These files are tightly coupled — the prompt templates in `cmd_implement.md` must match what `coding_phase_prompt.md` expects, and the responsibilities split cleanly between them.

## Steps

### Step 1: Rewrite `skill/references/cmd_implement.md`

Transform from a mixed manager/coder spec into a pure manager specification.

**Keep unchanged:**
- Pre-Checks section (active project, spec completeness)
- Routing section (next/all/phase N)

**Add new sections:**
- Manager Role: what the manager does and does NOT do
- Single Phase Flow: step-by-step orchestration (spawn coding agent → CR loop → commit → verify)
- Prompt Templates: four verbatim templates from functional spec (initial coding, CR feedback resume, commit resume, CR agent)
- Escalation: roadblock handling from functional spec
- Edge cases: commit verification failure, phase already complete

**Remove:**
- Coding Persona section (moves to coding_phase_prompt.md exclusively)
- Implementation Loop steps 1-12 (internal to coding agent)
- CR Iteration Loop (manager's version is simpler: spawn CR, route feedback)
- Non-Interactive Rule (stays in coding_phase_prompt.md only)

**Simplify:**
- Implement All becomes a thin loop over the single-phase flow

### Step 2: Rewrite `skill/references/coding_phase_prompt.md`

Restructure from a monolithic flow into three invocation modes.

**Keep unchanged:**
- Your Role section (senior IC persona)
- Context Loading section

**Restructure:**
- Replace "Implementation Loop" with "Initial Invocation: Plan and Implement" (write phase plan + build/test/check steps, ending with "return summary")
- Add "CR Feedback Invocation: Address Review" (read feedback, fix, check, return summary)
- Add "Commit Invocation: Finalize" (commit, mark complete, return message)

**Remove:**
- CR Iteration Loop (manager handles CR spawning now)
- Self code-review step (step 6 of current Implementation Loop)
- Any reference to spawning CR sub-agents

**Update:**
- Non-Interactive rule: add clarity that escalation means returning a roadblock message, not a completion summary
- Completion section: describe what "done" means for each invocation mode
- Design note: reflect three-invocation pattern

### Step 3: Verify consistency

- Prompt templates in cmd_implement.md reference `skill/references/coding_phase_prompt.md` and `skill/references/cr_agent_prompt.md` correctly
- coding_phase_prompt.md contains NO CR spawning logic
- cmd_implement.md contains NO coding-level detail (persona, implementation internals)
- Cross-references between files resolve
- Terminology is consistent across both files
