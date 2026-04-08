---
status: complete
---

# Functional Spec: Deep Code Review

## Overview

`/spec deep cr` (aliases: `/spec deep review`, `/spec deep code_review`) is a standalone command that runs a thorough, multi-phase agentic code review. The manager orchestrates focused sub-agents, each examining a different concern. All output is written to persistent files under `reviews/projects/[review_name]/`.

This is read-only — no code changes are made (only review artifacts written to `reviews/`).

## Command Invocation

```
/spec deep cr              # review current branch vs fork point
/spec deep cr main         # review current branch vs main
/spec deep cr abc123       # review current branch vs specific commit
```

## Determining the Diff

Compare the current branch against its fork point (the commit where it diverged from its upstream base).

**Algorithm:**
1. If a specific base is provided as an argument, use that — skip steps 2-4
2. Get the current branch name: `git branch --show-current`
3. Check for an open GitHub PR for this branch: `gh pr view --json baseRefName 2>/dev/null`. If a PR exists, use its base branch — this is the most reliable signal for what the change is against
4. Find the upstream tracking branch: `git rev-parse --abbrev-ref @{upstream} 2>/dev/null`
5. If no base found yet, try common defaults: `main`, `master`, `develop` — use the first that exists
6. Find the fork point: `git merge-base <base> HEAD`
7. The diff is: `git diff <fork-point>...HEAD`

If no base can be determined, present the best guess (most common default branch names, most recent merge-base candidates) and ask the user to confirm or correct. Never present a blank prompt — always offer a concrete suggestion.

## Context Loading (Step 0)

### Git Context
- Read `git log --oneline <fork-point>..HEAD` for commit history on the branch
- Read `git diff --stat <fork-point>...HEAD` for a file change summary

### Spec/Task Context
- Read `.specs_skill_state/current_project.md` if it exists
- If it points to a project: read `functional_spec.md` and `architecture.md`
- If it points to a task: read the task file
- Determine if the active spec/task is relevant to the diff (compare file paths, commit messages)
- If relevant: note it for sub-agents to review against. If not: ignore it.

### Review Name Generation
- If active spec/task matches: use the project/task name (slug format)
- Otherwise: generate a descriptive name from the branch name or commit messages
- Check if `reviews/projects/[name]` already exists. If so, append `-v2`, `-v3`, etc.

## Phase Planning (Step 1)

The manager designs a review plan with two categories of phases:

### Project-Specific Phases

Custom phases tailored to the actual diff. The manager reads the diff stat and file contents to identify distinct concern areas worth separate review. Examples:
- "Database migration review" (if migrations are in the diff)
- "Authentication flow review" (if auth code changed)
- "Payment integration review" (if payment logic changed)

Each project-specific phase gets a one-paragraph description of what to focus on, plus the list of files relevant to that phase.

### Reusable Template Phases

Standard phases included when their trigger condition is met. The manager only needs to know the trigger — the detailed review instructions live in resource files that the sub-agent loads directly.

| Phase | Trigger | Resource File |
|-------|---------|---------------|
| UI Design Review | Diff includes UI files (templates, components, CSS, frontend code) | `references/deep_cr/phase_ui_review.md` |
| REST API Review | Diff includes API endpoint definitions, route handlers, or API schemas | `references/deep_cr/phase_api_review.md` |
| Data Model Review | Diff includes database models, migrations, schemas, or ORMs | `references/deep_cr/phase_data_model_review.md` |
| Security Review | Diff includes auth, input handling, secrets, crypto, or new dependencies | `references/deep_cr/phase_security_review.md` |
| Test Quality Review | Diff includes test files | `references/deep_cr/phase_test_review.md` |
| Dependency Review | Diff includes dependency manifest changes (package.json, pyproject.toml, Cargo.toml, etc.) | `references/deep_cr/phase_dependency_review.md` |
| Performance Review | Diff includes code operating at real scale — cloud services, large dataset processing, high-throughput pipelines, database queries on large tables. NOT for typical application code iterating small collections. | `references/deep_cr/phase_performance_review.md` |

The manager may add multiple reusable phases if the diff touches multiple areas. If none trigger, that's fine — project-specific phases cover everything.

### Plan Output

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

## Phases

