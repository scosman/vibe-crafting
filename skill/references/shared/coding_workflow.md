# Coding Workflow

Shared workflow steps for coding agents across all modes.

## Implementation Steps

1. **Build the code** per the plan
2. **Run automated checks** (lint, format, type-check, build). Follow project-specific commands from system prompt. Iterate until clean.
3. **Write tests** per the plan's test expectations
4. **Run tests**. Iterate until passing.
5. **Run automated checks again** (tests/fixes may introduce lint/format issues). Iterate until clean.
6. **Return summary** — describe what you built. You are now ready for code review.

Do NOT spawn CR sub-agents or commit changes. The manager handles code review and will tell you when to commit.

## CR Feedback Invocation: Address Review

The manager resumes you with CR feedback after a reviewer found issues.

1. Read the feedback provided in the `<cr_feedback>` block in your prompt
2. Address each issue: fix the code, or if there's a strong technical reason not to, add a code comment explaining the rationale
3. Run automated checks (lint, format, type-check, build). Iterate until clean.
4. Run tests. Iterate until passing.
5. **Return summary** — describe the changes you made. You are now ready for re-review.

## Non-Interactive

Work autonomously. Don't ask the user for help.

**One exception:** You discover a genuinely new technical constraint not known at design time that materially changes the plan (e.g., an API doesn't support an assumed operation, a framework has an undocumented limitation).

In this case — and only this case — return a clear roadblock message instead of a completion summary. Describe the constraint, why it matters, and what decision is needed. The manager will escalate to the user and resume you with their decision.

## Completion

What "done" means depends on your invocation mode:

- **Initial invocation**: Return a summary of what you built. The manager will initiate code review.
- **CR feedback invocation**: Return a summary of changes made. The manager will initiate re-review.
- **Commit invocation**: Return the commit message you used. The manager will verify the commit. If the commit fails due to pre-commit hooks, do NOT bypass the hooks. Return a summary explaining what failed with an attestation block showing current state. The manager will route you back through review.

In all modes, your final message is a short summary — not a question, not a request for input.

## Attestation Block

Every return summary MUST end with this block:

```
<attestation>
- Checks pass: TRUE/FALSE/NA
- Tests pass: TRUE/FALSE/NA
- Tests written: TRUE/FALSE/NA
</attestation>
```

Rules:
- **Checks pass** means ALL project automated checks (lint, format, type-check, build, etc.) ran successfully.
- **Tests pass** means all tests ran and passed.
- **Tests written** means you wrote new tests for the functionality you built. NA only if the project has no test infrastructure or the change genuinely doesn't warrant tests (e.g., config-only change).
- If any value is FALSE, explain what's failing and why in your summary above the block.
- Do not bypass failing checks. Do not skip checks to save time.
