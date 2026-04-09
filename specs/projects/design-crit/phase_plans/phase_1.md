---
status: draft
---

# Phase 1: Shared Standards and Phase Sub-Agent Prompt

## Overview

Create the two foundational files that all design crit sub-agents reference: `references/shared/design_review_standards.md` (the shared review dimensions and severity labels) and `references/design_crit_phase_prompt.md` (the self-contained prompt template for phase sub-agents). These must exist before any phase resource files or the manager command reference can be built.

## Steps

1. Create `references/shared/design_review_standards.md` with:
   - Review Dimensions section (Completeness, Clarity, Consistency, Feasibility, Maintainability) with specific sub-items per the functional spec
   - Issues to Watch For section (undefined error behavior, missing state transitions, vague quantifiers, circular dependencies, missing integration points, implicit assumptions, features without design, design without requirements, hand-waving, over-engineering)
   - Severity Labels section (Critical, Moderate, Mild) with design-specific calibration per the functional spec
   - Follow the structural pattern of `references/shared/cr_review_standards.md` but with entirely different content appropriate for design review

2. Create `references/design_crit_phase_prompt.md` with:
   - Role and persona section: senior architect reviewing a design before implementation
   - Context loading instructions: read spec files, cross-reference related specs, load phase resource file if provided, load shared design review standards
   - Phase-specific focus section: go deep on assigned concern, flag critical issues outside focus
   - Output format section: Critical/Moderate/Mild with `spec_file:section` references + Phase Summary
   - Return format section: short summary to manager
   - Follow the structural pattern of `references/deep_cr_phase_prompt.md` but adapted for spec review (no git diff, prose references instead of file:line)

## Tests

- NA (markdown/prompt-writing project, no automated tests)
