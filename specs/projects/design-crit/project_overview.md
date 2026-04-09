---
status: complete
---

# Design Crit Command

Adding a "design crit" sub-command to the `/spec` skill. Invokable as `/spec design crit` or `/spec crit`.

## What It Does

Multi-phase agentic review of spec/design documents (not code). For larger or more technical projects, we want extra deep reviews of the design before starting implementation. Similar to a "deep cr" command, but focused on reviewing design not code.

## Motivation

Currently this is emulated by hacking the `deep cr` command with custom instructions to review spec files instead of code diffs. It works but is awkward — deep CR is built around git diffs, fork points, and code review standards. A dedicated command that directly says "this extends the process for design review" would be cleaner and more effective.

## How It Follows Deep CR Patterns

- High-level manager orchestrates phases, dispatches review sub-agents, summarizes
- Sub-agents handle each review phase independently
- Saves results to `reviews/...` directory
- Plan approval gate before autonomous review phases

## What's Different from Deep CR

- Clarified that it's for specs, not code (adapted CR process)
- Requires clarity on what we're reviewing — can suggest current project specs, but doesn't start without absolute clarity and agreement from user
- Phase templates focused on design concerns (completeness, feasibility, architecture, etc.) rather than code concerns (security, tests, dependencies)
- Resolution phase: actually fix the issues found

## Resolution Phase (new concept)

A new interactive phase to work through found issues:

1. Read review docs and build issue queue
2. Work through queue interactively: making decisions for each issue, saving results to `reviews/[path]/refinements/NN_[name].md`. Track won't-fix decisions in `wont_fix.md`. Present issues in priority order, grouping related items and quick decisions.
3. When queue is done, verify all issues are addressed (either in refinements or wont_fix)
4. Fold accepted refinements into the actual spec files

## Context

This is a writing/config project — creating markdown reference files that instruct the AI manager and sub-agents. No application code. The "codebase" is the skill itself at the skill base directory.
