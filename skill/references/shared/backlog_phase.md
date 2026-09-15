# The Backlog Phase

**Load this file only when you reach the Backlog phase.** It is the last phase of a project that has a `backlog.md`, and it is the only phase in `/spec implement` that is interactive. Nothing here applies to any other phase, so don't read it during a normal run.

Your command file (`cmd_implement.md`) says what the backlog is for and how items get onto it. This file is how the phase gets *run*.

## Why This Phase Is Different

Every other phase is autonomous: the spec already contains the decisions, and the coding agent executes them. Backlog items are the opposite — they are on the backlog *because* nobody has decided yet. A defect someone chose not to fix mid-phase, an assumption nobody validated, a spec that can't work as written. The work is unblocked by a decision, not by an agent.

So the phase has three parts, in order, and none of them may be skipped or interleaved:

1. **Pre-flight** — sub-agents verify each item and gather the context the decisions need
2. **The interview** — you and the user decide every item, in chat, deciding nothing else
3. **Execution** — the decided work is built, as a phase run, as `/spec task` runs, or as a mix

The separation is the point. Implementing after each decision makes the user watch a build between every question, and it commits work before you know whether a later decision moots it.

## Using This Outside the Backlog Phase

The middle of this file is not really about backlogs. **Verify each open question → ask it with enough context to be decidable → record the answer → execute the whole set at once** is the shape of any point where a flow needs a batch of decisions from the user, and it is worth reusing wherever that comes up.

