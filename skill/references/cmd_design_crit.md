# `/spec design crit` — Design Crit

Multi-phase agentic review of spec/design documents. Identifies design concerns, runs focused sub-agents per concern area, produces persistent review artifacts, and offers an interactive resolution phase to triage and fix issues found.

## Manager Role

**You are a manager. You do NOT review specs, make technical judgments about design quality, or write review feedback — ever.** If you catch yourself about to analyze a spec for issues, stop. You are in the wrong role. Your only tools are: spawning sub-agents, reading spec file metadata, reading/writing review plan and summary files, driving the resolution phase interactively, and outputting progress blocks.

The manager's responsibilities:
- Identify spec files in scope and confirm with the user
- Design review phases tailored to the spec content
- Write `crit_plan.md` (the review plan)
- Spawn phase sub-agents and track their progress
- Read phase feedback files and write `crit_summary.md` (the consolidated summary with Issue Queue)
- Present results to the user
- Drive the interactive resolution phase

**Exception to read-only:** The manager writes `crit_plan.md` and `crit_summary.md` directly. These are orchestration artifacts, not design review. All actual review work is delegated to sub-agents. The manager also drives the resolution phase (Step 5), which involves writing refinement files and editing spec files based on user decisions.

**Important:** even if asked to review specs by the user, default to using sub-agents per these instructions, unless the user specifically requests you do it in this context. You are a manager: delegate.

## Progress Tracker

→ Read [references/shared/progress_tracker.md](shared/progress_tracker.md) for the progress block format, round counters, and rules. Follow them precisely.

Use the label **"Design Crit Progress"** for the progress block. The full step list for this command:

```
- Step 0: Scope
- Step 1: Plan
- Step 2a: Phase 1 ([name])
- Step 2b: Phase 2 ([name])
- Step 2c: Phase 3 ([name])
  ... (one sub-step per phase, determined during Step 1)
- Step 3: Summary
- Step 4: Present
- Step 5: Resolution (interactive)
```

Example mid-flow:

```
<progress>
Design Crit Progress:
- [x] Step 0: Scope — complete (5 spec files)
- [x] Step 1: Plan — complete (4 phases)
- [x] Step 2a: Phase 1 (Completeness) — complete (0 critical, 3 moderate)
- [x] Step 2b: Phase 2 (Architecture) — complete (1 critical, 2 moderate)
- [ ] Step 2c: Phase 3 (Security Design) — in progress
- [ ] Step 2d: Phase 4 (Consistency) — pending
- [ ] Step 3: Summary — pending
- [ ] Step 4: Present — pending
- [ ] Step 5: Resolution — pending (interactive)
</progress>
```

## Step 0: Scope — Identify and Confirm Spec Files

Unlike deep CR which uses `git diff`, design crit reviews spec files directly.

### Determine What to Review

**Algorithm:**
1. If a specific project or path was provided as an argument, use that
2. Otherwise, read `.specs_skill_state/current_project.md` to find the active project
3. If no active project exists and no path was provided, tell the user and suggest they either specify a path or set an active project. Stop here.
4. Scan the project directory for spec artifacts: `project_overview.md`, `functional_spec.md`, `ui_design.md`, `architecture.md`, `components/*.md`, `implementation_plan.md`
5. Read each file's frontmatter to get its status (draft/complete)

### Confirm Scope

Present the identified scope to the user:

```
Design crit scope for project `[name]`:

- `functional_spec.md` (complete)
- `architecture.md` (complete)
- `components/auth.md` (complete)
- `components/data_pipeline.md` (draft)
- `implementation_plan.md` (complete)

Review all of these, or a subset?
```

**Do NOT proceed without explicit user confirmation of scope** — this is non-negotiable.

The user may narrow scope (e.g., "just architecture and components") or confirm all. Adjust accordingly.

### Generate Review Name

- Use the project name with `-crit` suffix (slug format): e.g., `my-project-crit`
- Check if `reviews/projects/[name]` already exists. If so, append `-v2`, `-v3`, etc.
- Create the directory: `reviews/projects/[review_name]/`

## Step 1: Plan

Design the review phases. There are two categories:

### Spec-Specific Phases

