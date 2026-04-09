---
status: complete
---

# Functional Spec: Design Crit

## Overview

`/spec design crit` (alias: `/spec crit`) is a standalone command that runs a thorough, multi-phase agentic review of spec/design documents. The manager orchestrates focused sub-agents, each examining a different design concern. All output is written to persistent files under `reviews/projects/[review_name]/`.

After review phases complete, an interactive resolution phase lets the user triage found issues and fold accepted fixes back into the spec files.

## Command Invocation

```
/spec design crit                        # review active project's specs
/spec design crit project_name           # review a specific project's specs
/spec crit                               # alias
/spec crit specs/projects/my_project     # explicit path
```

## Determining What to Review (Step 0)

Unlike deep CR which uses `git diff`, design crit reviews spec files directly.

**Algorithm:**
1. If a specific project or path is provided as an argument, use that
2. Otherwise, read `.specs_skill_state/current_project.md` to find the active project
3. Scan the project directory for spec artifacts: `project_overview.md`, `functional_spec.md`, `ui_design.md`, `architecture.md`, `components/*.md`, `implementation_plan.md`
4. Read each file's frontmatter to get its status (draft/complete)
5. Present the identified scope to the user with file list and statuses
6. **Do NOT proceed without explicit user confirmation of scope** — this is non-negotiable

If no active project exists and no path was provided, tell the user and suggest they either specify a path or set an active project.

**Scope confirmation prompt:**

```
Design crit scope for project `[name]`:

- `functional_spec.md` (complete)
- `architecture.md` (complete)
- `components/auth.md` (complete)
- `components/data_pipeline.md` (draft)
- `implementation_plan.md` (complete)

Review all of these, or a subset?
```

The user may narrow scope (e.g., "just architecture and components") or confirm all. Adjust accordingly.

### Review Name Generation

- Use the project name with `-crit` suffix (slug format): e.g., `my-project-crit`
- Check if `reviews/projects/[name]` already exists. If so, append `-v2`, `-v3`, etc.
- Create the directory: `reviews/projects/[review_name]/`

## Phase Planning (Step 1)

The manager designs a review plan with two categories of phases:

### Spec-Specific Phases