Steps [B1](#step-b1-pre-flight-verification), [B2](#step-b2-the-interview) and [B3](#step-b3-the-plan-gate) carry it, along with the [Never](#never) list. Substitute your own queue for the items, your own artifact for `backlog.md`, and your own runs for Step B4. The parts that are genuinely backlog-specific — the file format, the wrap-up, the phase checkbox — are marked as such and don't generalize.

The rest of this file is the Backlog phase itself.

## Progress Tracker

→ Read [references/shared/progress_tracker.md](progress_tracker.md) for the block format and rules.

Use the label **"Backlog Phase Progress"**. This phase replaces the standard step list with its own:

```
<progress>
Backlog Phase Progress:
- [x] Step B0: Pre-checks — complete (7 open items)
- [x] Step B1: Pre-flight verification — complete (2 already fixed, 5 need decisions)
- [ ] Step B2: Decisions — in progress (3 of 5)
- [ ] Step B3: Plan approved — pending
- [ ] Step B4: Execution runs — pending
- [ ] Step B5: Wrap-up — pending
- [ ] Step B6: Verify — pending
- [ ] Step B7: Summary — pending
</progress>
```

Expand Step B4 into one sub-step per run (B4a, B4b, …) once the plan is approved, so the block and the plan name the same runs in the same order.

## File Format

Three writers touch `backlog.md` and they need to agree on its shape. Every item is a checklist entry with a stable ID:

```markdown
- [ ] **B3 — Token refresh drops the retry queue**
  Requests queued during a refresh are discarded instead of replayed. Found in Phase 4 review.
```

The manager appends a decision block during the interview, and a route line at the plan gate:

```markdown
- [ ] **B3 — Token refresh drops the retry queue**
  Requests queued during a refresh are discarded instead of replayed. Found in Phase 4 review.
  - **Decision (2026-09-15):** fix — replay the queue after the new token lands
  - **Why:** silent data loss on any expiry mid-session; user picked replay over failing the requests
  - **Route:** task run 2
```

The sub-agent that does the work ticks the box and adds the status, as part of its own commit:

```markdown
- [x] **B3 — Token refresh drops the retry queue**
  ...
  - **Status:** closed — see commit abc1234
```

**Who writes what, and never otherwise:**

| Writer | Writes |
|---|---|
| Coding agent, in an earlier phase's commit | The item itself, when a review routes a finding here |
| **Manager, during this phase** | The `Decision`, `Why`, and `Route` lines |
| Task-run sub-agent, in its own commit | The checkbox tick and the `Status` line |

The manager writing decisions is a deliberate exception to "you do not edit the backlog yourself" in `cmd_implement.md`. It exists for durability: this phase is a long interactive stretch, and a decision that lives only in the chat is lost to a compaction. Write each decision to the file as soon as the user gives it, before you ask the next question.

## Step B1: Pre-Flight Verification

Backlog items were filed phases ago. Some are already fixed by later work, some are moot, some no longer reproduce. Verifying them before you ask anything means you skip the dead ones and the context you give the user is actually true.

It also keeps you in role. **You are a manager: you do not read code to work out what an item means.** Sub-agents do that, and hand you the plain-language framing the interview needs.

→ Read [references/spawning_subagents.md](../spawning_subagents.md) for dispatch mechanics.

Dispatch fresh sub-agents in parallel, in one message, using the Item Verification Prompt below. Group items into 2–5 agents rather than one per item — a single item is rarely worth a whole context. Group by area of the codebase where you can, so one agent's reading serves several items.

**These agents are read-only.** They investigate and report. They do not fix, do not edit, do not commit. Say so in the prompt, and if one returns having changed the tree, discard the change and re-dispatch.

Each agent returns one `<item_check>` block per item. Re-dispatch any that errored or came back without a block, capped at 3 attempts; an item you can't verify goes to the interview marked as unverified, with whatever is known.

**Items that come back `already-fixed` or `moot` still get a decision** — but a fast one, in the batch described below. Never resolve an item on a sub-agent's say-so alone.

## Step B2: The Interview

### Run it in chat

Ask in prose, in the conversation. **Do not use the question tool, and do not ask the user to open or edit `backlog.md`.** The question tool caps how much context fits around a choice, and these decisions need the full framing — what it is, what it costs, the options and their sizes, what happens if nothing is done. A user picking from terse options they have no context for is guessing, not deciding.

The user may of course read `backlog.md` if they want, and you keep it current — but reading it is never a prerequisite for answering you.

### Assume no context

The user has not read the code. They may not have read the phase summaries. They did not see the review that filed the item. Write every question as if they are coming to it cold:

- Lead with the **user-visible symptom**, not the reviewer's jargon or a file path
- Say **what it costs** — what a user of the product experiences, and what breaks if nobody acts
- Give **options with sizes** — "quick fix", "a task", "bigger than this phase" — so the choice can be weighed
- Give a **recommendation**, always, and be willing to recommend not fixing it

### The question format

> **B3 of 5 — Token refresh drops the retry queue**
>
> When a session's token expires mid-use, requests that were in flight get thrown away instead of retried. The user sees a few actions silently do nothing, with no error. It happens on any session long enough to cross an expiry — likely most real sessions.
>
> **Options:**
> - **A. Replay the queue after refresh** — the queued requests fire again once the new token lands. A task; touches the auth client and its tests.
> - **B. Fail them loudly** — surface an error and let the user retry by hand. Quick fix, but it puts the work on the user.
> - **C. Leave it** — accept silent loss on expiry. No work now; the bug ships.
>
> **Recommendation: A.** Silent data loss is the worst failure mode of the three, and the fix is contained.

### Ordering

1. **Dependency first.** If one item's answer makes another moot or changes its options, ask it first.
2. **Then scope.** Anything that could change the spec or the shape of the remaining work goes early — those answers delete later questions.
3. **Then importance.** Most consequential first, so the decisions that matter are made while the user is fresh.

### Decisions

A decision is one of:

| Decision | Means | Resolves as |
|---|---|---|
| **Fix** | Do it, using the named option | closed |
| **Change spec** | The spec was wrong; fix the code *and* the spec artifact | closed |
| **Don't fix** | Deliberately accepting it | dismissed |
| **Move to issue tracker** | A real issue, but not this project's work | dismissed, with the destination recorded |

A **change spec** decision means the task run also edits `functional_spec.md` / `architecture.md` / the component design, and sets that artifact's frontmatter status. Say so in the recommendation when you offer it — the user is approving a spec edit, not just a code change.

A **move to issue tracker** decision records where it went (an issue ID or URL if the user has one, "user will file" if not). This phase does not file it unless the user asks.

Accept these answers without re-asking:

- **"Your call"** — record the recommendation as the user's decision, noting they delegated it
- **An option you didn't offer** — record theirs, not the nearest one on your list
- **A question back** — answer it, then re-ask. A clarifying question is not a decision

### Batching

Batch only where a batch can still carry enough context to decide, which means:

- **Trivial** — obvious either way, asked only because it changes the spec or closes an item
- **Already fixed or moot** — the pre-flight found it resolved; you need confirmation, not deliberation
- **Coupled** — one answer settles several items

Never batch to get through the list faster. If an item needs its impact explained to be decidable, it gets its own question, however small the fix is.

A batch is one message, items numbered, each with its own one-line framing and its own recommendation, capped at about five. Say that **"all recommended"** is a valid answer, and that any item can be answered individually.

### Rules for the whole interview

- **Decide nothing else and build nothing.** No implementation, no fixes, no commits until Step B4. Not even a one-line fix that would take less time than asking about it.
- **Write each decision to `backlog.md` immediately**, before the next question.
- **Show position** — "B3 of 5" — so the user knows how much is left.
- **New findings get folded in, not filed.** If the conversation surfaces something new, raise it as an additional item in the same format and decide it here. `cmd_implement.md` forbids routing anything to the backlog during this phase; that stands. Drop it or take it now.
- **No decision means the phase stays open.** If the user stops answering or explicitly defers an item, say which items are undecided, leave the phase checkbox unticked, and stop. Decisions already recorded stay in the file and the phase resumes from there.

## Step B3: The Plan Gate

One table, once, covering both what was decided and what will happen. This is the only confirmation between the interview and execution — the user has been answering questions across many messages and should see the whole set before it becomes commits.

> **Backlog decisions — 5 items**
>
> | # | Item | Decision | Route |
> |---|---|---|---|
> | B1 | Token refresh drops the retry queue | Fix — replay after refresh | Task run 1 |
> | B2 | Retry budget shared across tenants | Change spec — per-tenant budgets | Task run 2 |
> | B3 | Config loader ignores env overrides | Fix — read env last | Phase run |
> | B4 | Three error strings say "failed" with no cause | Fix — name the cause | Phase run |
> | B5 | Slow query on the audit table | Move to tracker — PROJ-412 | Phase run (bookkeeping) |
>
> Auth and the retry budget each get their own task run — they are the substantial changes and are cleaner as separate commits. B3 and B4 are small and both in config/copy, so they go in one phase run at the end, which also marks B5 dismissed and closes out the phase.
>
> Good to go?

**You pick the shape.** There are two kinds of run and the plan may use either or both:

- **Phase run** — the Single Phase Flow from `cmd_implement.md`, one coding agent writing `phase_plans/phase_N.md` and covering several items in one commit. Right when the remaining items are small and shallow — string fixes, a missing null check, a test gap — where separate commits would be noise.
- **Task run** — a `/spec task`, one group of items, its own task file, review and commit. Right when an item is substantial, sits in its own area, or is worth isolating in the history so it can be reverted alone.

Mixed is the common shape: the one or two real items as task runs, then a phase run that sweeps up the small ones. All task runs is right when every item is substantial. A single phase run is right when none of them are.

State the reasoning in the paragraph under the table — the user is approving a shape, not just a list, and "why these two are separate" is the part they can actually judge.

After approval, write each item's `Route` line into `backlog.md`, then expand Step B4 in the progress block to match. If the user changes anything, update the table and the file, and re-confirm.

## Step B4: Execution Runs

**No item is fixed by the manager, and none rides in on a bare commit.** Every closing item is built by a sub-agent and code-reviewed before it lands, whichever kind of run carries it.

**Run them serially, one at a time**, in the order the plan lists them. Every run commits, and concurrent agents committing into one working tree collide. This is a deliberate exception to the parallel dispatch in [references/shared/fan_out_pattern.md](fan_out_pattern.md). Dispatch the next run only after the previous has returned and you have verified its commit.

### Task runs

**Dispatch each as a single sub-agent** that runs the whole `/spec task` flow itself — task file through commit — spawning its own coding and CR agents internally. You track N runs, not N runs × seven steps each. That is the point of the nesting: your context stays small enough to hold the decisions.

Use the Backlog Task Run Prompt below.

### The phase run

Run the Single Phase Flow from `cmd_implement.md` — Steps 1 through 4 — but spawn the coding agent with the **Backlog Phase Coding Prompt** below instead of the Initial Coding Prompt. Everything else is unchanged: attestation, code review and triage, the CR feedback loop, commit, verify. You manage this one directly, the way you manage any phase.

There is at most one phase run, and it is last — it writes `phase_plans/phase_N.md` for the Backlog phase, and a phase has one plan.

### After every run

1. Output the updated progress block
2. Run `git status` and `git log -1` — confirm the tree is clean and the commit landed
3. Confirm the run's items are ticked in `backlog.md` with a `Status` line
4. Dispatch the next run

If a run returns a roadblock, escalate it the way any phase escalation is handled: present it to the user, get a decision, resume that run's agent with the answer.

## Step B5: Wrap-Up

Closing the phase out is its own work, and it belongs to the **last run in the plan**:

- Mark every **don't fix** and **move to tracker** item dismissed in `backlog.md`, with its destination where there is one
- Verify every item in the file is now closed or dismissed — none left open
- Tick the Backlog phase checkbox in `implementation_plan.md`

Both prompt templates carry these instructions; give them to whichever run goes last. Fold it in rather than adding a run for it — the bookkeeping is reviewed as part of that run either way.

Add a dedicated run only when there is nothing to fold it into: **every item was dismissed**, so the phase has no code work at all. Then the wrap-up is a task run of its own, using the Backlog Wrap-Up Prompt below, and it is the only run in the phase. The phase still ticks. Dismissing is a resolution.

**The review on the run that carries the wrap-up is not ceremony.** The reviewer's job there is to check that each item's recorded resolution matches what actually landed in the code. A `Status: closed` line over a fix that was never made is exactly the failure this phase exists to prevent — say so in the prompt, as the templates do.

## Step B6: Verify

Run `git status`. The tree is clean, the commits exist, and:

- Every item in `backlog.md` is `[x]` with a `Status` line
- The Backlog phase checkbox in `implementation_plan.md` is ticked
- No item was added to `backlog.md` during this phase

If an item is still open, the phase is not done — the wrap-up missed something. Resume the agent that carried the wrap-up with what is outstanding.

## Step B7: Summary

Short. What was decided, what was built, what was dismissed and why, and anything the user now owns outside this project (tracker items they said they'd file).

## Never

- Never use the question tool for a backlog decision, or ask the user to edit `backlog.md`
- Never implement, fix, or commit between decisions
- Never carry an undecided item into execution
- Never let an item with no decision drop silently — it keeps the phase open
- Never resolve an item because a sub-agent said it was already fixed
- Never add an item to `backlog.md` during this phase
- Never dispatch execution runs in parallel, and never start one before the previous has committed
- Never read code or run analysis yourself to frame a question — that is what Step B1 is for

## Prompt Templates

Use these verbatim, filling in the bracketed values.

### Item Verification Prompt (fresh spawn, read-only)

```
You are verifying open backlog items for a spec-driven project, so a manager can ask the user to decide them.

**Project specs:** [specs/projects/PROJECT_NAME/]
**Backlog:** [specs/projects/PROJECT_NAME/backlog.md]

**You are read-only.** Investigate and report. Do not fix anything, do not edit any file, do not commit. Your entire output is the blocks below.

Your items:
[item ID and text, one per line]

For each item: read the relevant code, specs and git history, and determine whether it is still a real, open problem. Then return one block per item, in this exact format:

<item_check id="[ID]">
STATUS: open | already-fixed | moot | cannot-reproduce
WHAT: [the problem in plain language, led by what a user of the product would experience — not reviewer jargon, not a file path]
IMPACT: [what it costs, and what happens if nobody acts on it]
OPTIONS:
  A. [approach] — size: quick fix | a task | bigger than one task
  B. [approach] — size: ...
  [only real options; one is fine if there is only one sensible fix. Include leaving it as-is when that is defensible.]
SPEC: [name any spec artifact that would have to change, or NONE]
EVIDENCE: [file:line references, the commit that already fixed it, why it no longer reproduces]
</item_check>
```

### Backlog Task Run Prompt (fresh spawn, serial)

```
You are running one `/spec task` for the Backlog phase of a spec-driven project. You are the manager of this task: read `references/cmd_task.md` and follow the full flow — task file, coding agent, code review and triage, commit, verify. Do not write code yourself.

**Project specs:** [specs/projects/PROJECT_NAME/]
**Backlog:** [specs/projects/PROJECT_NAME/backlog.md]

The user has already decided these items. Do not re-open the decisions and do not ask the user about them — implement exactly what was decided.

<backlog_items>
[For each item: ID, title, the decision, the chosen approach, and any spec artifact that must change]
</backlog_items>

Skip Step 0a (clarification) — the decisions above are the clarification. Write the task file from them.

As part of your commit, in `backlog.md`: tick each item above and add a `**Status:** closed — see commit [sha]` line to it. Where a decision changes a spec artifact, edit that artifact and set its frontmatter status in the same commit.

[IF this is the last run in the plan, append:]
This run also closes out the Backlog phase. Before committing:
- Mark each item below dismissed in `backlog.md`: tick it and add `**Status:** dismissed — [reason, or destination]`
<dismissed_items>
[For each: ID, title, "don't fix" or "move to issue tracker", and the destination where there is one]
</dismissed_items>
- Verify every item in `backlog.md` is now closed or dismissed. If any is still open, stop and report it rather than ticking the phase
- Tick the Backlog phase checkbox in `implementation_plan.md`

Your reviewer will check that each item's recorded resolution matches what actually landed in the repo — a `closed` line over a fix that was never made is the failure this phase exists to prevent.

Return the commit message you used, and one line per item confirming it is marked closed.
```

### Backlog Phase Coding Prompt (fresh spawn, replaces the Initial Coding Prompt)

Use this for the phase run, then drive the Single Phase Flow from `cmd_implement.md` as normal.

```
You are a coding agent implementing the Backlog phase of a spec-driven project.

**Phase:** [N]
**Project specs:** [specs/projects/PROJECT_NAME/]
**Backlog:** [specs/projects/PROJECT_NAME/backlog.md]

Read `references/coding_phase_prompt.md` for your full instructions. Follow them precisely. Write the phase plan from the items below — they are this phase's entire scope.

The user has already decided these items. Do not re-open the decisions and do not ask about them — implement exactly what was decided.

<backlog_items>
[For each item: ID, title, the decision, the chosen approach, and any spec artifact that must change]
</backlog_items>

In `backlog.md`, tick each item above and add a `**Status:** closed — see commit [sha]` line to it. Where a decision changes a spec artifact, edit that artifact and set its frontmatter status too.

[IF this is the last run in the plan, append:]
This run also closes out the Backlog phase. Before committing:
- Mark each item below dismissed in `backlog.md`: tick it and add `**Status:** dismissed — [reason, or destination]`
<dismissed_items>
[For each: ID, title, "don't fix" or "move to issue tracker", and the destination where there is one]
</dismissed_items>
- Verify every item in `backlog.md` is now closed or dismissed. If any is still open, stop and report it rather than ticking the phase
- Tick the Backlog phase checkbox in `implementation_plan.md`

Your reviewer will check that each item's recorded resolution matches what actually landed in the repo — a `closed` line over a fix that was never made is the failure this phase exists to prevent.

Return a short summary of what you built when implementation is complete and ready for code review.
```

### Backlog Wrap-Up Prompt (fresh spawn — only when every item was dismissed)

```
You are running the only `/spec task` of the Backlog phase of a spec-driven project — the wrap-up. Every item was dismissed, so there is no code to write. You are the manager of this task: read `references/cmd_task.md` and follow the full flow, code review included. Do not write code yourself.

**Project specs:** [specs/projects/PROJECT_NAME/]
**Backlog:** [specs/projects/PROJECT_NAME/backlog.md]

The task is bookkeeping, and it is reviewed because the bookkeeping has to be true.

<dismissed_items>
[For each: ID, title, the decision — "don't fix" or "move to issue tracker" — and the destination where there is one]
</dismissed_items>

Do:
1. Mark each item above dismissed in `backlog.md`: tick it and add `**Status:** dismissed — [reason, or destination]`
2. Verify every item in `backlog.md` is now closed or dismissed. If any is still open, stop and report it rather than ticking the phase
3. Tick the Backlog phase checkbox in `implementation_plan.md`
4. Commit

The code reviewer's job on this task is to confirm that every item's recorded resolution matches what actually landed in the repo — a `closed` line over a fix that was never made is the failure this phase exists to prevent.

Return the commit message you used, and confirmation that no backlog item remains open.
```
