# `/spec deep cr` — Deep Code Review

Multi-phase agentic code review. Compares the current branch against its fork point, designs review phases tailored to the diff, runs focused sub-agents per phase, and produces persistent review artifacts.

## Manager Role

**You are a manager. You do NOT review code, make technical judgments about code quality, or write review feedback — ever.** If you catch yourself about to analyze code for issues, stop. You are in the wrong role. Your only tools are: spawning sub-agents, running git commands, reading/writing review plan and summary files, and outputting progress blocks.

The manager's responsibilities:
- Load git context and determine the diff scope
- Design review phases tailored to the diff
- Write `cr_plan.md` (the review plan)
- Spawn phase sub-agents and track their progress
- Read phase feedback files and write `cr_summary.md` (the consolidated summary)
- Present results to the user

**Exception to read-only:** The manager writes `cr_plan.md` and `cr_summary.md` directly. These are orchestration artifacts, not code review. All actual review work is delegated to sub-agents.

**Important:** even if asked to review code by the user, default to using sub-agents per these instructions, unless the user specifically requests you do it in this context. You are a manager: delegate.

## The Pattern

This command is an instance of the shared fan-out pattern: scope → plan → fan out one sub-agent per unit → wait → collapse into one summary.

→ Read [references/shared/fan_out_pattern.md](shared/fan_out_pattern.md) for those mechanics — unit sizing, the plan artifact, plan approval, dispatch and the per-return loop, model selection, re-dispatching failures, and how the collapse works. Follow them precisely.

This file supplies the deep-CR specifics. **Working folder: `reviews/projects/[review_name]/`** — created in Step 0, which also settles the review name and collision handling. The unit is a **review phase**, the plan is `cr_plan.md`, the consolidated summary is `cr_summary.md`, both at the top of the working folder, and **the manager writes the summary itself** rather than dispatching a summary sub-agent. Each phase's focus paragraph and file list travel in its dispatch prompt, not in the plan — the plan stays a checklist.

## Progress Tracker

→ Read [references/shared/progress_tracker.md](shared/progress_tracker.md) for the progress block format, round counters, and rules. Follow them precisely.

Use the label **"Deep CR Progress"** for the progress block. The full step list for this command:

```
- Step 0: Context & diff
- Step 1: Plan
- Step 2a: Phase 1 ([name])
- Step 2b: Phase 2 ([name])
- Step 2c: Phase 3 ([name])
  ... (one sub-step per phase, determined during Step 1)
- Step 3: Summary
- Step 4: Present
```

Example mid-flow:

```
<progress>
Deep CR Progress:
- [x] Step 0: Context & diff — complete
- [x] Step 1: Plan — complete (5 phases)
- [x] Step 2a: Phase 1 (Auth flow) — complete (1 critical, 2 moderate)
- [x] Step 2b: Phase 2 (DB migrations) — complete (0 critical, 1 moderate)
- [ ] Step 2c: Phase 3 (UI review) — in progress
- [ ] Step 2d: Phase 4 (Security) — pending
- [ ] Step 2e: Phase 5 (Test quality) — pending
- [ ] Step 3: Summary — pending
- [ ] Step 4: Present — pending
</progress>
```

## Step 0: Context & Diff

### Determine the Diff

Compare the current branch against its fork point (the commit where it diverged from its upstream base).

**Algorithm:**
1. If a specific base was provided as an argument to the command, use that — skip steps 2-5
2. Get the current branch name: `git branch --show-current`
3. Check for an open GitHub PR for this branch: `gh pr view --json baseRefName 2>/dev/null`. If a PR exists, use its base branch — this is the most reliable signal for what the change is against
4. Walk the git log looking for the fork point: run `git log --format='%H %D'` and find the first commit that has a branch ref other than the current branch and HEAD. That branch is the base. For example, if the log shows a commit decorated with `origin/leonard/chat-integration, leonard/chat-integration`, the base is `leonard/chat-integration`. Prefer local branch names over `origin/` prefixes when both exist.
5. If the log walk finds nothing (e.g., very long-lived branch with no visible refs), fall back to common defaults: `main`, `master`, `develop` — use the first that exists
6. Find the fork point: `git merge-base <base> HEAD`
7. The diff is: `git diff <fork-point>...HEAD`

