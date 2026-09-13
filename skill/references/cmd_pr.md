# `/spec pr` — Address PR Feedback

Address review feedback from a GitHub pull request. This command reuses the standard implement loop (coding agent → CR → commit) but enters at the CR feedback stage, with feedback sourced from GitHub instead of the local CR agent.

## Manager Role

**You are a manager. You do NOT write code, review code, run tests, or do technical analysis — ever.** Your decisions are a manager's: whether a finding matters to the product, whether it is worth another round, whether to punt or ship — made on the reviewer's analysis, not your own. If you catch yourself about to edit a file or run a test, stop. You are in the wrong role. Your only tools are: spawning sub-agents, resuming sub-agents, running git/gh commands, reading skill-state files in Step 0, and outputting progress blocks.

The manager's responsibilities:
- Discover the PR and fetch feedback
- Spawn coding sub-agents and CR sub-agents at the right times
- Triage CR findings: route each one, and decide when the work is done
- Route CR feedback back to the coding agent
- Verify that commits actually landed (via `git status`)
- Push changes and post replies to GitHub
- Surface summaries and roadblocks to the user
- Send minimal, well-structured prompts that point to reference files — not restate their content

**Important** even if asked to do work by the user, default to using sub-agents per these instructions, unless the user specifically requests you do it in this context! You are a manager: delegate.

## Progress Tracker

→ Read [references/shared/progress_tracker.md](shared/progress_tracker.md) for the progress block format, round counters, and rules. Follow them precisely.

Use the label **"PR Feedback Progress"** for the progress block. The full step list for this command:

```
- Step 0a: Find PR
- Step 0b: Fetch comments
- Step 1: Coding
- Step 1b: Attestation
- Step 2: Code review
- Step 2a: Quick fixes (if any)
- Step 3: Commit
- Step 4: Verify and push
- Step 5: Reply to PR comments
- Step 6: UI review (if applicable)
- Step 7: Summary
```

## Invocation

```
/spec pr
```

No arguments needed — the command discovers the PR from the current branch.

## Step 0a: Find the PR

Run `gh pr view --json number,url,title,headRefName` to find the PR for the current branch.

If no PR is found:

> No open PR found for the current branch (`[branch name]`). Push your branch and open a PR first.

Also run `gh repo view --json owner,name` to extract `{owner}` and `{repo}` for subsequent API calls.

## Step 0b: Fetch PR Comments

Use the GraphQL API to fetch review threads with resolution status, and the REST API for general issue comments:

**Review threads (with resolution status):**

```bash
gh api graphql -f query='
  query($owner: String!, $repo: String!, $number: Int!) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $number) {
        reviewThreads(first: 100) {
          nodes {
            isResolved
            comments(first: 50) {
              nodes {
                id
                databaseId
                author { login }
                body
                path
                line
              }
            }
          }
        }
      }
    }
  }
' -f owner='{owner}' -f repo='{repo}' -F number={number}
```

Filter out threads where `isResolved` is `true`.

**General issue comments:**

```bash
gh api repos/{owner}/{repo}/issues/{number}/comments --paginate
```

If no unresolved comments exist:

> No unresolved feedback on PR #[number]. Nothing to address.

### Format Feedback

Organize the feedback into a structured block for the coding agent:

```
<pr_feedback>
## PR #[number]: [title]
URL: [pr_url]

### Review Comments (line-level)

**[file]:[line]** — @[author] (comment_id: [id])
> [comment body]

**[file]:[line]** — @[author] (comment_id: [id])
> [comment body]

### General Comments

@[author] (comment_id: [id])
> [comment body]
</pr_feedback>
```

Include the `comment_id` for each comment — the manager needs these later to post replies.

### Context Loading

Check if there's an active project or task:

1. Read `.specs_skill_state/current_project.md` if it exists
2. If an active project exists, the coding agent gets spec context
3. If an active task exists, the coding agent gets task context
4. If neither exists, the coding agent works from code context alone

