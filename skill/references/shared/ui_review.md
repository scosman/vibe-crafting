# UI Review

The human review step for user-visible changes. Shared by `/spec implement`, `/spec task`, and `/spec pr` — your command file says where it sits in the step list and what step number it gets.

## Why It Runs After Commit

1. **The agent is often not on the user's machine.** Containers, remote sandboxes, and worktrees mean the user frequently can't see your working tree at all. Committing — and pushing, where the command pushes — is what makes the change reviewable. They sync, then look.
2. **Code review covers code quality, not design.** A passed code review is not a signoff on how something looks or feels. That's a separate human judgment, and it belongs at the end of the flow rather than as a gate in the middle of it.

Committing UI on a triaged code review alone is intentional. If the user wants changes, that's a follow-up commit — not a reason to have held the first one.

## When It Applies

Run it when the changes are **significant and user-visible**: new screens, layout changes, new components, visual redesigns, changed flows.

Skip it when:

- The coding agent returned `<ui_review>NONE</ui_review>`
- Nothing user-visible changed (backend work, refactors, tooling, tests)
- The changes are trivial (copy tweaks, minor spacing)

When you skip it, mark the step `skipped` in the progress block with the reason. These criteria decide whether the step runs at all; a return pass after a quick fix always re-presents.

## What to Send the User

Build the message from the coding agent's `<ui_review>` block — don't rewrite its bullets or pad them out. On a return pass after a quick fix, reuse the original block and list the changes just applied.

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
- **User requests changes**: route each request down one of two paths. If any request takes the functional route, fold the quick fixes into that feedback — the round gets reviewed anyway — and skip the quick-fix agent.
  - **Quick fix — copy and style.** Strings must be dictated exactly — where, from what, to what. Style requests — spacing, colour, size, alignment on a named element — can be relayed in the user's words ("more space above the header"): the quick-fix agent picks the concrete value from the project's existing scale, because a human judges the visual result at the next pass here. Spawn a fresh quick-fix sub-agent with the UI Quick Fix Prompt below. It is not code-reviewed: string changes are trusted, and style changes are judged by the user. When it returns complete and in scope with an all-TRUE/NA attestation, resume it with the UI Quick Fix Commit Prompt, run your command's verify step (resuming the quick-fix agent, not the coding agent, if the commit is incomplete; push where the command pushes), and return here. If it returns a scope change, the change needed interpretation after all — send that item down the functional route. A hook failure on its commit goes the same way. A FALSE attestation is a scope change too. Fixes it did apply stay in the tree — the functional round's review covers them — and when a hook failed, put its report in the feedback block alongside the user's words.
  - **Functional change — anything bigger.** New or altered behaviour, layout that needs design judgement, edits across several places: resume the coding agent with the UI Feedback Prompt below (or, after a consolidated review, spawn a fresh one — see below), then re-enter the flow at attestation → code review and triage → commit → verify, and return here. Increment the round counter on those steps.

A quick fix names the element and the adjustment; a functional change would need you to describe behaviour or structure. When in doubt, take the functional route.

**A UI quick fix is committed without a code review.** It earns that by being dictated, not designed — the user asked for this string, this spacing. Everything else from a UI review is a code change like any other: it goes through the coding agent and a review before it is committed.

## Consolidated Review (`implement all`)

`/spec implement all` does **not** run this step per phase — the `all` directive means the run does not stop for it. Instead:

1. Keep each phase's `<ui_review>` block as you go
2. After the last phase commits and verifies, present **one** review covering the whole run, bullets grouped under their phase
3. Drop phases whose block was `NONE`. If every phase was `NONE`, skip the step entirely.

For functional-route feedback on a consolidated review, spawn a **fresh** coding agent with the UI Feedback Coding Prompt below rather than resuming a phase agent — the feedback may span several phases, and the relevant agent may be many phases back. The quick-fix route is the same as ever: its agent is a fresh spawn either way, and it commits its own work.

## Prompt Templates

### UI Quick Fix Prompt (fresh spawn)

```
You are applying quick fixes from a UI review, on work that is already committed.

[IF project]: **Project specs:** [specs/projects/PROJECT_NAME/]
[IF task]: **Task file:** [.specs_skill_state/tasks/SLUG.md]

Read `references/quick_fix_prompt.md` for your full instructions. Follow them precisely.

<quick_fixes>
[One item per change: for strings — where, exact current text, exact new text; for style — the element and the user's words]
</quick_fixes>

Return one of the two completion messages described in your instructions.
```

### UI Quick Fix Commit Prompt (resume quick-fix agent)

```
Your fixes are accepted. Commit all changes with a short message describing the UI feedback you applied.

Return the commit message you used.
```

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