- [ ] Phase 1: [Name] — [one-line description]
- [ ] Phase 2: [Name] — [one-line description]
- ...
```

## Phase Execution (Step 2)

For each phase, spawn a fresh sub-agent. The sub-agent receives:

**Input (passed in prompt):**
- The review name and phase number
- The diff scope (fork point, relevant file list for this phase)
- The phase description (one paragraph for project-specific, or a pointer to the resource file for reusable phases)
- The base ref for git commands
- Spec/task context path (if relevant)
- Pointer to shared review standards: `references/shared/cr_review_standards.md`

**Sub-agent responsibilities:**
1. Load shared review standards from `references/shared/cr_review_standards.md`
2. If reusable phase: load the phase-specific resource file
3. If spec/task context provided: read the relevant spec artifacts
4. Read the actual code (not just the diff — understand surrounding context)
5. Review with the persona of a senior architect who will own this code
6. Write findings to `reviews/projects/[review_name]/phase_N_feedback.md`

**Output file format (`phase_N_feedback.md`):**

```markdown
# Phase N: [Phase Name]

## Critical (must fix)

- [file:line] **[Issue title]**
  [Description and why it matters]
  [Suggested fix if applicable]

## Moderate (should fix)

[Same format]

## Mild (consider fixing)

[Same format]

## Phase Summary

[2-3 sentences: what was reviewed, overall assessment]
```

**Sub-agent return to manager:** A short summary — phase name, count of issues by severity, any notable concerns. The manager does NOT need the full findings (those are in the file).

## Continuation Without Input (Steps 2-4)

The manager drives all phases to completion without stopping for user input. After each sub-agent returns, update the progress block and immediately spawn the next phase. No exceptions — this is not interactive.

## Summary Generation (Step 3)

After all phases complete, the manager generates `reviews/projects/[review_name]/cr_summary.md` directly. The manager already has the diff context from planning and the phase summaries from sub-agent returns. It also reads all `phase_N_feedback.md` files to get the full findings (sub-agent return summaries may differ from what was written to file).

**Summary format:**

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

## Recommendations

[Top 3-5 prioritized actions]
```

## Final Output (Step 4)

The manager presents to the user:
- Link to the review folder
- The issues overview table (from summary)
- Count of critical/moderate/mild across all phases
- If critical issues exist: highlight them prominently

## Shared Review Standards (Refactor)

Currently, `cr_agent_prompt.md` contains both process instructions (how to run a CR) and review standards (what to look for). This project extracts the "what to look for" into `references/shared/cr_review_standards.md`, referenced by both:
- `cr_agent_prompt.md` (existing single-pass CR)
- Deep CR phase sub-agents

### Content of `cr_review_standards.md`

**Review dimensions** (from existing `cr_agent_prompt.md`):
- Spec/task compliance
- Code quality (architecture, naming, composition, error handling, tests)
- Consistency with existing patterns
- Project-specific standards

**Issues to watch for** (from user's draft + existing):
- Bugs: code that doesn't do what it claims to do
- Poor names: function or class names that don't represent their purpose
- Code in the wrong place: logic in a class/file where it doesn't belong
- Editing globals: rarely a good idea; singletons must be clearly labeled; never set globals on external libs unless this is an application (not a library)
- Python `json.dumps` should always use `ensure_ascii=False`
- GPL or copyleft dependencies: immediate critical failure
- Dead code or unused imports introduced by the change
- Inconsistent error handling patterns
- Missing input validation at system boundaries
- Hardcoded values that should be configurable

**Severity labels** (unchanged):
- Critical — must fix before merging
- Moderate — should fix
- Mild — consider fixing

## Setup Integration

`/spec setup` adds `reviews/` to `.gitignore` (same pattern as `.specs_skill_state/`). This is a one-line addition to the setup command reference.

## Files Created/Modified

### New files
| File | Purpose |
|------|---------|
| `references/cmd_deep_code_review.md` | Manager command reference |
| `references/deep_cr_phase_prompt.md` | Base prompt template for phase sub-agents |
| `references/shared/cr_review_standards.md` | Shared review standards (extracted from cr_agent_prompt.md) |
| `references/deep_cr/phase_ui_review.md` | UI review phase resource |
| `references/deep_cr/phase_api_review.md` | REST API review phase resource |
| `references/deep_cr/phase_data_model_review.md` | Data model review phase resource |
| `references/deep_cr/phase_security_review.md` | Security review phase resource |
| `references/deep_cr/phase_test_review.md` | Test quality review phase resource |
| `references/deep_cr/phase_dependency_review.md` | Dependency review phase resource |
| `references/deep_cr/phase_performance_review.md` | Performance review phase resource |

### Modified files
| File | Change |
|------|--------|
| `SKILL.md` | Add `/spec deep cr` command entry |
| `references/cmd_setup.md` | Add `reviews/` to gitignore step |
| `references/cr_agent_prompt.md` | Replace inline standards with `→ Read shared/cr_review_standards.md` |

## Out of Scope

- Fixing code (this is read-only)
- Integration with `/spec continue` or state tracking
- Comparing arbitrary commits (only branch vs fork-point)
- Running automated checks or tests
- GitHub PR integration (that's `/spec pr`)