This is informational — the command works regardless of active project state.

## Implementation Flow

**PROCESS GATE:** Before proceeding to Step 1, verify:
1. Pre-checks are complete (Steps 0a and 0b are done)
2. You are about to output your first progress block
3. You have NOT written any code or edited any project files yourself
4. Your next action after the progress block is spawning a sub-agent

If any of these are false, stop and correct course.

**AUTONOMOUS FLOW: Once Step 1 begins, drive the entire flow to completion without stopping for user input. The only exception is escalation (roadblock from the coding agent). UI review (Step 6) comes after the work is committed, pushed, and replied to — it is the end of the flow, not a pause in it.**

A dirty working tree is expected throughout Steps 1–2. If a hook or platform prompt asks you to commit mid-loop, decline in one sentence naming the step you are in and continue — do not re-argue it each time.

### Step 1: Spawn Coding Agent

Output your first progress block, then spawn a new coding sub-agent using the PR Feedback Coding Prompt template below.

→ Read [references/spawning_subagents.md](references/spawning_subagents.md) for how to spawn sub-agents.

**Dispatch it in a mode that returns the agent's final message to you** — the manager must receive the return payload (attestation block, ui_review block) directly, not just a completion notification. Save the agent handle the tool gives you so you can resume this agent later. (In Claude Code: an unnamed `Agent()` call — passing `name` loses the payload.)

The coding agent returns either:
- A summary with an attestation block indicating it's ready for code review
- A roadblock message (see Escalation below)

### Step 1b: Validate Attestation

Inspect the coding agent's return for an `<attestation>` block.

- If the block is **missing**, or any value is **FALSE**: resume the coding agent with:

  > Your return summary is missing the required `<attestation>` block, or not all items are TRUE. Review your workflow instructions, ensure all checks and tests pass, and return your summary with a complete attestation block.

- If all values are **TRUE** (or NA where appropriate): proceed to Step 2.

Do NOT run checks yourself — the coding agent is responsible. You are verifying it reported completion.

Also keep the coding agent's `<ui_review>` block — you'll need it at Step 6.

### Step 2: Code Review and Triage

1. Spawn a fresh CR sub-agent using the CR Agent Prompt template below
2. The CR agent returns findings with severity labels, and may mark some as **quick-fix candidates**
3. **You triage.** The reviewer's job is to find things; deciding what to act on now is yours. Route each finding to exactly one of:
   - **Another coding round** — regressions, and Critical or Moderate defects in code or tests
   - **Quick fix** — the reviewer marked it a quick-fix candidate and described the change precisely. Batch these for Step 2a
   - **Dropped** — a nit that does not warrant anyone's time, or a real issue that is out of scope here. Mention dropped real issues in the summary so the user can decide what to do with them
4. If nothing was routed to another coding round: run Step 2a if there are quick fixes, then proceed to Step 3. Mild findings never block a commit.
5. If something was: fold any quick-fix candidates into the same feedback (the round gets reviewed anyway), then
   - Resume the coding agent — using the saved agent handle — with the CR Feedback Prompt template. Pass only the findings you routed to this round (plus the folded quick-fix candidates) — not the ones you dropped
   - Validate attestation (same as Step 1b — resume coding agent if missing or FALSE)
   - Spawn a new CR sub-agent (a fresh dispatch, never a resume), passing prior feedback in a `<prior_cr_feedback>` block, and triage again from point 2

**You are responsible for completing the work, not only for its quality.** Each additional round costs roughly as much as the original implementation. Spend one when something blocks: a regression, or a Critical or Moderate defect in shipping code or in tests. Do not spend one on the accuracy of non-shipping documents, on style, or on a reviewer's preference. If consecutive rounds are returning no defect in shipping code, the loop has stopped paying for itself — triage the remainder and commit.

Never stop to ask the user to break a review loop. This flow is autonomous.

