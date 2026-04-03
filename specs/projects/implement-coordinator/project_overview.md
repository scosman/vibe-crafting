---
status: complete
---

# Implement Coordinator

## Problem

The "implement" sub-skill has trouble staying on task during long projects. As context grows, the agent starts forgetting the process flow — skipping agentic CRs, forgetting to commit, drifting from the defined workflow. The current `cmd_implement.md` describes a "Coordinator Context" concept for `implement all`, but it's not working in practice because the coordinator still does too much itself and loses discipline.

## Goal

Restructure the implement command so the top-level agent is a strict manager/coordinator that does nothing but enforce process flow. All actual work happens in sub-agents. This keeps the coordinator's context small and focused — it never gets buried in code details or loses track of where it is in the loop.

## Architecture: Three Roles

### 1. Manager (top-level implement agent)

The manager's only job is orchestrating the phase loop. It does NOT code, does NOT review — it just ensures the right sub-agents are spawned in the right order, checks that work is committed, and moves to the next phase.

Responsibilities:
- Read `implementation_plan.md` to determine which phases remain
- For each phase: spawn a coding sub-agent, then a CR loop, then a commit step
- Verify commits landed (via `git status`) before moving to the next phase
- Surface phase summaries to the user between phases
- Stop and escalate only if a sub-agent reports a genuine technical roadblock

### 2. Phase Coding Sub-Agent

A single sub-agent instance per phase that handles all coding work:
- Writes the phase plan
- Implements the code
- Runs automated checks
- Addresses CR feedback (re-invoked with feedback, not re-spawned — preserves coding context)
- Generates commit message and commits when CR passes

The coding agent loads its own context from spec files — the manager passes it minimal, well-structured prompts that point to files rather than restating content.

This agent is re-used across the CR feedback loop within a single phase — same context window, so it remembers what it built and can efficiently address feedback.

### 3. Code Review Sub-Agent

A fresh sub-agent spawned for each CR iteration. Gets clean context every time — no coding agent thinking, no prior CR history except what's explicitly passed in `<prior_cr_feedback>`.

Loads spec files from the repo. Reviews `git diff`.

## Prompt Design Principles

- **Point, don't paste.** Manager prompts should link to spec files and reference files, not restate their content. The coding agent and CR agent load their own context.
- **Structured but minimal.** Manager prompts should be well-formatted and unambiguous, but short. No walls of text.
- **Self-loading sub-agents.** Both coding and CR sub-agents have reference files (`coding_phase_prompt.md`, `cr_agent_prompt.md`) that contain their full instructions. The manager tells them which file to read and what phase to work on.

## Key Behavioral Changes from Current Design

1. **Manager never codes.** Today the single-phase flow has the same agent code AND manage CRs. The manager should only orchestrate.
2. **Manager verifies commits.** After the coding agent says it committed, the manager runs `git status` to confirm. Don't trust — verify.
3. **Coding agent is re-used within a phase.** For CR feedback loops, the same coding agent is re-invoked (not a fresh one). This preserves context about what was built.
4. **CR agent is always fresh.** Every CR iteration spawns a new agent with clean context, per the existing design.
5. **The "one exception" rule must exist in the coding phase prompt.** The non-interactive rule with the technical-roadblock exception currently exists in `cmd_implement.md`. It needs to be in `coding_phase_prompt.md` too (or instead), since that's what the coding sub-agent actually reads.

## Files to Modify

- `skill/references/cmd_implement.md` — Rewrite to define the manager role and the phase loop with sub-agents. Remove the coding-level detail (that belongs in `coding_phase_prompt.md`).
- `skill/references/coding_phase_prompt.md` — Update to be the complete, self-contained prompt for the coding sub-agent. Ensure it includes the non-interactive rule with the "one exception." Ensure it covers the commit step at the end.
- `skill/references/cr_agent_prompt.md` — Likely minimal changes. Verify it's fully self-contained.
- `skill/references/spawning_subagents.md` — May need updates to describe the re-invocation pattern (resuming a coding agent vs. spawning fresh CR agents).
