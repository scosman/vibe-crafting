# Fan-Out Pattern

The shape shared by `/spec deep cr`, `/spec design crit`, and `/spec research`: **determine scope → design N units of work → write a plan → fan out one sub-agent per unit → wait and re-dispatch failures → collapse the results into one consolidated summary.**

This file is the mechanics. Your command file supplies the specifics: what a unit is (a review phase, a research subtopic), what the plan and summary files are called, where they live, and any preconditions or extra pauses of its own.

## Why This Shape

**Depth needs isolation; the user needs one answer.** A single agent asked to cover six concern areas gives each one a shallow pass and runs out of context doing it. Six agents each owning one area go deep — but six reports are not an answer.

So: fan out for depth, collapse for legibility. The manager holds a small context throughout because the work lands in **files**, not in return payloads. Sub-agents return a few lines; the manager never asks one to paste its findings back.

## The Shape

1. **Scope** — determine what's in play (a diff, a set of spec files, a topic). Command-specific.
2. **Plan** — decompose into units, write the plan artifact, get approval.
3. **Fan out** — one fresh sub-agent per unit.
4. **Wait** — all units complete; re-dispatch failures.
5. **Collapse** — one consolidated summary that links down to per-unit detail.
6. **Present** — hand the user the artifacts, not the contents.

## Designing the Units

Each unit goes to exactly one sub-agent, so each should be:

- **Independently workable** — an agent alone, with its prompt and the repo, can finish it
- **Non-overlapping** — two agents covering the same ground pay twice for one result
- **Worth a whole agent** — a unit that's five minutes of work belongs folded into a neighbor

Prefer fewer, meatier units over many thin ones. Every unit costs a full agent context.

Give each unit a **focus paragraph**: what it should cover, and what is explicitly some other unit's job. That boundary statement is what keeps the fan-out from collapsing into N copies of the same general pass.

Commands may also define **trigger-based units** — standard units included when a condition holds, whose detailed instructions live in their own resource file. When a command has those, you only evaluate the trigger; the sub-agent loads the resource itself.

## The Plan Artifact

The manager writes the plan file. This is the one authoring exception to the manager role — a plan is orchestration, not the work itself.

Every plan file, whatever the command calls it, carries:

- **The scope** it was built from (branch and fork point, spec files, research goal)
- **A checklist of units**, one line each — this gets checked off as they complete

Every unit also needs a **focus paragraph** — what it covers, and what is explicitly some other unit's job. Where that paragraph lives is a command decision: in the plan file, in the unit's dispatch prompt, or both. Your command file's plan template and prompt templates settle it. If the plan carries the focus paragraphs, sub-agents read the plan to find their lane; if the prompts do, the plan stays a checklist and the prompt is the sub-agent's whole brief.

**After writing the plan, expand the per-unit sub-steps in your progress block to match it** (Step 2a, 2b, 2c… — your command file says which step number). The plan and the progress block should name the same units in the same order.

## Plan Approval

Present the plan and **wait for explicit approval before dispatching anything.** Show the scope, the unit list, and where the output will go.

If the user wants changes — add, drop, merge, or re-scope units — update the plan file and re-confirm. Don't dispatch against a plan the user has already asked you to change.

Approval covers form (are we working on the right things?) and, where the command spends money per unit, cost. Your command file states whether approval is unconditional and any exception to it.

## Fan Out

Spawn a **fresh** sub-agent per unit, using the prompt template in your command file.

→ Read [references/spawning_subagents.md](../spawning_subagents.md) for how to spawn sub-agents.

- **Dispatch in a mode that returns the agent's final message to you.** A completion notification is not a result. (In Claude Code: an unnamed `Agent()` call — passing `name` loses the payload.)
- **Save the handle** the tool gives you, so you can chase a stalled agent.
- **One unit per agent.** Never hand an agent two units to save a spawn.
- **Sub-agents write to files and return a short summary** — name, headline result, counts, gaps. Their prompt tells them where to write, and carries whatever context your command file's template says it carries.

**After each sub-agent returns:**

1. Output the updated progress block with that unit's result
2. Check the unit off in the plan file
3. Confirm the agent actually wrote its output file — a return summary is not evidence the file exists
4. Immediately dispatch the next pending unit. Do not stop for user input.

## Wait and Re-Dispatch

**Wait for every unit before collapsing.** A summary written over a partial fan-out is worse than no summary — it reads as complete and isn't.

Re-dispatch a unit whose agent errored, returned without writing its output file, or reported it couldn't finish. Give the replacement the same prompt plus a line about the prior attempt, so it builds rather than restarts:

```
A previous agent worked this unit and did not finish. Anything already written under
[unit's output path] is yours to build on or replace.
```

**Cap the attempts at 3** unless your command file says otherwise. Some units genuinely can't be completed — a paywalled source, a file that no longer exists. When one hits the cap, stop retrying, mark it in the progress block, and carry it into the collapse step as a known gap. Never stall the whole run on one unit.

## Collapse

One consolidated summary at the root of the run folder. It is the deliverable — the per-unit files are the depth beneath it.

**Progressive disclosure, one level at a time.** The summary links to the per-unit outputs; those link to whatever detail sits below them. Don't reach past a level: link the unit summary, not the deep doc behind it.

The summary:

- **Leads with the answer.** Someone reading only this file should be able to act.
- **Synthesizes, rather than concatenating.** Cross-unit patterns, conflicts between units, and deduplicated findings are the value the collapse adds. N reports stapled together add nothing.
- **Reports its own gaps** — units that failed or came back thin, and what that leaves unanswered.
- **Has working links.** Verify the paths exist before you finish.

Who writes it is a command decision: the manager, or a **fresh** sub-agent that reads the per-unit outputs. Never a resumed unit agent — its synthesis is colored by having done one slice of the work. Your command file says which, and gives the summary's format.

## Present

Keep it short. Give the user:

- The run folder path and the summary path
- The summary's headline — the overview table, or the bottom line
- Anything that needs attention now (critical findings, failed units)

Don't restate the summary in chat. You just wrote it to a file for a reason.

## Autonomous Flow

**Once the fan-out begins, drive to completion without stopping for user input.** After each return: progress block, then the next dispatch. After the last unit: collapse, then present.

The pauses are the ones your command file names — plan approval always, plus any scope confirmation before it or interactive phase after it. Between those, don't ask "should I continue?", don't wait for approval between units, and don't stop because a unit found something alarming. The progress block tells you what to do next — do it.