Custom phases tailored to the actual spec content. Read the spec files to identify distinct concern areas worth separate, focused review. Examples:
- "Payment flow design review" (if the spec includes payment handling)
- "Multi-tenancy design review" (if the spec describes tenant isolation)
- "Migration strategy review" (if there's an upgrade/migration path)
- "Concurrency model review" (if the spec describes concurrent operations)

Each spec-specific phase gets a one-paragraph description of what to focus on, plus the list of spec files relevant to that phase.

### Reusable Template Phases

Standard phases included when their trigger condition is met. You only need to evaluate the trigger — the detailed review instructions live in resource files that the sub-agent loads directly.

| Phase | Trigger | Resource File |
|-------|---------|---------------|
| Completeness & Coverage | Always (when any spec file beyond project_overview is in scope) | `references/design_crit/phase_completeness_review.md` |
| Technical Feasibility | Architecture or component specs are in scope | `references/design_crit/phase_feasibility_review.md` |
| Architecture Review | Architecture or component specs are in scope | `references/design_crit/phase_architecture_review.md` |
| API/Contract Design | Spec describes APIs, endpoints, interfaces, or service contracts | `references/design_crit/phase_api_design_review.md` |
| Data Model Design | Spec describes data models, schemas, or storage design | `references/design_crit/phase_data_model_design_review.md` |
| Security Design | Spec involves auth, user data, external integrations, or trust boundaries | `references/design_crit/phase_security_design_review.md` |
| Spec Consistency | Two or more spec files are in scope | `references/design_crit/phase_consistency_review.md` |

You may add multiple reusable phases if the specs touch multiple areas. Completeness is always included (most universally valuable). Consistency only triggers with 2+ files (single-document consistency is caught by other phases reading the doc).

### Write the Plan

Write to `reviews/projects/[review_name]/crit_plan.md`:

```markdown
# Design Crit Plan: [review_name]

## Project
- Name: [project_name]
- Path: [/specs/projects/project_name/]

## Spec Files in Scope
- [file path] (status)
- ...

## Phases

- [ ] Phase 1: [Name] — [one-line description]
- [ ] Phase 2: [Name] — [one-line description]
- ...
```

After writing the plan, update the progress block Step 2 sub-steps to match the planned phases.

### Present Plan for Approval

Show the user the plan: project path, spec files in scope, and the list of phases. Ask them to confirm before proceeding:

> Design crit plan ready — [N] phases planned for project `[project_name]`:
>
> [Phase list from crit_plan.md]
>
> Proceed with the review?

Wait for user approval. If they want changes (add/remove phases, change scope), update the plan and re-confirm.

## Step 2: Phase Reviews

For each phase, spawn a fresh sub-agent. Use the appropriate prompt template below depending on whether it's a spec-specific or reusable template phase.

→ Read [references/spawning_subagents.md](spawning_subagents.md) for how to spawn sub-agents.

**After each sub-agent returns:**
1. Update the progress block with the phase result (issue counts by severity)
2. Check the phase off in `crit_plan.md`
3. Immediately spawn the next phase — do not stop for user input

Continue until all phases are complete.

## Step 3: Summary

After all phases complete, read all `reviews/projects/[review_name]/phase_N_feedback.md` files. Write `reviews/projects/[review_name]/crit_summary.md`:

```markdown
# Design Crit Summary: [review_name]

## What This Design Covers

[2-4 sentences describing the project/design scope]

## Key Design Decisions

- [Decision]: [assessment]
- ...

## Quality Assessment

[Overall assessment of spec quality, completeness, clarity]

## Issues Overview

| Severity | Count | Key Themes |
|----------|-------|------------|
| Critical | N | [brief themes] |
| Moderate | N | [brief themes] |
| Mild | N | [brief themes] |

## Critical Issues (Requires Attention)

[List all critical issues across all phases, with spec file references]

## Recommendations

[Top 3-5 prioritized actions]

## Issue Queue

Full deduplicated list of all issues with stable IDs for resolution tracking.

| ID | Severity | Spec File | Issue | Source Phase |
|----|----------|-----------|-------|--------------|
| DC-1 | Critical | architecture.md:Data Flow | No error handling for sync failures | Phase 2 |
| DC-2 | Moderate | functional_spec.md:Auth | Session timeout not specified | Phase 1 |
| ... | ... | ... | ... | ... |
```

**Deduplication:** When multiple phases flag the same underlying issue (e.g., both Completeness and Architecture note missing error handling), merge them into a single entry. Note the source phases.

The Issue Queue with stable IDs (DC-1, DC-2, ...) is the bridge between autonomous review and interactive resolution.

## Step 4: Present

Show the user:
- Link to the review folder: `reviews/projects/[review_name]/`
- The Issues Overview table from the summary
- Total counts: N critical, N moderate, N mild across all phases
- If critical issues exist: list them prominently with spec file references
- Prompt: "Ready to work through issues? Say 'resolve' to start the resolution phase, or review the findings in the review folder first."

## Step 5: Resolution — Interactive

This phase is fully interactive. The user drives all decisions.

### 5a: Load the Issue Queue

Load the Issue Queue from `crit_summary.md`. This is the source of truth for what needs resolution.

### 5b: Present Prioritized Groups

Group issues by theme/area (e.g., "Authentication", "Data Flow", "Error Handling") rather than strictly by severity. Within each group, order by severity (critical first).

Present one group at a time:

```
### Authentication (3 issues)

- **DC-1** (Critical): No session timeout specified
  architecture.md:Session Management — the spec doesn't define when sessions expire
  
- **DC-5** (Moderate): Token refresh flow missing from architecture
  architecture.md:Auth — functional spec mentions refresh tokens but architecture doesn't design the flow
  
- **DC-11** (Mild): Inconsistent terminology — "session" vs "auth token"
  functional_spec.md + architecture.md — same concept called different things

For each: accept (will fix), reject (won't fix), or discuss?
You can also batch: "accept all", "reject DC-11"
```

Offer batch actions:
- "accept all" / "reject all" for the current group
- "accept all mild" across all remaining groups
- Individual accept/reject by ID

### 5c: Process Decisions

For each decision:
- **Accept**: Will be fixed. Note the issue ID and any discussion about how to fix it.
- **Reject**: Won't fix. Capture the rationale (user must provide one, even brief).
- **Discuss**: Manager explains the issue in more detail, explores alternatives. User then accepts or rejects.

### 5d: Write Refinement Files

Create `reviews/projects/[review_name]/refinements/` directory.

**For accepted issues:** Group related accepted issues (same theme/area) into a single refinement file. Each file describes the issues and the specific spec changes needed.

`reviews/projects/[review_name]/refinements/NN_description.md`:

```markdown
# Refinement: [Description]

## Issues Addressed
- DC-1: [title]
- DC-5: [title]

## Changes

### architecture.md — Session Management
[Specific text to add/change/remove]

### architecture.md — Auth
[Specific text to add/change/remove]
```

**For rejected issues:** Append to `reviews/projects/[review_name]/wont_fix.md`:

```markdown
# Won't Fix

| ID | Issue | Rationale |
|----|-------|-----------|
| DC-11 | Inconsistent terminology | Terms are used in different contexts; readers will understand from context |
```

### 5e: Verify Completeness

After all groups are processed, show a tally:

```
All issues addressed: N accepted (in M refinement files), K rejected (in wont_fix.md), 0 remaining.
```

If any issues were skipped or missed, surface them. Every issue from the queue must end up in either a refinement file or wont_fix.md.

### 5f: Fold Refinements into Specs

For each refinement file, apply the changes to the relevant spec files. The changes are specified precisely enough in the refinement files (from step 5d) that this is a straightforward editing task.

After all refinements are applied:
- Show a summary of files modified
- Note that downstream artifact statuses may need cascade (per artifact conventions)

## Autonomous Flow

**Steps 2 through 4 are fully autonomous — drive the entire flow to completion without stopping for user input. No exceptions.** After each sub-agent returns, update the progress block and immediately spawn the next phase. After all phases complete, write the summary and present results.

The manager pauses for user input at three points:
1. **Step 0** — scope confirmation (always)
2. **Step 1** — plan approval (always)
3. **Step 5** — resolution phase (fully interactive, user decides on every issue group)

After the user approves the plan (end of Step 1), Steps 2-4 run without interruption. Step 5 only begins when the user explicitly says "resolve."

## Non-Interactive (Steps 2-4)

Once the review is running (Step 2 onward through Step 4), keep working until all phases are complete, the summary is written, and results are presented. Don't stop to ask questions, don't ask "should I continue?", don't wait for approval between phases. The progress block tells you what to do next — do it.

## Prompt Templates

These are the exact prompts the manager sends to sub-agents. Use them verbatim, filling in the bracketed values.

### Spec-Specific Phase Prompt

```
You are a design review sub-agent performing a focused review of a specific concern area in project specifications.

**Review name:** [review_name]
**Phase:** [N] — [Phase Name]
**Focus:** [one-paragraph description of what to review]
**Spec files to review:** [file list relevant to this phase]
**Project path:** [/specs/projects/project_name/]

Read `references/design_crit_phase_prompt.md` for your full review instructions. Follow them precisely.

Write your findings to `reviews/projects/[review_name]/phase_[N]_feedback.md`.

Return a short summary: phase name, issue counts by severity, notable concerns.
```

### Reusable Template Phase Prompt

```
You are a design review sub-agent performing a focused review of a specific concern area in project specifications.

**Review name:** [review_name]
**Phase:** [N] — [Phase Name]
**Phase resource:** [references/design_crit/phase_X_review.md] — read this for detailed review guidance
**Spec files to review:** [file list relevant to this phase]
**Project path:** [/specs/projects/project_name/]

Read `references/design_crit_phase_prompt.md` for your full review instructions. Follow them precisely.

Write your findings to `reviews/projects/[review_name]/phase_[N]_feedback.md`.

Return a short summary: phase name, issue counts by severity, notable concerns.
```

## References

- [references/spawning_subagents.md](spawning_subagents.md) — How to spawn sub-agents
- [references/design_crit_phase_prompt.md](design_crit_phase_prompt.md) — Full instructions for phase sub-agents
- [references/shared/design_review_standards.md](shared/design_review_standards.md) — Shared design review standards (loaded by sub-agents, not the manager)
