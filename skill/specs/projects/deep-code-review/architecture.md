---
status: complete
---

# Architecture: Deep Code Review

This is a prompt/skill project — no application code. Architecture covers file organization, prompt design, and integration with existing skill files.

## File Structure

```
references/
├── cmd_deep_code_review.md          # Manager command reference (new)
├── deep_cr_phase_prompt.md          # Phase sub-agent prompt template (new)
├── cr_agent_prompt.md               # Existing — modified to reference shared standards
├── shared/
│   ├── cr_review_standards.md       # Extracted from cr_agent_prompt.md (new)
│   └── ... (existing files unchanged)
├── deep_cr/
│   ├── phase_ui_review.md           # Reusable phase resources (new)
│   ├── phase_api_review.md
│   ├── phase_data_model_review.md
│   ├── phase_security_review.md
│   ├── phase_test_review.md
│   ├── phase_dependency_review.md
│   └── phase_performance_review.md
SKILL.md                             # Modified — add command entry
```

## Component Design

### `cmd_deep_code_review.md` — Manager Command Reference

Follows the same structure as `cmd_implement.md`: manager role declaration, progress tracker reference, step-by-step flow, prompt templates.

**Manager role:** Same constraint as implement — never reviews code, only orchestrates. The one exception to the "read-only" rule: the manager writes `cr_plan.md` and `cr_summary.md` directly (not delegated to sub-agents).

**Progress tracker label:** `"Deep CR Progress"`

**Step list:**

```
- Step 0: Context & diff — loading git context, spec context, determining base
- Step 1: Plan — designing review phases, writing cr_plan.md
- Step 2: Phase reviews — spawning sub-agents per phase (sub-steps: 2a, 2b, 2c...)
- Step 3: Summary — manager reads phase feedback files, writes cr_summary.md
- Step 4: Present — show results to user
```

Step 2 uses sub-labels (2a, 2b, 2c...) matching phase numbers so the progress block shows which phase is in progress. Example:

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

**Prompt templates:** Two templates — one for project-specific phases, one for reusable template phases. The only difference is whether the phase description is inline or a pointer to a resource file.

#### Project-Specific Phase Prompt Template

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

#### Reusable Template Phase Prompt Template

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

### `deep_cr_phase_prompt.md` — Phase Sub-Agent Prompt

Self-contained prompt addressed to the phase sub-agent (same pattern as `cr_agent_prompt.md`). Contains:

1. **Role and persona** — Senior architect who will own this code long-term. Critical, direct, specific. "The spec said to" is not a reason to implement inferior code.
2. **Context loading instructions** — How to read the diff (`git diff <base>...HEAD -- <files>`), how to load spec context if provided, how to load phase resource file if provided.
3. **Shared standards reference** — `→ Read references/shared/cr_review_standards.md` for review dimensions, issues to watch for, severity labels.
4. **Phase-specific focus** — Review through the lens of the phase description/resource, not a generic full review. Other phases cover other concerns.
5. **Output format** — Write to the specified `phase_N_feedback.md` path using the format from the functional spec (Critical/Moderate/Mild sections + Phase Summary).
6. **Return format** — Short summary to manager: phase name, counts, notable concerns.

**Key difference from `cr_agent_prompt.md`:** The existing CR prompt covers the full change in one pass. The deep CR phase prompt is scoped — it reviews through a specific lens and trusts other phases to cover other concerns. It should still flag critical issues outside its focus area if it notices them, but its primary job is depth on its assigned concern.

### `shared/cr_review_standards.md` — Shared Review Standards

Extracted content from `cr_agent_prompt.md`. Referenced by both existing CR and deep CR.

**Sections:**

1. **Review Dimensions** — Spec/task compliance, code quality, consistency, project-specific standards. (Moved verbatim from `cr_agent_prompt.md` sections 1-4.)
2. **Issues to Watch For** — The combined list from functional spec (bugs, poor names, wrong-place code, globals, json.dumps, GPL deps, dead code, inconsistent error handling, missing validation, hardcoded values).
3. **Severity Labels** — Critical/Moderate/Mild definitions. (Moved verbatim from `cr_agent_prompt.md`.)

### `cr_agent_prompt.md` — Modification

Replace the inline "Review Dimensions," "Severity Labels," and add the issues-to-watch-for by inserting a reference:

```
→ Read [references/shared/cr_review_standards.md](shared/cr_review_standards.md) for review dimensions, issues to watch for, and severity labels. Apply all of them.
```

Keep the CR-specific content in place: role/persona, context loading (project vs task mode), create a review plan, output format, re-review protocol. These are process-specific to single-pass CR and don't belong in the shared file.

