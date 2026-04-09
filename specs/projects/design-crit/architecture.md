---
status: complete
---

# Architecture: Design Crit

This is a prompt/skill project — no application code. Architecture covers file organization, prompt design, and integration with existing skill files.

## File Structure

```
references/
├── cmd_design_crit.md                  # Manager command reference (new)
├── design_crit_phase_prompt.md         # Phase sub-agent prompt template (new)
├── shared/
│   ├── design_review_standards.md      # Design review standards (new)
│   ├── cr_review_standards.md          # Existing — unchanged
│   ├── progress_tracker.md             # Existing — unchanged
│   └── ... (other existing files unchanged)
├── design_crit/
│   ├── phase_completeness_review.md    # Reusable phase resources (new)
│   ├── phase_feasibility_review.md
│   ├── phase_architecture_review.md
│   ├── phase_api_design_review.md
│   ├── phase_data_model_design_review.md
│   ├── phase_security_design_review.md
│   └── phase_consistency_review.md
SKILL.md                                # Modified — add command entry
```

## Component Design

### `cmd_design_crit.md` — Manager Command Reference

Follows the same structure as `cmd_deep_code_review.md`: manager role declaration, progress tracker reference, step-by-step flow, prompt templates.

**Manager role:** Same constraint — never reviews specs, only orchestrates. Exception: the manager writes `crit_plan.md` and `crit_summary.md` directly, and drives the resolution phase interactively.

**Progress tracker label:** `"Design Crit Progress"`

**Step list:**

```
- Step 0: Scope — identify and confirm spec files to review
- Step 1: Plan — design review phases, write crit_plan.md
- Step 2: Phase reviews — spawn sub-agents per phase (sub-steps: 2a, 2b, 2c...)
- Step 3: Summary — read phase feedback files, write crit_summary.md with Issue Queue
- Step 4: Present — show results, offer resolution
- Step 5: Resolution — interactive issue triage and spec fixes (sub-steps: 5a-5f)
```

Step 2 uses sub-labels (2a, 2b, 2c...) matching phase numbers. Step 5 uses sub-labels (5a-5f) for resolution stages.

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

**Interactive vs autonomous boundaries:**
- Step 0: interactive (scope confirmation required)
- Step 1: interactive (plan approval required)
- Steps 2-4: fully autonomous (no user input)
- Step 5: fully interactive (user decisions on every issue group)

**Prompt templates:** Two templates — one for spec-specific phases, one for reusable template phases. Same pattern as deep CR but with spec-relevant fields instead of git-relevant fields.

#### Spec-Specific Phase Prompt Template

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

#### Reusable Template Phase Prompt Template

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

### `design_crit_phase_prompt.md` — Phase Sub-Agent Prompt

Self-contained prompt addressed to the phase sub-agent. Modeled on `deep_cr_phase_prompt.md` but adapted for spec review.

**Key differences from deep CR phase prompt:**

1. **Persona:** Senior architect reviewing a design *before any code is written*, not reviewing existing code. Focus is on whether the design is complete, sound, and implementable.
2. **Context loading:** Reads spec files directly (no `git diff`). Reads full documents, not diffs. Cross-references related specs.
3. **Reference format:** Issues cite `spec_file:section` not `file:line` (prose documents, not code).
4. **Shared standards:** Points to `references/shared/design_review_standards.md` (not `cr_review_standards.md`).
5. **Phase focus scope rule:** Same as deep CR — go deep on assigned concern, but flag critical issues outside your focus if you notice them.

**Sections:**

1. **Role and persona** — Senior architect reviewing a design before implementation. Critical, direct, specific. Evaluates whether the design is complete enough to implement without guesswork. Does not accept hand-waving.
2. **Context loading instructions** — Read all spec files listed in prompt. Read related specs for cross-reference. Load phase resource file if provided. Load shared design review standards.
3. **Phase-specific focus** — Review through the lens of the assigned concern area. Trust other phases for other concerns.
4. **Output format** — Write to specified path using Critical/Moderate/Mild sections with `spec_file:section` references + Phase Summary.
5. **Return format** — Short summary to manager.

### `shared/design_review_standards.md` — Shared Design Review Standards

Analogous to `cr_review_standards.md` but for design review. Referenced by all design crit sub-agents.

**Sections:**

1. **Review Dimensions** — Completeness, Clarity, Consistency, Feasibility, Maintainability (with specific sub-items for each).
2. **Issues to Watch For** — Undefined error behavior, missing state transitions, vague quantifiers, circular dependencies, missing integration points, implicit assumptions, features without design, design without requirements, hand-waving, over-engineering.
3. **Severity Labels** — Critical (implementation failure, redesign required), Moderate (suboptimal implementation, tech debt), Mild (clarity improvement, minor inconsistency).

### `design_crit/phase_*.md` — Reusable Phase Resources

Each file is a self-contained review guide for one design concern area. Addressed to the sub-agent. Same structure as deep CR phase resources:

