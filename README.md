```
██╗   ██╗██╗██████╗ ███████╗
██║   ██║██║██╔══██╗██╔════╝
██║   ██║██║██████╔╝█████╗  
╚██╗ ██╔╝██║██╔══██╗██╔══╝  
 ╚████╔╝ ██║██████╔╝███████╗
  ╚═══╝  ╚═╝╚═════╝ ╚══════╝
 ██████╗██████╗  █████╗ ███████╗████████╗██╗███╗   ██╗ ██████╗
██╔════╝██╔══██╗██╔══██╗██╔════╝╚══██╔══╝██║████╗  ██║██╔════╝
██║     ██████╔╝███████║█████╗     ██║   ██║██╔██╗ ██║██║  ███╗
██║     ██╔══██╗██╔══██║██╔══╝     ██║   ██║██║╚██╗██║██║   ██║
╚██████╗██║  ██║██║  ██║██║        ██║   ██║██║ ╚████║╚██████╔╝
 ╚═════╝╚═╝  ╚═╝╚═╝  ╚═╝╚═╝        ╚═╝   ╚═╝╚═╝  ╚═══╝ ╚═════╝ 
```

# Vibe Crafting
### An agent skill for spec-driven development

`/spec` is an agent skill that runs a full spec-driven build process: upfront specs written with you, phased autonomous builds, and layered code review. You make the decisions; the agent does the drafting, the building, and the first few passes of review.

It's the process I use to ship code I actually care about — an iOS app, Mac apps, a Python data pipeline, Git sync engines — without writing the code myself, and without compromising on architecture or quality.

[Install it](#install) and you get [ten `/spec` commands](#commands) (plus a bare `/spec` router) in the agent of your choice.

**The full story** — how the process evolved, where the AI still gets things wrong, what it costs, and the sandboxing/tooling pain — is in the blog post: **[Vibe Crafting: Vibe Coding for Stuff You Care About](https://scosman.net/blog/vibe_crafting)**.

## Quickstart

[Install the skill](#install) into the coding agent of your choice. Then, the shortest real path from nothing to shipped:

```
/spec setup                  # once per repo
/spec new_project            # the long, interactive part — expect real back-and-forth
/spec implement all          # walk away; it builds every phase, reviewing and committing as it goes
/spec deep cr                # full-branch review once the phases are done
/spec pr                     # after you open the PR: pulls review comments and fixes them
```

A few things that matter more than they look:

**Write the project overview yourself.** It's the one artifact that's entirely yours, and everything downstream inherits its quality. Motivation, technical constraints, non-negotiables, the stuff you already know. Don't skimp.

**Ask the agent to push back, early.** Agents are sycophantic by default — they'll build exactly the bad idea you described without ever mentioning it's a bad idea. The skill bakes pushback into the planning steps, but the first two steps are where challenging assumptions is cheapest.

**Don't let the coding agent stop to ask permission.** `/spec implement all` is only useful if it actually runs unattended. I run in cloud sandboxes in yolo mode, or locally in a sandbox with `claude --permission-mode=dontAsk`. Give it build, test, lint, and format tools it can call on its own.

**If a phase gets badly stuck, you have a spec problem.** Throw the code out, fix the spec, run the phase again. That's cheaper than debugging your way out.

## Commands

| Command | Aliases | What it does |
|---|---|---|
| `/spec` | — | Router. Reads current state and suggests the next action. Also interprets open-ended requests. |
| `/spec setup` | — | One-time per-repo setup. Adds `.specs_skill_state/` and `reviews/` to `.gitignore`, creates `/specs/projects/`, detects monorepo layout, checks for missing config. Idempotent. |
| `/spec new_project` | `/spec new` | Plan a new project from scratch: project overview, functional spec, UI design (if relevant), architecture, component designs, implementation plan. Sets it as the active project. |
| `/spec continue` | `/spec cont` | Resume the active project. Shows current state and routes to the next logical action. |
| `/spec implement` | `/spec impl` | Build the active project. `implement next` for one phase, `implement all` for every remaining phase, `implement phase N` for a specific one. Each phase runs code → review → commit. |
| `/spec task` | — | A one-off change without the full spec process. Describe it inline, get the same implement loop (code → review → commit) with no planning artifacts. |
| `/spec research` | — | Research a topic on the web. Splits it into subtopics, runs a sub-agent per subtopic, writes a tree of research docs under one summary. Needs web search + fetch tools. Runs standalone or inside `new_project`. |
| `/spec cr` | `/spec code_review` | Fast, spec-aware code review of the current diff (or a scope you name). Always runs in a sub-agent with clean context. |
| `/spec deep cr` | `/spec deep review` | Multi-phase agentic review of the whole branch against its fork point. Designs review phases for the diff, runs focused sub-agents in parallel, writes persistent review artifacts. |
| `/spec design crit` | `/spec crit` | The same treatment for spec and design docs, before any code exists. Ends with an interactive pass to triage and fix what it found. |
| `/spec pr` | — | Address GitHub PR feedback. Finds the PR for the branch, pulls unresolved comments, fixes them through the standard CR loop, commits, pushes, and replies to each thread. |

Full details for each command live in [`skill/references/`](skill/references), one file per command. The skill loads them on demand.

## Install

The skill is a [standard agent skill](https://agentskills.io/home): a `SKILL.md` plus a `references/` folder. The **directory name is the command name**, so the contents of `skill/` need to land in a directory called `spec`.

For Claude Code, user-level (available in every repo):

```bash
git clone https://github.com/scosman/vibe-crafting.git
mkdir -p ~/.claude/skills/spec
cp -R vibe-crafting/skill/. ~/.claude/skills/spec/
```

Or project-level, checked into a single repo (available to anyone who clones it):

```bash
mkdir -p .claude/skills/spec
cp -R /path/to/vibe-crafting/skill/. .claude/skills/spec/
```

Either way you should end up with `SKILL.md` at `<skills-dir>/spec/SKILL.md`. Start a new session and `/spec` should be available.

Other agents that support the skill format work too — drop the same `spec/` directory wherever that tool looks for skills.

Then, once per repo:

```
/spec setup
```

## What else is in this repo

- [`skill/`](skill) — the skill itself. This is the thing you install.
- [`prompts/`](prompts) — the hand-rolled prompt files that predate the skill, from before any of this was packaged. The blog post references them as examples; the skill supersedes them.
- [`example_specs/phase13.md`](example_specs/phase13.md) — a real per-phase plan, so you can see what the coding agent is actually handed.
- [`specs/`](specs) — the specs used to build this repo, including the blog post and several of the skill's own features. It's self-demonstrating: `/spec` was built with `/spec`.