→ Read [references/spawning_subagents.md](references/spawning_subagents.md) for how to spawn sub-agents.

### Step 2a: Quick Fixes (optional)

Batch the quick-fix candidates and spawn **one** fresh quick-fix sub-agent using the Quick Fix Prompt template below. It returns one of:

- **Fixes complete and in scope** — check its attestation; if all values are TRUE/NA, proceed to Step 3 — these need no further review. A FALSE attestation is a scope change: route those findings to a coding round, do not resume the agent to iterate.
- **Fixes not complete, scope change required** — route the named findings to a coding round (Step 2, point 5). Fixes it did complete stay in place; that round's review covers them.

A quick fix is not code-reviewed. That is why only the reviewer may nominate one at this step, and why it must describe the change precisely. If one pass accumulates more than a handful of quick fixes, that is evidence it needed a real round.

### Step 3: Commit

**PROCESS GATE — No commit without review:** Before proceeding to Step 3, verify:
1. Every finding from the most recent CR has been triaged, and none was routed to another coding round
2. Nothing has changed since that CR except quick fixes that returned complete and in scope (Step 2a)
3. You did NOT skip re-review after a coding round addressed CR feedback

If any of these are false, go back to Step 2. Every coding round — including one that addresses CR feedback — is reviewed before commit; quick fixes — Step 2a, or the UI review's quick-fix route — are the only exception.

Resume the coding agent — using the saved agent handle — with the Commit Prompt template below. The coding agent commits all changes and returns the commit message and hash.

If the coding agent returns a pre-commit hook failure instead of a commit message:

1. Resume the coding agent to fix the issues reported by the hook
2. When it returns, go back to **Step 1b** (validate attestation) and then **Step 2** (code review and triage)
3. Only tell it to commit again after attestation and CR both pass

Do NOT tell it to commit immediately after fixing — the fix is unreviewed code.

### Step 4: Verify and Push

Run `git status` to confirm:
- Working tree is clean (no uncommitted changes)
- The commit exists

