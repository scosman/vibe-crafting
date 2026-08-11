# UI Review

The human review step for user-visible changes. Shared by `/spec implement`, `/spec task`, and `/spec pr` — your command file says where it sits in the step list and what step number it gets.

## Why It Runs After Commit

1. **The agent is often not on the user's machine.** Containers, remote sandboxes, and worktrees mean the user frequently can't see your working tree at all. Committing — and pushing, where the command pushes — is what makes the change reviewable. They sync, then look.
2. **Code review covers code quality, not design.** A clean CR is not a signoff on how something looks or feels. That's a separate human judgment, and it belongs at the end of the flow rather than as a gate in the middle of it.

Committing UI on a clean CR alone is intentional. If the user wants changes, that's a follow-up commit — not a reason to have held the first one.

## When It Applies

Run it when the changes are **significant and user-visible**: new screens, layout changes, new components, visual redesigns, changed flows.

Skip it when:

- The coding agent returned `<ui_review>NONE</ui_review>`
- Nothing user-visible changed (backend work, refactors, tooling, tests)
- The changes are trivial (copy tweaks, minor spacing)

When you skip it, mark the step `skipped` in the progress block with the reason.

## What to Send the User

Build the message from the coding agent's `<ui_review>` block — don't rewrite its bullets or pad them out.

> **Ready for UI review**
>
> [One or two sentences on what was built.]
>
> These changes are committed[ and pushed] — sync your working copy to see them.
>
> **What to check:**
> - [bullets from the `<ui_review>` block]
>
> Look good, or what should change?

## The Feedback Loop

- **User approves** (or has no changes): continue to the next step in your command's list.
- **User requests changes**: resume the coding agent with the UI Feedback Prompt below, then re-enter the flow at attestation → CR loop → commit → verify, and return here for another review. Increment the round counter on those steps.

**UI fixes are code changes like any other.** They go through a clean CR before they get committed. Never commit straight off UI feedback.

## Consolidated Review (`implement all`)

`/spec implement all` does **not** run this step per phase — the `all` directive means the run does not stop for it. Instead:

1. Keep each phase's `<ui_review>` block as you go
2. After the last phase commits and verifies, present **one** review covering the whole run, bullets grouped under their phase
3. Drop phases whose block was `NONE`. If every phase was `NONE`, skip the step entirely.

For feedback on a consolidated review, spawn a **fresh** coding agent with the UI Feedback Coding Prompt below rather than resuming a phase agent — the feedback may span several phases, and the relevant agent may be many phases back.

## Prompt Templates

### UI Feedback Prompt (resume coding agent)

```
The user reviewed the UI and asked for changes. Address the feedback below, then run automated checks and tests until clean.

Return a short summary of the changes you made when ready for re-review.

<ui_feedback>
[User's feedback — verbatim where possible]
</ui_feedback>
```

### UI Feedback Coding Prompt (fresh spawn, after a consolidated review)

```
You are a coding agent addressing UI feedback on work that has already been built and committed.

**Project specs:** [specs/projects/PROJECT_NAME/]

Read `references/coding_phase_prompt.md` for your full instructions. Follow them precisely — but note that the phase plans are already written and the phases are already complete. Your job is only the UI feedback below.

<ui_feedback>
[User's feedback — verbatim where possible]
</ui_feedback>

Return a short summary of the changes you made when ready for code review.
```
