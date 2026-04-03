---
status: complete
---

# Architecture: Implement Coordinator

## Overview

Four files in `skill/references/` need changes. No new files are created. The changes restructure responsibilities: `cmd_implement.md` becomes the manager spec, `coding_phase_prompt.md` becomes the coding agent spec (with CR spawning removed), `cr_agent_prompt.md` gets minor updates, and `spawning_subagents.md` documents the resume pattern.

## File Changes

### 1. `skill/references/cmd_implement.md` — Major Rewrite

**Current:** Mixes manager-level orchestration with coding-level detail (persona, implementation loop, CR iteration loop). The "Implement All" section describes a coordinator, but "Single Phase" describes a monolithic agent that does everything.

**New:** Pure manager specification. Two sections: single-phase and implement-all (which is just a loop over single-phase). No coding persona, no implementation details — those live in `coding_phase_prompt.md`.

#### Structure

```markdown
# `/spec implement` — Implement Project

## Pre-Checks
[Unchanged: active project, spec completeness]

## Routing
[Unchanged: next/all/phase N]

## Manager Role
[New: what the manager does and doesn't do]

## Single Phase Flow
[New: step-by-step manager orchestration]
### Step 1: Spawn Coding Agent
### Step 2: CR Loop
### Step 3: Commit
### Step 4: Verify

## Implement All
[Simplified: loop over single-phase flow]

## Prompt Templates
[New: exact prompts the manager sends for each step]
### Initial Coding Prompt
### CR Feedback Prompt (resume)
### Commit Prompt (resume)
### CR Agent Prompt

## Escalation
[New: roadblock handling]

## References
[Updated links]
```

#### Key Content

**Manager Role section** — Short and emphatic:
- Manager orchestrates, does not code or review
- Manager sends minimal prompts that point to reference files
- Manager verifies commits via `git status`
- Manager surfaces roadblocks to user

**Single Phase Flow section** — The numbered orchestration steps from the functional spec. Each step says what the manager does (spawn/resume/verify), not what the sub-agent does internally.

**Prompt Templates section** — The four prompt templates from the functional spec, verbatim. These are the actual text the manager passes to sub-agents.

**Implement All section** — Collapses to: read implementation plan, loop over incomplete phases running the single-phase flow, present summary between phases, don't stop to ask.

#### What Gets Removed

- Coding Persona section (moves entirely to `coding_phase_prompt.md`)
- Implementation Loop (steps 1-12) — the internal coding steps are the coding agent's concern
- CR Iteration Loop — the manager's version is simpler (spawn CR, route feedback, repeat)
- The "Non-Interactive Rule" — stays in `coding_phase_prompt.md` only; the manager is interactive (it surfaces roadblocks to users)

### 2. `skill/references/coding_phase_prompt.md` — Restructure

**Current:** Describes implementation loop including self-spawning CR sub-agents.

**New:** Describes what the coding agent does in each of its three invocation modes. CR spawning is removed — the manager handles that.

#### Structure

```markdown
# Coding Phase Prompt

[Design note: self-contained, passed to coding sub-agent]

---

## Your Role
[Unchanged: senior IC persona]

## Context Loading
[Unchanged: read specs]

## Initial Invocation: Plan and Implement

### Write Phase Plan
[Unchanged]

### Implementation Steps
1. Build the code
2. Run automated checks, iterate until clean
3. Write tests
4. Run tests, iterate until passing
5. Run automated checks again, iterate until clean
6. Return summary — ready for code review

## CR Feedback Invocation: Address Review

1. Read the feedback provided in your prompt
2. Address each issue (fix or explain with code comment)
3. Run automated checks, iterate until clean
4. Return summary — ready for re-review

## Commit Invocation: Finalize

1. Commit all changes with the descriptive message from your prompt
2. Mark phase checkbox in implementation_plan.md
3. Mark phase plan status: complete
4. Return the commit message used

## Non-Interactive Rule
[Unchanged: autonomous, one exception for genuine roadblocks]
[Add: when escalating, return a clear roadblock message instead of a completion summary]

## Completion
[Updated: describes what "done" means for each invocation mode]
```

#### Key Changes

- **Three invocation modes** replace the single monolithic flow. The coding agent needs to handle being invoked fresh (initial), resumed with feedback (CR fix), and resumed with approval (commit).
- **CR spawning removed.** Steps 6-7 of the current Implementation Loop (spawn CR sub-agent, iterate) are deleted. The coding agent's job now ends at "return ready for CR" — the manager takes over from there.
- **Commit step added.** Currently the coding agent doesn't explicitly commit (the implement-all coordinator does). Now the coding agent commits when told to, as its third invocation mode.
- **"One exception" rule stays** and adds clarity: when hitting a roadblock, return a message that clearly signals it's a roadblock, not a completion. The manager will recognize this and escalate.
- **Design note updated** to reflect the three-invocation pattern.

### 3. `skill/references/cr_agent_prompt.md` — Minimal Changes

The CR agent prompt is already self-contained and well-structured. Changes:

- **Context Loading section**: currently hardcodes `specs/projects/PROJECT_NAME/`. The manager's CR prompt template already passes the project path, so this doesn't need to change — the CR agent just reads the path from its prompt. No structural change needed.
- **Verify the prompt is fully self-contained** with no implicit dependency on being spawned by the coding agent. It already is — the design note at the bottom confirms this.

Net change: likely zero or cosmetic only. Verify during implementation.

### 4. `skill/references/spawning_subagents.md` — Add Resume Pattern

**Current:** Describes spawning fresh sub-agents only.

**New:** Add a section on resuming/re-invoking sub-agents (for the coding agent CR feedback loop).

#### Addition

```markdown
## Resuming Sub-Agents

Some workflows need to send follow-up messages to an existing sub-agent
rather than spawning a fresh one. This preserves the agent's context.

Use when: the sub-agent needs to continue work it already started
(e.g., a coding agent addressing CR feedback on code it just wrote).

### Claude Code
Use Task() with the agent ID from the initial spawn.

### Cursor
Use the Task tool with the `resume` parameter, passing the agent ID.

### When to Resume vs. Fresh Spawn
- **Resume**: coding agent receiving CR feedback (needs its prior context)
- **Fresh spawn**: CR agent (must NOT have coding context)
```

## Cross-References

After changes, the reference loading map:

| File | Loads | Loaded By |
|------|-------|-----------|
| `cmd_implement.md` | — | `SKILL.md` (link) |
| `coding_phase_prompt.md` | `cr_agent_prompt.md` (removed) | `cmd_implement.md` (prompt template points here) |
| `cr_agent_prompt.md` | — | `cmd_implement.md` (prompt template points here) |
| `spawning_subagents.md` | — | `cmd_implement.md` (link) |

Note: `coding_phase_prompt.md` no longer references `cr_agent_prompt.md` since it doesn't spawn CR agents anymore. The manager (`cmd_implement.md`) references both.

## Testing Strategy

This is a writing project — no automated tests. Validation is:

1. **Self-containment check**: `coding_phase_prompt.md` and `cr_agent_prompt.md` make sense read in isolation (no dangling references to manager context)
2. **Cross-reference audit**: every file reference resolves
3. **Consistency check**: terminology, prompt templates, and role descriptions are uniform
4. **Scenario walkthrough**: mentally trace through single-phase and implement-all flows using the new files, verifying every step has a clear owner (manager vs. coding agent vs. CR agent)