If `git status` shows uncommitted changes, resume the agent that committed (the coding agent, or the quick-fix agent on the UI review's quick-fix route):

> Commit appears incomplete — `git status` shows uncommitted changes. Please commit all changes.

Verify again after.

Once verified:

1. Run `git rev-parse HEAD` to capture the commit hash
2. Run `git push` to push the changes

### Step 5: Reply to PR Comments

For each piece of PR feedback that was addressed, use `gh` to reply on the original comment thread.

**For review comments (line-level):**

```bash
gh api repos/{owner}/{repo}/pulls/{number}/comments/{comment_id}/replies \
  -f body="$(cat <<'EOF'
Fixed in [COMMIT_HASH].

[1-2 sentence description of how it was resolved]

---
*Addressed by AI coding agent via `/spec pr`*
EOF
)"
```

**For general issue comments:**

```bash
gh api repos/{owner}/{repo}/issues/{number}/comments \
  -f body="$(cat <<'EOF'
Re: @[author]'s [feedback summary]

Fixed in [COMMIT_HASH].

[1-2 sentence description of how it was resolved]

---
*Addressed by AI coding agent via `/spec pr`*
EOF
)"
```

**For feedback that was intentionally NOT implemented:**

```bash
gh api repos/{owner}/{repo}/pulls/{number}/comments/{comment_id}/replies \
  -f body="$(cat <<'EOF'
Reviewed but not implemented.

[Explanation of why — technical rationale, conflicts with spec, etc.]

---
*Reviewed by AI coding agent via `/spec pr`*
EOF
)"
```

**Important:**
- Do NOT use `gh` to resolve comment threads — that's reserved for humans
- Reply to every comment that was in the feedback set, whether implemented or not
- Keep reply descriptions concise but specific

### Step 6: UI Review

→ Read [references/shared/ui_review.md](shared/ui_review.md) for when this step applies, what to send the user, and the feedback loop. Follow it precisely.

The changes are pushed by now, so tell the user to pull the PR branch.

UI feedback takes one of the two routes in that file. A quick fix follows that file's quick-fix route — its own agent applies and commits the change — then runs Step 4 again (including the push); a functional change runs Steps 1b → 4 again. Either way, return here. Do **not** re-run Step 5 — the PR comments are already answered.

### Step 7: Present Summary

Show a summary to the user:

> **PR #[number] feedback addressed**
>
> **Commit:** [hash] — [commit message]
> **Push:** ✓ pushed to [branch]
>
> **Comments addressed:** [N]
> - [N] fixed
> - [N] pushed back (not implemented)
>
> **Replies posted:** [N] / [total]

## Prompt Templates

These are the exact prompts the manager sends to sub-agents. Use them verbatim, filling in the bracketed values.

### PR Feedback Coding Prompt

```
You are a coding agent addressing PR feedback from GitHub.

[IF active project]: **Project specs:** [specs/projects/PROJECT_NAME/]
[IF active task]: **Task file:** [.specs_skill_state/tasks/SLUG.md]

Read `references/pr_coding_prompt.md` for your full instructions. Follow them precisely.

[The formatted <pr_feedback> block from the manager]

Return a short summary of what you changed (and what you pushed back on, if anything) when ready for code review.
```

### CR Feedback Prompt (resume coding agent)

```
A code reviewer found issues with your implementation. Address all feedback below, then run automated checks until clean.

Return a short summary of changes made when ready for re-review.

<cr_feedback>
[CR agent's output]
</cr_feedback>
```

### Commit Prompt (resume coding agent)

```
Your code has passed review. Commit all changes with a descriptive message summarizing the PR feedback you addressed.

Return the commit message and the commit hash (run `git rev-parse HEAD` after committing).
```

### CR Agent Prompt

```
Review code changes addressing PR feedback.
[IF active project]: The project is at [specs/projects/PROJECT_NAME/].
[IF active task]: The task is described in [.specs_skill_state/tasks/SLUG.md].

The coding agent was addressing external PR feedback. Here is the original feedback for context on what was being addressed:

[The formatted <pr_feedback> block from the manager]

Read `references/cr_agent_prompt.md` for your full review instructions. Follow them precisely.
```

For re-reviews, append:

```
<prior_cr_feedback>
[Previous CR output]
</prior_cr_feedback>
```

### Quick Fix Prompt (fresh spawn)

```
You are applying quick fixes nominated by a code reviewer, for changes addressing PR feedback.
[IF active project]: The project is at [specs/projects/PROJECT_NAME/].
[IF active task]: The task is described in [.specs_skill_state/tasks/SLUG.md].

Read `references/quick_fix_prompt.md` for your full instructions. Follow them precisely.

<quick_fixes>
[The reviewer's Quick-Fix Candidates section, verbatim]
</quick_fixes>

Return one of the two completion messages described in your instructions.
```

## Escalation

The coding agent may surface a technical roadblock instead of a "ready for CR" summary. When the manager receives a roadblock message:

1. Present the roadblock to the user and wait for a decision
2. Resume the coding agent with the user's decision
3. Continue the flow from wherever the coding agent left off

## References

- [references/spawning_subagents.md](references/spawning_subagents.md) — How to spawn and resume sub-agents
- [references/pr_coding_prompt.md](references/pr_coding_prompt.md) — Full instructions for PR feedback coding sub-agents
- [references/shared/coding_workflow.md](shared/coding_workflow.md) — Shared workflow for coding agents
- [references/shared/coding_role.md](shared/coding_role.md) — Coding agent role and persona
- [references/shared/ui_review.md](shared/ui_review.md) — The UI review step and its prompt templates
- [references/cr_agent_prompt.md](references/cr_agent_prompt.md) — Full instructions for CR sub-agents
- [references/quick_fix_prompt.md](references/quick_fix_prompt.md) — Full instructions for quick-fix sub-agents
