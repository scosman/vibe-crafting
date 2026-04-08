---
status: complete
---

# Deep Code Review

A new command for the spec skill (`/spec deep cr` or `/spec deep review`) that provides a thorough, multi-phase agentic code review. Unlike the existing `/spec cr` (single-pass review), this uses the manager pattern to orchestrate multiple focused review sub-agents, each examining a different concern.

## What It Does

Compares the current branch against its fork point, then runs a structured multi-phase code review:

1. **Context loading**: Reads git log, checks for active spec/task, generates a review name
2. **Phase planning**: Designs review phases tailored to the diff (project-specific + reusable templates)
3. **Phased review execution**: Spawns a sub-agent per phase, each focused on one concern area
4. **Summary generation**: Produces a consolidated issues summary and change overview

All output is written to `reviews/projects/[review_name]/` — the review is persistent, not just printed.

## Key Differences from `/spec cr`

| Aspect | `/spec cr` | `/spec deep cr` |
|--------|-----------|-----------------|
| Agent pattern | Single sub-agent | Manager + N sub-agents |
| Output | Printed to conversation | Written to `reviews/` files |
| Scope | Single-pass | Multi-phase, concern-separated |
| Planning | None | Review plan with phases |
| Reusability | No artifacts | Persistent review artifacts |

## What It Produces

- `reviews/projects/[name]/cr_plan.md` — Review phases with completion tracking
- `reviews/projects/[name]/phase_N_feedback.md` — Per-phase findings (critical/moderate/mild)
- `reviews/projects/[name]/cr_summary.md` — What the PR does, architectural decisions, quality assessment

## Integration Points

- Reuses existing CR severity labels (critical/moderate/mild) from `cr_agent_prompt.md`
- Follows manager pattern from `/spec implement` (progress blocks, autonomous flow)
- Reuses subagent spawning patterns from `spawning_subagents.md`
- `reviews/` directory added to `.gitignore` via `/spec setup`
- References existing issues-to-watch-for from `cr_agent_prompt.md` rather than duplicating

## Constraints

- Fully autonomous — no user interaction during review
- Manager never reviews code — only orchestrates
- Each phase sub-agent gets clean context (no bleed between phases)
- Works without an active spec/task (but uses spec context when available)