### `deep_cr/phase_*.md` — Reusable Phase Resources

Each file is a self-contained review guide for one concern area. Addressed to the sub-agent. Structure:

```markdown
# [Phase Name] Review

## What to Look For

[Detailed checklist of specific things to review for this concern area]

## Common Issues

[Patterns and anti-patterns specific to this domain]

## Severity Guidance

[When something in this domain is critical vs moderate vs mild — domain-specific calibration]
```

The manager never reads these files. Only the sub-agent does, when told to by the prompt template.

**Phase resource contents (summary of each):**

#### `phase_ui_review.md`
- Visual hierarchy and layout consistency
- Accessibility (ARIA, keyboard nav, contrast, screen readers)
- Responsive design and breakpoints
- String quality: typos, spelling, grammar, tone consistency
- Loading states, empty states, error states
- Consistent use of design system / component library
- Form validation UX (inline errors, field focus, submit states)
- Navigation patterns and information architecture

#### `phase_api_review.md`
- REST conventions: resource naming, HTTP methods, status codes
- Request/response schema design (consistency, naming, nesting)
- Pagination, filtering, sorting patterns
- Versioning strategy
- Error response format and consistency
- Authentication/authorization on endpoints
- Rate limiting and input size limits
- Idempotency for mutating operations
- Documentation/OpenAPI alignment

#### `phase_data_model_review.md`
- Schema design: normalization, appropriate denormalization
- Index coverage for query patterns
- Migration safety: backwards compatibility, data loss risk, rollback plan
- Constraints: foreign keys, NOT NULL, unique, check constraints
- Naming conventions for tables, columns, indexes
- Soft delete vs hard delete patterns
- Temporal data handling (timestamps, timezones)
- Enum/status field design (extensibility)

#### `phase_security_review.md`
- Authentication and authorization logic
- Input validation and sanitization
- SQL injection, XSS, CSRF, command injection vectors
- Secrets management (no hardcoded secrets, proper env var usage)
- Dependency vulnerabilities (known CVEs)
- CORS and CSP configuration
- Sensitive data exposure (logging, error messages, API responses)
- Cryptographic choices (algorithms, key sizes, proper usage)

#### `phase_test_review.md`
- Test coverage of new/changed code paths
- Test quality: do tests verify behavior or just exercise code?
- Brittle tests: tests that break on refactor without behavior change
- Missing edge case tests
- Test isolation: no shared mutable state between tests
- Mock quality: mocking at correct boundary, not over-mocking
- Test naming and readability
- Integration vs unit test balance

#### `phase_dependency_review.md`
- License compatibility (GPL/copyleft = critical)
- Dependency reputation and maintenance status
- Version pinning strategy
- Transitive dependency bloat
- Duplicate functionality (new dep overlaps existing)
- Security advisories on added versions
- Size impact (especially frontend bundles)

#### `phase_performance_review.md`
- Algorithm complexity for data scale (O(n^2) on large datasets, unnecessary full-table scans)
- Database query patterns: N+1 queries, missing indexes for query patterns at scale, unbounded result sets
- Memory usage: loading large datasets into memory, unbounded caches, large object allocation in loops
- Concurrency: connection pool sizing, lock contention, async/await correctness
- Network: chatty service-to-service calls, missing batching, no retry/backoff
- Caching strategy: appropriate cache levels, invalidation correctness, cache stampede risk
- Applies model's world knowledge about performance characteristics of specific libraries, databases, and cloud services being used

### `SKILL.md` — Modification

Add a command entry between `/spec cr` and the Router section:

```markdown
### `/spec deep cr` or `/spec deep review`

Multi-phase agentic code review. Compares current branch against its fork point, designs review phases tailored to the diff, runs focused sub-agents per phase, and produces persistent review artifacts.

→ Read [deep code review command reference](references/cmd_deep_code_review.md)
```

### `cmd_setup.md` — Modification

In Step 1 (Gitignore), add `reviews/` alongside `.specs_skill_state/`:

> Add `.specs_skill_state/` and `reviews/` to `.gitignore`

Same check-and-append logic for both entries.

## Integration Notes

- **No state tracking**: Deep CR does not write to `.specs_skill_state/current_project.md`. It reads it (for spec context) but never modifies it.
- **No continue integration**: `/spec continue` and the router do not reference deep CR state.
- **reviews/ directory**: Created on first use by the manager (no setup prerequisite, but setup adds the gitignore entry).
- **Autonomous flow**: Like `/spec implement`, once Step 1 begins the manager drives to completion. No user interaction except the fork-point confirmation edge case in Step 0.
