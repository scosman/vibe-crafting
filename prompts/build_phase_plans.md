For each phase in the implementation plan, write a detailed phase plan in `specs/phase_plans/phase_N.md`.

Read `specs/implementation_plan.md` and the architecture doc for context. Each phase plan should be a self-contained reference — the coding agent should be able to implement the phase from this file alone, without going back to the full architecture doc.

### Structure

**Overview** — A short summary of what this phase accomplishes and why. Reference the relevant spec file.

**Steps** — Break the phase into ordered steps. For each step:
- Specify the file(s) to change
- Describe the specific changes to make (new functions, renamed fields, updated signatures, etc.)
- Copy relevant details from the architecture doc and expand on them — don't just reference the doc, pull the information into the plan so it stands alone
- Include code snippets where they clarify intent (signatures, key logic, data structures)

**Tests** — Describe the testing strategy:
- List specific automated test cases by name and what they verify
- Group tests by file / component
- Note if manual testing is required (UI, system integrations, permissions) and what the manual test plan should cover

**Completion Criteria** — A checklist of everything that should be true when the phase is done. Include code changes, tests passing, and any tooling checks (lint, format, types).

### Guidelines

- Each step should be small enough to verify independently
- Be specific about what changes — "update the key construction" is too vague, "rename `params_path` to `params_hash` in `AssetKey`" is right. Include class interfaces and function signatures.
- Include edge cases and error handling in both the steps and test cases
- If a step touches existing tests, call that out explicitly