If no base can be determined, present the best guess and ask the user to confirm or correct. Never present a blank prompt — always offer a concrete suggestion. This is the only place before plan approval where the manager stops for user input.

### Load Git Context

- Read `git log --oneline <fork-point>..HEAD` for commit history on the branch
- Read `git diff --stat <fork-point>...HEAD` for a file change summary

### Load Spec/Task Context

- Read `.specs_skill_state/current_project.md` if it exists
- If it points to a project: read `functional_spec.md` and `architecture.md`
- If it points to a task: read the task file
- Determine if the active spec/task is relevant to the diff (compare file paths, commit messages)
- If relevant: note it for sub-agents to review against. If not: ignore it.

### Generate Review Name

- If active spec/task matches: use the project/task name (slug format)
- Otherwise: generate a descriptive name from the branch name or commit messages
- Check if `reviews/projects/[name]` already exists. If so, append `-v2`, `-v3`, etc.
- Create the directory: `reviews/projects/[review_name]/`

## Step 1: Plan

Design the review phases. There are two categories:

### Project-Specific Phases

Custom phases tailored to the actual diff. Read the diff stat and file contents to identify distinct concern areas worth separate review. Examples:
- "Database migration review" (if migrations are in the diff)
- "Authentication flow review" (if auth code changed)
- "Payment integration review" (if payment logic changed)

Each project-specific phase gets a focus paragraph (per the shared pattern) plus the list of files relevant to that phase.

### Reusable Template Phases

Standard phases included when their trigger condition is met. You only need to evaluate the trigger — the detailed review instructions live in resource files that the sub-agent loads directly.

| Phase | Trigger | Resource File |
|-------|---------|---------------|
| UI Design Review | Diff includes UI files (templates, components, CSS, frontend code) | `references/deep_cr/phase_ui_review.md` |
| REST API Review | Diff includes API endpoint definitions, route handlers, or API schemas | `references/deep_cr/phase_api_review.md` |
| Data Model Review | Diff includes database models, migrations, schemas, or ORMs | `references/deep_cr/phase_data_model_review.md` |
| Security Review | Diff includes auth, input handling, secrets, crypto, or new dependencies | `references/deep_cr/phase_security_review.md` |
| Test Quality Review | Diff includes test files | `references/deep_cr/phase_test_review.md` |
| Dependency Review | Diff includes dependency manifest changes (package.json, pyproject.toml, Cargo.toml, etc.) | `references/deep_cr/phase_dependency_review.md` |
| Performance Review | Diff includes code operating at real scale — cloud services, large dataset processing, high-throughput pipelines, database queries on large tables. NOT for typical application code iterating small collections. | `references/deep_cr/phase_performance_review.md` |

You may add multiple reusable phases if the diff touches multiple areas. If none trigger, that's fine — project-specific phases cover everything.

### Write the Plan

Write to `reviews/projects/[review_name]/cr_plan.md`:

```markdown
# Code Review Plan: [review_name]

## Branch
- Current: [branch_name]
- Base: [base_branch_name]
- Fork point: [commit_hash]
- Files changed: [count]

## Spec Context
[Link to relevant spec/task, or "None — standalone review"]

## Run
- Model: [model the phase agents will run on]

## Phases

- [ ] Phase 1: [Name] — [one-line description]
- [ ] Phase 2: [Name] — [one-line description]
- ...
```

### Present Plan for Approval

Show the user the plan: base branch, fork point, spec context, the model the phase agents will run on, and the list of phases. Ask them to confirm before proceeding:

> Review plan ready — [N] phases planned for branch `[branch_name]` against `[base_branch_name]`:
>
> [Phase list from cr_plan.md]
>
> Proceed with the review?

Approval is required — dispatch nothing until you have it. If the user wants phases added, dropped, or re-scoped, update `cr_plan.md` in place and re-confirm.

A base-branch correction is the one change that reaches back into Step 0, and it reaches back **partially**: re-determine the fork point and reload the git context against the corrected base, then rewrite `cr_plan.md` in place. **Keep the review name and folder you already created** — do not re-run Generate Review Name, which would append `-v2` and orphan the plan the user is looking at.

## Step 2: Phase Reviews

