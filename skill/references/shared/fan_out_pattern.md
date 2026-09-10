# Fan-Out Pattern

The shape shared by `/spec deep cr`, `/spec design crit`, and `/spec research`: **determine scope → design N units of work → write a plan → fan out one sub-agent per unit → wait and re-dispatch failures → collapse the results into one consolidated summary.**

This file is the mechanics. Your command file supplies the specifics: what a unit is (a review phase, a research subtopic), what the plan and summary files are called, where they live, and any preconditions or extra pauses of its own.

## The Working Folder

**Every run of this pattern writes into exactly one folder, and the calling command supplies its path.** This file calls it the **working folder** and writes it `[working_folder]`.

It is **not** the shell's current directory — nothing here ever changes where you're `cd`'d to. It is a specific path inside the repo that your command file names, and it is where the whole run lands:

```
[working_folder]/
  [the plan file]           # written by the manager
  [the consolidated summary] # the deliverable
  [per-unit output]          # one file or one directory per unit
```

Your command file states its own `[working_folder]` — `reviews/projects/[review_name]/` for the review commands, `specs/research/[topic]/` for standalone research — along with whether it is created fresh per run and how name collisions are handled. Resolve it **once**, before writing the plan, and use it everywhere below. Every path in this file is relative to it.

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
- **The model** the units will run on (see [Model Selection](#model-selection)) — the user is approving a spend, and the model is most of it

Every unit also needs a **focus paragraph** — what it covers, and what is explicitly some other unit's job. Where that paragraph lives is a command decision: in the plan file, in the unit's dispatch prompt, or both. Your command file's plan template and prompt templates settle it. If the plan carries the focus paragraphs, sub-agents read the plan to find their lane; if the prompts do, the plan stays a checklist and the prompt is the sub-agent's whole brief.

**After writing the plan, expand the per-unit sub-steps in your progress block to match it** (Step 2a, 2b, 2c… — your command file says which step number). The plan and the progress block should name the same units in the same order.

## Plan Approval

Present the plan and **wait for explicit approval before dispatching anything.** Show the scope, the unit list, the model the units will run on, and where the output will go.

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

## Model Selection

**Default: spawn the units on the same model you're running on** — the user picked it, and it should hold for the whole process. Per [references/spawning_subagents.md](../spawning_subagents.md), don't drop to a cheaper model for speed.

**The one exception is cost, and it exists because this pattern is a multiplier.** One agent's tokens become N agents' tokens, and a fan-out on a very expensive model is the most expensive thing this skill does.

- If you are running on a **very expensive frontier model** (Fable / Astra tier, ~$50/M tokens), **step the units down** to a reasonable model — Opus, GPT Terra, or equivalent.
- Otherwise, **use the current model.**
- **Never step up** to a model more expensive than the user selected.

This is the default for every command built on this pattern, and it covers every agent the fan-out spawns — the unit agents and the collapse agent, when the collapse is delegated. A command may override it; if yours does, its own Model Selection section says so and that wins. Name the resulting model in the plan file, so approval covers it.

## Wait and Re-Dispatch

**Wait for every unit before collapsing.** A summary written over a partial fan-out is worse than no summary — it reads as complete and isn't.

Re-dispatch a unit whose agent errored, returned without writing its output file, or reported it couldn't finish. Give the replacement the same prompt plus a line about the prior attempt, so it builds rather than restarts:

```
A previous agent worked this unit and did not finish. Anything already written under
[unit's output path] is yours to build on or replace.
```

**Cap the attempts at 3** unless your command file says otherwise. Some units genuinely can't be completed — a paywalled source, a file that no longer exists. When one hits the cap, stop retrying, mark it in the progress block, and carry it into the collapse step as a known gap. Never stall the whole run on one unit.

## Collapse

One consolidated summary at the top of the working folder. It is the deliverable — the per-unit files are the depth beneath it.

**Progressive disclosure, one level at a time.** The summary links to the per-unit outputs; those link to whatever detail sits below them. Don't reach past a level: link the unit summary, not the deep doc behind it.

The summary:

- **Leads with the answer.** Someone reading only this file should be able to act.
- **Synthesizes, rather than concatenating.** Cross-unit patterns, conflicts between units, and deduplicated findings are the value the collapse adds. N reports stapled together add nothing.
- **Reports its own gaps** — units that failed or came back thin, and what that leaves unanswered.
- **Has working links.** Verify the paths exist before you finish.

Who writes it is a command decision: the manager, or a **fresh** sub-agent that reads the per-unit outputs. Never a resumed unit agent — its synthesis is colored by having done one slice of the work. Your command file says which, and gives the summary's format.

## Present

Keep it short. Give the user:

- The working folder path and the summary path
- The summary's headline — the overview table, or the bottom line
- Anything that needs attention now (critical findings, failed units)

Don't restate the summary in chat. You just wrote it to a file for a reason.

## Autonomous Flow

**Once the fan-out begins, drive the entire run to completion without stopping for user input. No exceptions.** After each return: update the progress block, then immediately dispatch the next unit. After the last unit: collapse, then present.

The only legitimate pauses are the ones your command file names — plan approval always, plus any scope confirmation before it or interactive phase after it. That is the entire list. Nothing else in this pattern is a stopping point.

## Non-Interactive

Work autonomously. Don't ask the user for help or confirmation during the run.

Once the fan-out is running, keep working until every unit is complete, the summary is written, and the results are presented. Don't stop to ask questions. Don't ask "should I continue?" Don't wait for approval between units. Don't narrate a decision back to the user hoping they'll make it for you. The progress block tells you what to do next — do it.

This is stated twice on purpose, and the repetition is not an accident to be cleaned up. Managers don't break this rule at the start of a run, when the instruction is fresh. They break it in the middle, when a unit comes back with something alarming, ambiguous, or bigger than anyone scoped for — and checking in with the user suddenly feels like the responsible thing to do. It isn't. It strands a half-finished fan-out, wastes every unit already dispatched, and hands the user a decision they can't actually make yet, because the run that would inform it is the thing you just stopped. Alarming findings belong in the summary. Ambiguity gets resolved by finishing the work and looking at all of it. Take the finding, put it in the progress block, dispatch the next unit.

A failed unit is not a pause either. Re-dispatch it per [Wait and Re-Dispatch](#wait-and-re-dispatch). If it hits the attempt cap, mark it a gap, carry it into the collapse, and keep going. A completed run with one honest gap is worth more to the user than a run that stopped to ask about it.
