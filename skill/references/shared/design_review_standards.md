# Design Review Standards

These are the shared review standards applied by all design crit sub-agents — reusable template phases and spec-specific phases alike. Apply all of them to every review.

## Review Dimensions

### 1. Completeness

Is the design fully specified? Could an engineer implement it without guessing?

- Every functional requirement has a corresponding design
- Edge cases are defined: error handling, empty states, boundary conditions
- All user/system flows are specified, not just the happy path
- Failure scenarios are addressed: timeouts, partial failures, concurrent access, invalid input
- Out-of-scope boundaries are explicit — clear what the system does NOT do
- Prerequisites and dependencies are stated, not assumed

### 2. Clarity

Is the spec unambiguous? Could two engineers read it and build the same thing?

- Concrete and specific — no hand-waving ("the system will handle X appropriately")
- Quantified where it matters — not "handles large volumes" but "supports 10k concurrent users"
- Terms are defined on first use, especially domain-specific ones
- Behavioral descriptions are precise enough to derive test cases from
- Decision rationale is included for non-obvious choices

### 3. Consistency

Do the spec files agree with each other?

- Same concept uses the same name everywhere (no "session" in one file and "auth token" in another for the same thing)
- Data flows match: producer and consumer specs agree on format, frequency, error handling
- Every feature in the functional spec has a design in the architecture
- Nothing in the architecture lacks a corresponding requirement
- Constraint values (timeouts, limits, thresholds) are consistent across documents
- Referenced sections and files exist and say what the referring document claims

### 4. Feasibility

Can this actually be built with the stated approach?

- Technology choices support the stated requirements
- Hard problems are designed, not deferred — distributed consensus, real-time sync, conflict resolution, offline-first
- Performance assumptions are realistic for the proposed architecture and stated scale targets
- Third-party dependencies exist and do what the spec assumes they do
- Implementation ordering is achievable — no circular dependencies in the build plan
- Operational complexity is acknowledged — deployment, monitoring, migration, rollback

### 5. Maintainability

Will this design age well?

- Component responsibilities are clear and single-purpose
- Coupling between components is proportional to their actual dependency
- Complexity is proportional to the problem — not over-engineered
- Extension points exist where variability is expected
- The design can evolve without requiring full redesign for likely changes

## Issues to Watch For

Beyond the review dimensions above, actively look for these specific patterns:

- **Undefined error behavior** — The spec describes the happy path but not what happens when things go wrong
- **Missing state transitions** — States are named but transitions between them are incomplete or undefined
- **Vague quantifiers** — "Handles large volumes," "responds quickly," "scales well" without concrete numbers
- **Circular dependencies** — Component A depends on B depends on C depends on A
- **Missing integration points** — Two systems need to talk but no interface is defined between them
- **Implicit assumptions** — The design only works if something unstated is true (e.g., "users will always have network connectivity")
- **Features without design** — Functional spec describes a feature, architecture doesn't design it
- **Design without requirements** — Architecture includes something the functional spec never asked for
- **Hand-waving** — "The system will handle X appropriately" or "standard approach applies here" without specifying what that means
- **Over-engineering** — Abstraction layers, plugin systems, or extensibility points for problems that don't exist yet and may never exist

## Severity Labels

Each issue gets one:

- **Critical** — Design gap that would cause implementation failure, data loss, security vulnerability, or require mid-implementation redesign. Must be resolved before building.
- **Moderate** — Design weakness that would cause suboptimal implementation, accumulate tech debt, or confuse the builder. Should be resolved but won't block implementation.
- **Mild** — Clarity improvement, minor inconsistency, or nice-to-have detail. Consider fixing for a cleaner spec.