```markdown
# [Phase Name]

## What to Look For

[Detailed checklist of specific things to review]

## Common Issues

[Patterns and anti-patterns specific to this domain]

## Severity Guidance

[Domain-specific calibration of Critical/Moderate/Mild]
```

**Phase resource contents (summary of each):**

#### `phase_completeness_review.md`
- Requirements traceability: every functional requirement has a corresponding design
- Edge cases: error handling, empty states, boundary conditions defined
- User flows: all paths through the system specified, not just the happy path
- Missing scenarios: what happens on timeout, partial failure, concurrent access, invalid input
- Out-of-scope clarity: is it clear what the system does NOT do
- Common issues: happy-path-only specs, undefined error recovery, missing cleanup/teardown, assumed-but-not-stated prerequisites

#### `phase_feasibility_review.md`
- Technology fit: can each component be built with the stated technology choices
- Hard problem acknowledgment: distributed consensus, real-time sync, conflict resolution, offline-first — are these designed or hand-waved
- Performance assumptions: are stated scale targets realistic with the proposed architecture
- Third-party dependency availability: do the needed libraries/services exist and do what the spec assumes
- Implementation ordering: are there circular dependencies in the build plan
- Common issues: assuming libraries handle complexity they don't, underestimating integration difficulty, ignoring operational complexity (deployment, monitoring, migration)

#### `phase_architecture_review.md`
- Component boundaries: are responsibilities clear and single-purpose
- Coupling: are components appropriately independent, or deeply entangled
- Data flow: is the flow of data between components well-defined and traceable
- Scalability: does the architecture support the stated scale targets without fundamental redesign
- Failure modes: what happens when a component fails — cascading failures, recovery strategies
- Extension points: where variability is expected, is the design ready for it
- Common issues: god components, leaky abstractions, distributed monolith, missing observability design

#### `phase_api_design_review.md`
- Interface completeness: all needed operations present for each consumer
- Naming consistency: resource naming, field naming, action naming across all APIs
- Error contract: error response format, error codes, client guidance
- Versioning strategy: how will breaking changes be managed
- Input/output validation: what are the constraints, who enforces them
- Idempotency: are mutating operations safe to retry
- Common issues: CRUD-only thinking (missing domain operations), inconsistent naming across services, missing pagination, no error schema

#### `phase_data_model_design_review.md`
- Entity completeness: all entities from the functional spec are represented
- Relationship correctness: cardinality, ownership, cascade behavior
- Query pattern support: does the model support the read patterns described in the spec
- Data lifecycle: creation, update, archival, deletion — all addressed
- Migration and evolution: how does the model change over time, what's the migration strategy
- Common issues: missing audit trails, no soft-delete strategy when needed, denormalization without justification, no consideration of data volume growth

#### `phase_security_design_review.md`
- Trust boundaries: where are they, what crosses them, what validation happens at each
- Authentication model: how are users/services identified, token lifecycle, session management
- Authorization model: permission structure, enforcement points, principle of least privilege
- Data classification: what is sensitive, how is it protected at rest and in transit
- Threat scenarios: what could go wrong, what's the blast radius
- Common issues: auth at the edge only (no defense in depth), overly broad permissions, sensitive data in logs/URLs, missing rate limiting design

#### `phase_consistency_review.md`
- Cross-document term consistency: same concept same name everywhere
- Feature parity: everything in functional spec has a design, nothing in architecture lacks a requirement
- Contradictory statements: spec says X in one place, Y in another
- Broken internal references: sections reference other sections or files that don't exist or say something different
- Data flow agreement: producer and consumer specs agree on format, frequency, error handling
- Common issues: renamed concepts (name changed partway through spec writing, old name lingers), feature scope drift between spec and architecture, inconsistent constraint values

### `SKILL.md` — Modification

Add a command entry after `/spec deep cr`:

```markdown
### `/spec design crit` or `/spec crit`

Multi-phase agentic review of spec/design documents. Identifies design concerns, runs focused sub-agents per concern area, produces persistent review artifacts, and offers an interactive resolution phase to triage and fix issues found.

→ Read [design crit command reference](references/cmd_design_crit.md)
```

## Integration Notes

- **No state tracking**: Design crit does not write to `.specs_skill_state/current_project.md`. It reads it (for scope identification) but never modifies it.
- **No continue integration**: `/spec continue` and the router do not reference design crit state. The resolution phase is entered from within the same command session.
- **reviews/ directory**: Created on first use by the manager. The setup command already gitignores `reviews/` (added during deep CR project).
- **Artifact naming**: Uses `crit_plan.md` and `crit_summary.md` (not `cr_plan.md` / `cr_summary.md`) to distinguish from code review artifacts. Both can coexist in the same review folder if a project gets both a code review and a design crit.
- **Shared standards separation**: `design_review_standards.md` is fully separate from `cr_review_standards.md`. Design review has different dimensions (completeness, clarity, consistency, feasibility, maintainability) vs code review (spec compliance, code quality, consistency, project standards). No shared base — they serve different purposes.
