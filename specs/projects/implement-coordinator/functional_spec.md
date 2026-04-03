---
status: complete
---

# Functional Spec: Implement Coordinator

## Overview

Restructure the `/spec implement` command so the top-level agent is a strict manager/coordinator. The manager orchestrates the process loop — spawning sub-agents, routing CR feedback, verifying commits — but never writes code or reviews it. All work happens in sub-agents.

This applies to both single-phase (`/spec implement next`, `/spec implement phase N`) and multi-phase (`/spec implement all`). The manager pattern is the same; implement-all just loops it.

## Three Roles

### Manager (top-level implement agent)

**Does:** orchestrate, verify, escalate.
**Does NOT:** code, review, run tests, make technical decisions.

Responsibilities:
- Read implementation plan, determine target phase(s)
- Spawn coding sub-agent for each phase
- Spawn CR sub-agents and route feedback back to coding agent
- Verify commits landed (`git status`)
- Surface phase summaries to user between phases
- Escalate roadblocks from coding agent to user

### Phase Coding Sub-Agent

One per phase. Re-invoked (resumed, not re-spawned) across the CR feedback loop within a phase, preserving its context about what it built.

Three invocation modes:

1. **Initial**: Load specs, write phase plan, implement, run checks/tests → return "ready for CR"
2. **CR fix**: Receive feedback, address issues, run checks → return "ready for re-review"
3. **Commit**: Receive approval, commit with descriptive message, mark phase complete → return commit summary

The coding agent loads its own context from spec files. The manager doesn't pass spec content — just phase number, project path, and a pointer to the instruction file.

### Code Review Sub-Agent

Fresh agent per CR iteration. Gets clean context every time — no coding agent thinking, no accumulated conversation history.

Loads spec files from the repo. Reviews `git diff`. Returns structured feedback with severity labels.

If prior CR feedback exists (re-review), it's passed as a `<prior_cr_feedback>` block so the reviewer can verify prior issues were addressed.

## Manager Flow: Single Phase

Pre-checks (unchanged from current design):
1. Verify active project exists
2. Verify all spec artifacts through `implementation_plan.md` are `status: complete`

Phase loop:

1. **Spawn coding sub-agent** — initial invocation with phase number and project path
2. **Receive response** — either "ready for CR" or "roadblock" (see escalation below)
3. **Spawn CR sub-agent** — fresh context, reviews `git diff`
4. **Receive CR feedback**:
   - If clean: proceed to step 5
   - If issues: **resume coding sub-agent** with CR feedback, then go to step 3 (spawn new CR agent, with prior feedback passed)
5. **Resume coding sub-agent** with approval — tells it to commit
6. **Receive commit confirmation** from coding agent
7. **Verify commit**: run `git status` to confirm working tree is clean and commit exists
8. **Present phase summary** to user

## Manager Flow: Implement All

Same as single phase, but wrapped in a loop:

1. Read implementation plan, find all incomplete phases
2. For each phase: run the single-phase flow above
3. Between phases: show the phase summary, then immediately continue to next phase (don't stop to ask)
4. After all phases: present final summary

## Prompt Templates

The manager sends minimal, well-structured prompts that point to files rather than restating content.

### Initial Coding Prompt

```
You are a coding agent implementing a phase of a spec-driven project.

**Phase:** [N]
**Project specs:** [specs/projects/PROJECT_NAME/]

Read `skill/references/coding_phase_prompt.md` for your full instructions. Follow them precisely.

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
Your code has passed review. Commit all changes with a descriptive message summarizing the work done in this phase. Mark the phase checkbox complete in implementation_plan.md.

Return the commit message you used.
```

### CR Agent Prompt

```
Review code changes for phase [N] of the project at [specs/projects/PROJECT_NAME/].

Read `skill/references/cr_agent_prompt.md` for your full review instructions. Follow them precisely.

[If re-review, append:]
<prior_cr_feedback>
[Previous CR output]
</prior_cr_feedback>
```

## Escalation: Technical Roadblocks

The coding agent's "one exception" rule (pause on genuinely new technical constraints) routes through the manager:

1. Coding agent returns a message describing the roadblock instead of "ready for CR"
2. Manager recognizes this as an escalation (the coding agent's response will clearly indicate it's a roadblock, not a completion)
3. Manager presents the roadblock to the user and waits for a decision
4. Manager resumes the coding agent with the user's decision

## Edge Cases

### CR Loop Doesn't Converge

No explicit loop limit. In practice, CRs converge within 2-3 iterations. If something is genuinely stuck, the coding agent will surface a roadblock through the escalation path.

### Commit Verification Fails

If `git status` shows uncommitted changes after the coding agent says it committed:
- Manager resumes coding agent: "Commit appears incomplete — `git status` shows uncommitted changes. Please commit all changes."
- Verify again after.

### Phase Already Complete

If the target phase checkbox is already checked in `implementation_plan.md`, skip it (for implement-all) or tell the user (for single-phase).

## Out of Scope

- No changes to the planning flow (`/spec new_project`, etc.)
- No changes to the standalone `/spec cr` command (that's user-initiated, not part of the implementation loop)
- No changes to CR agent behavior — only how/when it's spawned changes
- No new commands or routing changes