One fresh sub-agent per phase, dispatched and tracked per the [fan-out pattern](shared/fan_out_pattern.md#fan-out) — including re-dispatching any phase whose agent errors or returns without writing its feedback file.

Use the prompt template below that matches the phase: Project-Specific or Reusable Template. Each phase writes `reviews/projects/[review_name]/phase_[N]_feedback.md`; the per-phase result you record in the progress block is its issue counts by severity.

## Step 3: Summary

Collapse per the [fan-out pattern](shared/fan_out_pattern.md#collapse) — the manager writes this one. Read all `reviews/projects/[review_name]/phase_N_feedback.md` files, then write `reviews/projects/[review_name]/cr_summary.md`:

```markdown
# Code Review Summary: [review_name]

## What This Change Does

[2-4 sentences describing the PR/branch purpose]

## Key Architectural Decisions

- [Decision]: [rationale and assessment]
- ...

## Quality Assessment

[Overall assessment of code quality, test coverage, readability]

## Issues Overview

| Severity | Count | Key Themes |
|----------|-------|------------|
| Critical | N | [brief themes] |
| Moderate | N | [brief themes] |
| Mild | N | [brief themes] |

## Critical Issues (Requires Attention)

[List all critical issues across all phases, with file references]

## Incomplete Phases

[Any phase that did not complete after the re-dispatch attempt cap — including one that wrote partial findings. Say what it did cover and what it leaves unreviewed. "None — all phases completed." if clean.]

## Recommendations

[Top 3-5 prioritized actions]
```

## Step 4: Present

Show the user:
- Link to the review folder: `reviews/projects/[review_name]/`
- The Issues Overview table from the summary
- Total counts: N critical, N moderate, N mild across all phases
- Any phase that could not be completed, and what it leaves unreviewed
- If critical issues exist: list them prominently with file references

## Autonomous Flow

→ [fan-out pattern: Autonomous Flow](shared/fan_out_pattern.md#autonomous-flow) and [Non-Interactive](shared/fan_out_pattern.md#non-interactive). Read both — a review that stops halfway to ask a question is the failure mode they exist to prevent.

**Work autonomously. Don't ask the user for help or confirmation during the review.**

This command pauses at exactly two points: **Step 0**, if the fork point cannot be determined automatically, and **end of Step 1**, for plan approval. From Step 2 onward the run is fully autonomous through summary and presentation.

## Prompt Templates

These are the exact prompts the manager sends to sub-agents. Use them verbatim, filling in the bracketed values.

### Project-Specific Phase Prompt

```
You are a code review sub-agent performing a focused review of a specific concern area.

**Review name:** [review_name]
**Phase:** [N] — [Phase Name]
**Focus:** [one-paragraph description of what to review]
**Files to review:** [file list relevant to this phase]
**Base ref:** [fork_point_hash]
**Spec context:** [path to spec/task, or "None"]

Read `references/deep_cr_phase_prompt.md` for your full review instructions. Follow them precisely.

Write your findings to `reviews/projects/[review_name]/phase_[N]_feedback.md`.

Return a short summary: phase name, issue counts by severity, notable concerns.
```

### Reusable Template Phase Prompt

```
You are a code review sub-agent performing a focused review of a specific concern area.

**Review name:** [review_name]
**Phase:** [N] — [Phase Name]
**Phase resource:** [references/deep_cr/phase_X_review.md] — read this for detailed review guidance
**Files to review:** [file list relevant to this phase]
**Base ref:** [fork_point_hash]
**Spec context:** [path to spec/task, or "None"]

Read `references/deep_cr_phase_prompt.md` for your full review instructions. Follow them precisely.

Write your findings to `reviews/projects/[review_name]/phase_[N]_feedback.md`.

Return a short summary: phase name, issue counts by severity, notable concerns.
```

## References

- [references/shared/fan_out_pattern.md](shared/fan_out_pattern.md) — The shared plan → fan out → collapse mechanics
- [references/spawning_subagents.md](spawning_subagents.md) — How to spawn sub-agents
- [references/deep_cr_phase_prompt.md](deep_cr_phase_prompt.md) — Full instructions for phase sub-agents
- [references/shared/cr_review_standards.md](shared/cr_review_standards.md) — Shared review standards (loaded by sub-agents, not the manager)
