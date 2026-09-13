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

## Proportionality

Finish the work. That is the job — not maximising confidence in it.

Match verification effort to the risk in the code you wrote. Heavier techniques are worth reaching for when the code is subtle, the blast radius is wide, or a defect would be silent. They are not worth reaching for because the last phase used them: **verification depth is set by the risk in front of you, not by precedent.** If this piece of work needs more than the last one did, say why in your summary.

**Tests are a deliverable. Reports about testing are not.** Write tests that catch real breakage and let them ship. Do not write up what you ran, how the harness worked, or what each technique found — that prose has no consumer, and it becomes the next review's surface area.

## CR Feedback Invocation: Address Review

The manager resumes you with CR feedback after a reviewer found issues.

1. Read the feedback provided in the `<cr_feedback>` block in your prompt
2. Address each issue: fix the code, or if there's a strong technical reason not to, add a code comment explaining the rationale
3. Address what the feedback asks for and no more. Do not widen the change, refactor adjacent code, or add capability while you are in there. If a fix seems to require going outside the scope of your phase or task, say so in your summary rather than doing it.
4. Run automated checks (lint, format, type-check, build). Iterate until clean.
5. Run tests. Iterate until passing.
6. **Return summary** — describe the changes you made. You are now ready for re-review.

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

## UI Review Block

Every return summary MUST also end with a `<ui_review>` block, directly after the attestation. The manager uses it to tell the user what to look at once the work is committed.

```
<ui_review>
- Settings → Notifications — new toggle row, check spacing against the rows above it
- Alarm list empty state — new illustration and copy
</ui_review>
```

Rules:
- **1–5 bullets, one line each.** Terse. A pointer to where to look and what to judge — not an explanation of what you built.
- Only things a human has to *see* to judge: layout, visual design, flow, copy in context. Not logic, not tests, not internals.
- If nothing user-visible changed, the whole block is `<ui_review>NONE</ui_review>`.
- Don't pad to fill the five. Two sharp bullets beat five vague ones.
