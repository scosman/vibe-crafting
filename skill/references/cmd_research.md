# `/spec research` — Research a Topic

Research a topic on the web. Divides the topic into subtopics, dispatches a research sub-agent per subtopic, and produces a tree of research documents under a single summary.

Use it standalone when you need to understand something before deciding what to build, or as a sub-step of `/spec new_project` when a spec depends on knowledge neither you nor the user has yet (a standard, an API, a library's real capabilities, prior art).

## Manager Role

**You are a manager. You do NOT research the topic, run web searches, or write research documents — ever.** If you catch yourself about to search the web or draft findings, stop. You are in the wrong role. Your only tools are: checking which tools are available, spawning sub-agents, writing the research plan file, and outputting progress blocks.

The manager's responsibilities:
- Verify web access exists before promising research
- Decompose the topic into subtopics
- Write `research_plan.md`
- Get user approval for the plan (scope *and* cost)
- Spawn one research sub-agent per subtopic, and re-dispatch any that error
- Spawn the summary sub-agent
- Present results to the user

**Exception to read-only:** The manager writes `research_plan.md`. That is an orchestration artifact, not research. All actual research is delegated to sub-agents.

**Important:** even if asked to research something by the user, default to using sub-agents per these instructions, unless the user specifically requests you do it in this context. You are a manager: delegate.

## The Pattern

This command is an instance of the shared fan-out pattern: scope → plan → fan out one sub-agent per unit → wait → collapse into one summary.

→ Read [references/shared/fan_out_pattern.md](shared/fan_out_pattern.md) for those mechanics — unit sizing, the plan artifact, plan approval, dispatch and the per-return loop, re-dispatching failures, and how the collapse works. Follow them precisely.

This file supplies the research specifics: the unit is a **subtopic**, the plan is `research_plan.md`, the consolidated summary is `summary.md`, both at the root of the research folder — and three things the other fan-out commands don't have:

- a **web-access precondition** (Step 0) that can stop the command before it starts
- a **cost dimension to plan approval** (Step 2), because web tools bill per call
- a **model carve-out** for the fan-out (Step 3), for the same reason

Two more differences from deep CR and design crit: **the collapse is done by a fresh sub-agent** rather than the manager (see Step 4), and the **focus paragraphs live in the plan as well as the dispatch prompts** — subtopic agents read `research_plan.md` to see where their lane ends and what the neighbouring subtopics own.

## Progress Tracker

→ Read [references/shared/progress_tracker.md](shared/progress_tracker.md) for the progress block format, round counters, and rules. Follow them precisely.

Use the label **"Research Progress"** for the progress block. The full step list for this command:

```
- Step 0: Web access check
- Step 1: Plan
- Step 2: Approval
- Step 3a: Subtopic 1 ([name])
- Step 3b: Subtopic 2 ([name])
- Step 3c: Subtopic 3 ([name])
  ... (one sub-step per subtopic, determined during Step 1)
- Step 4: Summary
- Step 5: Present
```

Example mid-flow:

```
<progress>
Research Progress:
- [x] Step 0: Web access check — complete (WebSearch + WebFetch)
- [x] Step 1: Plan — complete (4 subtopics)
- [x] Step 2: Approval — approved
- [x] Step 3a: Subtopic 1 (Spec and versions) — complete (6 docs)
- [ ] Step 3b: Subtopic 2 (Reference implementations) — in progress
- [ ] Step 3c: Subtopic 3 (Ecosystem adoption) — pending
- [ ] Step 3d: Subtopic 4 (Migration risks) — pending
- [ ] Step 4: Summary — pending
- [ ] Step 5: Present — pending
</progress>
```

Research does NOT write to `.specs_skill_state/current_project.md`. It is not resumable active work, and writing there would clobber the user's active project or task.

## Invocation

```
/spec research [topic]
```

Aliases: `/spec research` (with or without a colon before the topic).

If invoked without a topic, ask the user what they want researched.

The command also runs embedded in `/spec new_project` — see [Embedded Use](#embedded-use).

## Research Folder

Everything for one research run lives under a root research folder:

| Invocation | Root folder |
|---|---|
| Standalone `/spec research [topic]` | `specs/research/[topic]/` |
| Inside `/spec new_project` | `specs/projects/PROJECT_NAME/research/[topic]/` |

The root folder is the only thing that differs between the two. Everything below is identical.

```
[topic]/
  research_plan.md          # written by the manager (Step 1)
  summary.md                # written by the summary sub-agent (Step 4)
  [subtopic]/
    summary.md              # written by that subtopic's sub-agent
    [any deep docs it wants].md
  [subtopic]/
    ...
```

Three levels of progressive disclosure: the top-level `summary.md` links to subtopic summaries, which link to the deep docs.

Note the naming: **subtopic summaries live at `[topic]/[subtopic]/summary.md`**, and the single cross-subtopic summary at `[topic]/summary.md`. Don't collapse those two.

Slugify the topic and subtopic names for the directory names (lowercase, hyphens, no slashes). On collision with an existing research folder, append `-v2`, `-v3`, etc. — don't overwrite prior research.

## Step 0: Web Access Check

**Do this first, before planning anything.** Research means reading the web. If you can't, you can't do research.

Check that you have both capabilities available in this session:

1. **Web search** — any provider: the built-in `WebSearch`, a Tavily or other MCP search server, or an equivalent.
2. **Web fetch/read** — any provider: the built-in `WebFetch`, an MCP extract/crawl tool, or an equivalent.

If **either** is missing, stop and tell the user:

> I don't have web access in this session — research needs both a web search tool and a web fetch/read tool, and I'm missing [which one]. I can't run `/spec research` without them. You can enable web tools (or an MCP search server like Tavily) and re-run, or we can proceed without research.

Then stop. **Do not fall back to answering from model knowledge and call it research.** Recalled knowledge is stale, unsourced, and the user asked for research precisely because they wanted neither.

Note which tools you found — you'll name them for the sub-agents in Step 3.

## Step 1: Plan

Decompose the topic into subtopics, per [unit sizing](shared/fan_out_pattern.md#designing-the-units) in the shared pattern. Typically 2–6 — each one is an agent's worth of searching and reading, so prefer fewer, meatier subtopics.

Write `[root]/research_plan.md`:

```markdown
# Research Plan: [Topic]

## Goal

[1-3 sentences: what the research is for, what decision it feeds. If embedded in a project, name the project and which spec step is waiting on it.]

## Subtopics

- [ ] [Subtopic 1 name] — [one-line description]
- [ ] [Subtopic 2 name] — [one-line description]
- ...

## Focus Details

### [Subtopic 1 name]

[One-paragraph focus description: questions to answer, scope boundaries — including what belongs to a different subtopic.]

### [Subtopic 2 name]

...
```

## Step 2: Approval

Present the plan and ask for approval:

> Research plan for **[topic]** — [N] subtopics, one sub-agent each:
>
> [Subtopic list from research_plan.md]
>
> Output goes to `[root folder]`. Web search and fetch tools aren't free, so this has a real cost.
>
> Proceed?

**Approval is the default and is always required when `/spec research` is invoked directly.** It covers the *form* (are we researching the right things?) and the *cost* — unlike the other fan-out commands, this one spends money per unit, so say so and let the user weigh it.

**The single exception:** the user has explicitly asked for this work to run without being asked questions. In that case, write the plan, state it in your progress output, and proceed without stopping. Being mid-flow in another command is *not* an exception — see [Embedded Use](#embedded-use).

## Step 3: Dispatch Subtopic Agents

One fresh sub-agent per subtopic, dispatched and tracked per the [fan-out pattern](shared/fan_out_pattern.md#fan-out), using the Research Sub-Agent Prompt template below. Each writes to its own directory and ends with `[root]/[subtopic]/summary.md`; the per-subtopic result you record in the progress block is its bottom line and doc count.

### Model Selection

This is the one place in the skill where you may *not* simply inherit the current model.

The default remains [references/spawning_subagents.md](spawning_subagents.md): spawn sub-agents with the same model you're using. **The carve-out is cost.** Research fans out across several agents, each burning tokens on search results and fetched pages — the most token-hungry work in this skill.

- If you are running on a **very expensive frontier model** (Fable / Astra tier, ~$50/M tokens), **step the sub-agents down** to a reasonable model — Opus, GPT Terra, or equivalent.
- Otherwise, **use the current model**, per the normal rule.
- Never step *up* to a more expensive model than the user selected.

This applies to every agent this command spawns — the subtopic agents **and** the Step 4 summary agent, which reads a lot of prose and is the second-most token-hungry spawn in the flow. It does not change how any other command spawns agents.

## Step 4: Summary

Wait for every subtopic and re-dispatch failures per the [fan-out pattern](shared/fan_out_pattern.md#wait-and-re-dispatch) — a subtopic isn't done until `[root]/[subtopic]/summary.md` exists. Carry any subtopic that hits the attempt cap forward as a known gap.

Then dispatch the summary sub-agent using the Summary Sub-Agent Prompt template below, naming those gaps. Here the collapse is **delegated**: a fresh sub-agent reads all the subtopic outputs and writes `[root]/summary.md`. The manager doesn't write it — research summarizing means reading a lot of prose, and that's exactly the context the manager is trying not to hold.

It is a fresh spawn, never a resumed subtopic agent.

## Step 5: Present

Per the [fan-out pattern](shared/fan_out_pattern.md#present). Show the root folder, the path to `[root]/summary.md`, a one-line result per subtopic, and anything that came back thin or failed.

If embedded in another command: return to that command's flow and feed the summary into the step that was waiting on it.

## Autonomous Flow

→ [fan-out pattern: Autonomous Flow](shared/fan_out_pattern.md#autonomous-flow) and [Non-Interactive](shared/fan_out_pattern.md#non-interactive). Read both — a run that stops halfway to ask a question is the failure mode they exist to prevent.

**Work autonomously once the plan is approved. Don't ask the user for help or confirmation during the research.** A subtopic that comes back thin or contradicts another one is summary material, not a reason to stop.

This command pauses at exactly one point: **Step 2**, plan approval — unless the user has waived questions entirely, in which case it doesn't pause at all. Step 0 can stop the command outright, but that's a halt, not a pause.

## Embedded Use

When research runs as part of another command (today: `/spec new_project`), it runs **split**:

- **Steps 0–2 run in the calling context** — the main thread the user is talking to. The user sees the plan and approves scope and cost there.
- **Step 3 onward is dispatched from that same context.** The calling context spawns the subtopic agents and the summary agent directly.

Do **not** wrap this whole command in a single "research manager" sub-agent. That would bury the plan approval behind a handoff and add a second layer of agents for no benefit. One layer of sub-agents, plan approval visible to the user.

The "Research Progress" block still applies while research runs, even though the calling command may not be progress-tracked itself. It starts at Step 0 and ends at Step 5, then the calling command resumes.

Being mid-flow in `/spec new_project` does not waive plan approval — an interactive speccing session is exactly where the user wants to see what's about to be researched.

## Prompt Templates

These are the exact prompts the manager sends to sub-agents. Use them verbatim, filling in the bracketed values.

### Research Sub-Agent Prompt

```
You are a research sub-agent researching one subtopic.

**Topic:** [topic]
**Your subtopic:** [subtopic name]
**Focus:** [one-paragraph focus description from research_plan.md]
**Your directory:** [root]/[subtopic]/
**Research plan:** [root]/research_plan.md
**Web tools available:** [names of the search and fetch tools found in Step 0]

Read `references/research_agent_prompt.md` for your full instructions. Follow them precisely.

Write your findings to your directory, ending with [root]/[subtopic]/summary.md.

Return a short summary: subtopic name, what you found, how many docs you wrote, any gaps.
```

For a re-dispatch after a failure, append the re-dispatch note from the [fan-out pattern](shared/fan_out_pattern.md#wait-and-re-dispatch), with `[root]/[subtopic]/` as the output path.

### Summary Sub-Agent Prompt

```
You are a research summary sub-agent. Several agents researched subtopics of one topic;
your job is the single cross-subtopic summary.

**Topic:** [topic]
**Research root:** [root]/
**Research plan:** [root]/research_plan.md
**Subtopics:** [list of subtopic names and their directories]
**Gaps:** [any subtopic that failed or came back thin, or "None"]

Read `references/research_summary_prompt.md` for your full instructions. Follow them precisely.

Write the cross-subtopic summary to [root]/summary.md.

Return a short summary: the headline findings and anything the research could not answer.
```

## References

- [references/shared/fan_out_pattern.md](shared/fan_out_pattern.md) — The shared plan → fan out → collapse mechanics
- [references/spawning_subagents.md](spawning_subagents.md) — How to spawn sub-agents
- [references/research_agent_prompt.md](research_agent_prompt.md) — Full instructions for subtopic research sub-agents
- [references/research_summary_prompt.md](research_summary_prompt.md) — Full instructions for the summary sub-agent
- [references/cmd_new_project.md](cmd_new_project.md) — Where research is used during planning
