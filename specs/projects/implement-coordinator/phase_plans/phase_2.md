---
status: complete
---

# Phase 2: Supporting Files and Validation

## Overview

Update `spawning_subagents.md` with the resume pattern (needed by the manager's CR feedback loop), verify `cr_agent_prompt.md` is self-contained, then audit all four reference files for cross-reference integrity and consistency.

## Steps

1. Add a "Resuming Sub-Agents" section to `skill/references/spawning_subagents.md` covering the resume pattern for Claude Code and Cursor, plus guidance on when to resume vs. fresh spawn.
2. Read `skill/references/cr_agent_prompt.md` and verify it is fully self-contained — no implicit dependency on being spawned by the coding agent, no dangling references.
3. Cross-reference audit: verify every file reference in the four reference files resolves, check self-containment of the two sub-agent prompts, verify terminology consistency, and walk through single-phase and implement-all flows.
4. Fix any inconsistencies found during the audit.

## Tests

1. Self-containment check: `coding_phase_prompt.md` and `cr_agent_prompt.md` make sense read in isolation
2. Cross-reference audit: every file reference in all four files resolves to a real section or file
3. Consistency check: terminology, prompt templates, and role descriptions are uniform
4. Scenario walkthrough: single-phase and implement-all flows trace cleanly with every step having a clear owner
