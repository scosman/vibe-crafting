# Code Review Standards

These are the shared review standards applied by all code review agents — single-pass CR and deep CR phase sub-agents alike. Apply all of them to every review.

## Review Dimensions

### 1. Spec/Task Compliance

Does the implementation match what was asked for?

**Project mode:** Review against spec artifacts (functional spec, architecture, phase plan).
**Task mode:** Review against the task file's `## Request` and `## Notes` sections.

- Missing features or requested changes
- Wrong behavior
- Incomplete edge case handling
- Unimplemented requirements

### 2. Code Quality

- Architecture: Does the code structure make sense?
- Naming: Are variables, functions, classes named clearly?
- Composition: Is the code well-composed, not tangled?
- Error handling: Are errors handled properly?
- Test quality: Do tests cover meaningful cases? Are they brittle?

### 3. Consistency

Does new code match existing patterns and conventions in the codebase?

### 4. Project-Specific Standards

Follow any code review guidelines defined in the project's system prompt config (CLAUDE.md, AGENTS.md, etc.).

## Issues to Watch For

Beyond the review dimensions above, actively look for these specific patterns:

- **Bugs** — Code that doesn't do what it claims to do
- **Poor names** — Function or class names that don't represent their purpose
- **Code in the wrong place** — Logic in a class/file where it doesn't belong
- **Editing globals** — Rarely a good idea; singletons must be clearly labeled; never set globals on external libs unless this is an application (not a library)
- **Python `json.dumps`** — Should always use `ensure_ascii=False`
- **GPL or copyleft dependencies** — Immediate critical failure
- **Dead code or unused imports** introduced by the change
- **Inconsistent error handling patterns**
- **Missing input validation** at system boundaries
- **Hardcoded values** that should be configurable

## Severity Labels

Each issue gets one:

- **Critical** — Must fix before merging. Breaking change, security issue, major bug, spec violation.
- **Moderate** — Should fix. Code smell, maintainability issue, minor bug, unclear behavior.
- **Mild** — Consider fixing. Nit, style inconsistency, minor improvement opportunity.