Custom phases tailored to the actual spec content. The manager reads the spec files to identify distinct concern areas worth separate, focused review. Examples:
- "Payment flow design review" (if the spec includes payment handling)
- "Multi-tenancy design review" (if the spec describes tenant isolation)
- "Migration strategy review" (if there's an upgrade/migration path)
- "Concurrency model review" (if the spec describes concurrent operations)

Each spec-specific phase gets a one-paragraph description of what to focus on, plus the list of spec files relevant to that phase.

### Reusable Template Phases

Standard phases included when their trigger condition is met. The manager only needs to evaluate the trigger — the detailed review instructions live in resource files that the sub-agent loads directly.

| Phase | Trigger | Resource File |
|-------|---------|---------------|
| Completeness & Coverage | Always (when any spec file beyond project_overview is in scope) | `references/design_crit/phase_completeness_review.md` |
| Technical Feasibility | Architecture or component specs are in scope | `references/design_crit/phase_feasibility_review.md` |
| Architecture Review | Architecture or component specs are in scope | `references/design_crit/phase_architecture_review.md` |
| API/Contract Design | Spec describes APIs, endpoints, interfaces, or service contracts | `references/design_crit/phase_api_design_review.md` |
| Data Model Design | Spec describes data models, schemas, or storage design | `references/design_crit/phase_data_model_design_review.md` |
| Security Design | Spec involves auth, user data, external integrations, or trust boundaries | `references/design_crit/phase_security_design_review.md` |
| Spec Consistency | Two or more spec files are in scope | `references/design_crit/phase_consistency_review.md` |

The manager may add multiple reusable phases if the specs touch multiple areas. Completeness is always included (most universally valuable). Consistency only triggers with 2+ files (single-document consistency is caught by other phases reading the doc).

### Plan Output

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

Present plan for user approval before proceeding.

## Phase Execution (Step 2)

For each phase, spawn a fresh sub-agent. The sub-agent receives:

**Input (passed in prompt):**
- The review name and phase number
- The list of spec files to review for this phase
- The phase description (one paragraph for spec-specific, or a pointer to the resource file for reusable phases)
- The project path
- Pointer to shared design review standards: `references/shared/design_review_standards.md`

**Sub-agent responsibilities:**
1. Load shared design review standards from `references/shared/design_review_standards.md`
2. If reusable phase: load the phase-specific resource file
3. Read the spec files fully (not just summaries — the complete documents)
4. Read related specs for cross-reference context
5. Review with the persona of a senior architect reviewing a design before implementation
6. Write findings to `reviews/projects/[review_name]/phase_N_feedback.md`

**Output file format (`phase_N_feedback.md`):**

```markdown
# Phase N: [Phase Name]

## Critical (must fix)

- [spec_file:section] **[Issue title]**
  [Description of the problem and why it matters]
  [Suggested improvement if applicable]

## Moderate (should fix)

[Same format]

## Mild (consider fixing)

[Same format]

## Phase Summary

[2-3 sentences: what was reviewed, overall assessment of this concern area]
```

References use `spec_file:section` format (not `file:line` since these are prose documents, not code).

**Sub-agent return to manager:** A short summary — phase name, count of issues by severity, any notable concerns.

## Continuation Without Input (Steps 2-3)

The manager drives all phases to completion without stopping for user input. After each sub-agent returns, update the progress block and immediately spawn the next phase. No exceptions.

## Summary Generation (Step 3)

After all phases complete, the manager reads all `phase_N_feedback.md` files and generates `reviews/projects/[review_name]/crit_summary.md`:

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

## Final Output (Step 4)

The manager presents to the user:
- Link to the review folder: `reviews/projects/[review_name]/`
- The Issues Overview table from the summary
- Total counts: N critical, N moderate, N mild across all phases
- If critical issues exist: list them prominently with spec file references
- Prompt: "Ready to work through issues? Say 'resolve' to start the resolution phase, or review the findings in the review folder first."

## Resolution Phase (Step 5) — Interactive

This is the distinguishing feature vs deep CR. It is fully interactive.

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

For each refinement file, the manager applies the changes to the relevant spec files. The changes are specified precisely enough in the refinement files (from step 5d) that this is a straightforward editing task.

After all refinements are applied:
- Show a summary of files modified
- Note that downstream artifact statuses may need cascade (per artifact conventions)

## Shared Design Review Standards

A new shared standards file at `references/shared/design_review_standards.md`, referenced by all design crit sub-agents. Analogous to `references/shared/cr_review_standards.md` for code review.

**Review dimensions:**
- Completeness: requirements fully specified, edge cases covered, error scenarios defined
- Clarity: unambiguous, concrete, implementable without guessing
- Consistency: spec files agree, terms used consistently, data flows match
- Feasibility: implementable with stated technology, hard problems acknowledged
- Maintainability: low coupling, proportional complexity, extension points where needed

**Issues to watch for:**
- Undefined error behavior
- Missing state transitions
- Vague quantifiers ("handles large volumes" without numbers)
- Circular dependencies
- Missing integration points
- Implicit assumptions
- Features without design (functional spec describes it, architecture doesn't)
- Design without requirements (architecture includes something not in functional spec)
- Hand-waving ("the system will handle X appropriately")
- Over-engineering

**Severity labels:**
- Critical: design gap causing implementation failure, data loss, security vulnerability, or mid-implementation redesign
- Moderate: design weakness causing suboptimal implementation, tech debt, or builder confusion
- Mild: clarity improvement, minor inconsistency, nice-to-have detail

## Files Created/Modified

### New files
| File | Purpose |
|------|---------|
| `references/cmd_design_crit.md` | Manager command reference |
| `references/design_crit_phase_prompt.md` | Base prompt template for phase sub-agents |
| `references/shared/design_review_standards.md` | Shared design review standards |
| `references/design_crit/phase_completeness_review.md` | Completeness & coverage phase resource |
| `references/design_crit/phase_feasibility_review.md` | Technical feasibility phase resource |
| `references/design_crit/phase_architecture_review.md` | Architecture review phase resource |
| `references/design_crit/phase_api_design_review.md` | API/contract design phase resource |
| `references/design_crit/phase_data_model_design_review.md` | Data model design phase resource |
| `references/design_crit/phase_security_design_review.md` | Security design phase resource |
| `references/design_crit/phase_consistency_review.md` | Spec consistency phase resource |

### Modified files
| File | Change |
|------|--------|
| `SKILL.md` | Add `/spec design crit` command entry |

## Out of Scope

- Modifying code (this reviews specs, not code)
- Reviewing code against specs (that's `/spec cr` or `/spec deep cr`)
- Creating new spec files (resolution folds changes into existing specs)
- Automated quality checks (no linting or validation tooling)
- GitHub/PR integration
