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
### Vibe Coding for Stuff You Care About

I recently built an iOS app without writing a single line of code. I'm an ex-Apple software engineer — I could have written it myself, but I wanted to push the boundary of AI development.

The goal: produce code that's as good or better than I would write, without writing any of it. Zero compromises on architecture or quality. Take as much time as needed to get it right. But offload all the coding.

**TLDR** I ended up developing a process — upfront technical specs, phased autonomous builds, and ample code review (both manual and agentic). It's like vibe coding in that I don't read every line or write any lines, but the AI doesn't get to wing it either. I'm calling it "vibe crafting" for lack of a better term. The result was a codebase better than I would have written — not because I can't, but because I'm lazy. I would never have written 288 tests and a full UI test suite on my own.

This repo contains the actual process I used, along with the real prompts, specs, and planning docs from the project. When I reference a prompt, you can click through and see exactly what I fed the agent.

## Table of Contents

> **[Two Tools:](#two-tools-interactive-vs-autonomous-agents)** Interactive Agent | Autonomous Agent
> 
> **[The Process:](#the-process)** [Project Overview](#human-writes-project_overviewmd) | [Spec & Plan](#build-spec--implementation-plan-interactive-agent) | [Build Phases](#build-phase-by-phase) | [Code Review](#code-review)
>
> **[Lessons Learned:](#lessons-learned)** [It Still Needs You](#it-still-needs-you) | [Sycophancy](#sycophancy-is-real) | [Manual Testing](#ai-driven-manual-testing) | [Free Polish](#more-polish-than-youd-bother-with) | [Speccing](#its-not-great-at-speccing) | [Projects & Cost](#example-projects-and-costs) | [Tools & Sandboxing](#the-hard-part-tools-and-sandboxing)
>
> **[Prior Art](#prior-art)** -- **[Conclusion](#conclusion)**

# Two Tools: Interactive vs Autonomous Agents

The whole process uses two tools — really two modes of working with AI. They're different in cost, attention level, and what they're good at.

> **Key Insight**: About 90% of the important decisions happen during planning. About 90% of the token usage happens during coding. Use two different tools.

## Interactive Agent — Planning and Review

This is the one where you're actually paying attention. You're going back and forth with the agent, reviewing diffs, pushing back on ideas, making design calls.

- High level of user attention, interactive loop
- Best model you can get — the decisions here ripple through everything
- IDE-based, so you can see changes inline and make quick edits
- Used for: spec writing, architecture, code review, iteration

My setup: Cursor + Claude Opus.

## Autonomous Agent — Coding Sessions

This is the opposite. You type "implement phase 4" and walk away. The agent reads the plan, writes code, runs tests, fixes lint errors, and keeps going until everything passes. You come back and review the results.

- No user attention. I prompt 'Implement phase 11' and walk away.
- Smart but much cheaper model. The planning model has hashed out the tough decisions. This is where the bulk of your token spend happens (writing code, writing tests, fixing tests, fixing linting).
- Built-in tools so it can really be autonomous -- build, test, lint, format and web-search for docs.

My setup: Claude Code using GLM 5 on the z.ai Coder plan, web-search MCP (Z.ai), [Hooks MCP](https://github.com/scosman/hooks_mcp) setup with project-specific build, test, lint, format, coverage, and UI test commands. 

> Yes it's ironic I'm using a Claude model in Cursor and non-Claude model in Claude Code. But it works and I like it.

# The Process

Here's the full flow, start to finish:

1. **Write a project overview** — you describe what you want to build, in plain language. Don't skimp on motivation, techical guidelines, and details. [Human]
2. **Build specs** — create detailed functional spec, architecture, component designs with test plans [Interactive Agent]
3. **Build an implementation plan** — order the work into phases, write detailed per-phase plans [Interactive Agent]
4. **For each phase:**
   - **Build** — in a clean agent context, simply prompt "implement phase N" and let it work [Autonomous Agent]
   - **Review** — review code, feedback/fixes/iteration [Interactive Agent]
   - **Manual Testing** [Optional, for UI components] — follow agent-generated test instructions, give feedback, fix what's off [Interactive Agent]
   - **Commit** — commit checkpoint
5. **End-to-end agentic code review** — full codebase review [Autonomous Agent → Interactive Agent]
6. **Code review** — open a PR and review

The key idea: you go broad to specific before any code is written. By the time the coding agent starts, it has a functional spec, architecture doc, component designs with test plans, a phased implementation plan, and [detailed per-phase plans](example_specs/phase13.md). It's not winging it — it's executing a well-defined plan.

This front-loaded planning is where most of the important decisions happen. It's also where the human does most of their work. Once you're in the build loop, you're mostly reviewing output and giving feedback.

## Human Writes `project_overview.md`
`[██████████] Human` · `[░░░░░░░░░░] Agent`

This is the one artifact you write from scratch. Everything else gets built collaboratively with the AI, but this starting document is yours.

Write down what you want to build. What the thing does, who it's for, why you're building it, any technical requirements, any constraints you already know about. Don't skimp. The more context you put here, the better everything downstream gets.

For the iOS app, my project overview covered the core feature set, target user, platform constraints (iOS-only, SwiftUI, specific system APIs I wanted to use), and the non-negotiables (offline-first, no backend). Several pages of details.

This document becomes the seed for everything that follows. It's best if it's complete, but you'll be adding detail over the next few steps so you can recover if you miss something.

## Build Spec & Implementation Plan [Interactive Agent]

This is where most of the real thinking happens. You're going to iteratively refine from a broad project overview into a detailed, step-by-step build plan. Five steps, each one more specific than the last.

The human interaction curve is steep at the start and tapers off. Steps 1 and 2 involve a lot of back-and-forth — you're making architecture calls, pushing back on suggestions, asking "what about X?". By steps 4 and 5, the agent mostly has what it needs and you're reviewing output rather than co-authoring it.

I'd call this "vibe-speccing." You're not reading every line of the spec like a legal contract. You're asking questions, forcing the agent to think end-to-end, and checking that the big decisions feel right. The specs are a collaboration — the AI drafts, you steer.

### The Five Steps

**Step 1: Functional Spec** (`functional_spec.md`)
`[█████░░░░░] Human` · `[░░░░░█████] Agent`
Start from your project overview and build out a full description of what the app does. Features, user flows, edge cases. The agent drafts, you iterate until it captures what you actually want. ([spec_guide.md](prompts/spec_guide.md))

**Step 2: Architecture** (`architecture.md`)
`[█████░░░░░] Human` · `[░░░░░█████] Agent`
High-level technical design — data model, key components, how they interact. This is where you make the big calls: what frameworks, what patterns, what tradeoffs. ([spec_guide.md](prompts/spec_guide.md))

**Step 3: Component Designs** (`components/[name].md`)
`[██░░░░░░░░] Human` · `[████████░░] Agent`
Individual specs for each component, including public interfaces, design patterns, API usage, and test plans. For UI components, I used a separate pass with its own prompt. ([spec_guide.md](prompts/spec_guide.md), [ui_spec_guide.md](prompts/ui_spec_guide.md))

**Step 4: Implementation Plan** (`implementation_plan.md`)
`[█░░░░░░░░░] Human` · `[█████████░] Agent`
Order the work into phases. What gets built first, what depends on what. Each phase should be roughly one component — small enough to review in one sitting. ([implementation_plan_prompt.md](prompts/implementation_plan_prompt.md))

**Step 5: Per-Phase Plans** (`phase_plan/phase_N.md`)
`[█░░░░░░░░░] Human` · `[█████████░] Agent`
Detailed plans for each phase, pulling relevant details from the spec into a self-contained reference. This is what the coding agent actually reads when it starts a phase — it shouldn't need to go back to the full spec. ([build_phase_plans.md](prompts/build_phase_plans.md), [example plan](example_specs/phase13.md))

### Challenge the AI Early

One thing I do in steps 1 and 2: challenge the plan and refine details early. This is a two-way street:
 - Challenge AI decisions, propose alternatives, fix its mistakes
 - Explicitly ask the agent to challenge my assumptions. Push back on my ideas. Tell me what's wrong with my approach. Suggest alternatives. Keep pushing for feedback on the top issues, until its feedback feels like nits you can skip.

This matters because AI agents are [sycophantic by default](#sycophancy-is-real) — they'll agree with whatever you say and build exactly what you asked for, even if it's a bad idea. If you ask for pushback early, you surface problems before they're baked into the spec. Much cheaper to fix a bad assumption in step 1 than to rewrite three components in step 4.

## Build Phase by Phase

For each phase, the loop is: build autonomously, review, test, iterate, commit. Here's what each step looks like.

### Build [Autonomous Agent]
`[░░░░░░░░░░] Human` · `[██████████] Agent`

The prompt is almost comically simple: "implement phase N." That's it. The agent reads the phase plan, knows what to build, and gets to work. You walk away.

Before each phase, clear the agent's context (`/clear` in Claude Code, or start a new session). Everything the agent needs is in the planning files — the phase plan is self-contained by design. A clean context means the agent isn't confused by stale conversation history from three phases ago.

The real magic isn't the prompt — it's the standing instructions the agent follows every time. I have a [`CLAUDE.md`](prompts/CLAUDE.md) that points to [`phase_instructions.md`](prompts/phase_instructions.md), which defines the full autonomous loop: read the plan, ask clarifying questions (only at the start), build, lint and format until clean, self code review, write tests, run tests until passing, mark phase complete, stop.

That loop is what makes "low attention" possible. You're not trusting the agent on every individual step — you're trusting the loop. The agent must not exit until all quality gates pass. If tests fail, it iterates. You come back to a clean result, or a stuck agent you can help.

If it ever gets really stuck, you probably have a spec problem. Throw it out, improve your spec, and start again. (More on why in [It's Not Great at Speccing](#its-not-great-at-speccing).)

### Review [Interactive Agent]
`[█████░░░░░] Human` · `[██░░░░░░░░] Agent`

Every phase gets a quick review before you move on. Check the code it generated, check the list of tests, make sure it looks good. Not line-by-line — that comes later in the end-to-end review.

For substantial phases, I'll feed the `git diff` and phase plan to the Interactive Agent and ask it to check the implementation against the spec. You're using the smarter model to code-review the cheaper model. It's also good for catching structural mismatches.

### Agent-Led Manual Testing (UI Only) [Interactive Agent]
`[█████░░░░░] Human` · `[█████░░░░░] Agent`

Not everything can be unit tested. UI components, system API integrations (notifications, alarms, live activities) — these need human help. When unit tests aren't sufficient, the agent writes a manual test plan as part of the phase ([manual_test_plan_guide.md](prompts/manual_test_plan_guide.md)). It creates a diagnostic view in the app and writes step-by-step instructions: "tap this button, verify this appears, check that the notification fires." 

See [AI-Driven Manual Testing](#ai-driven-manual-testing) in Lessons Learned for more on how this works and what it caught.

### Commit

One commit per phase. Keeps the history clean and makes it easy to bisect if something breaks later.

## Code Review
`[█████░░░░░] Human` · `[█████░░░░░] Agent`

Once all phases are built, I run a full codebase review — not a per-phase diff, but a comprehensive audit of the entire codebase against the spec.

### End-to-End Agentic Review [Autonomous Agent]

The prompt ([agentic_cr.md](prompts/agentic_cr.md)) tells the agent to read the full spec, create a review plan, go through it systematically producing several smaller CR reports, and produce an `issues_summary.md` labeled by severity. The instruction is explicit: "be critical like a senior architect." 

This runs autonomously — kick it off and come back to the summary. Then switch to the Interactive Agent: read through the findings, decide what's real, fix what matters. Some findings are noise, but some are genuinely good catches — architectural inconsistencies, missing error handling, tests that don't actually test what they claim to. 

### External Agent Code Review

I also run external reviews: Gemini code review and CodeRabbit on pull requests for an additional look.  Don't open the PR until you're actually ready — running review tools on half-finished code just drowns you in noise. In general you should stump these, and get only nits back if your last review was any good.

### Human Code Review

We still do a full human code review pass before shipping. This isn't vibe-coding, we want to know what we've build and stand by it.

# Lessons Learned

## It Still Needs You

The process works, but AI still gets things wrong — sometimes confidently, sometimes subtly. Here's where I had to step in.

**Technical design.** Three examples from the iOS app:

The agent wanted to JSON-serialize objects and hash the result to generate stable IDs. JSON serialization isn't deterministic — key ordering can vary between runs, across platforms, or after library updates. I caught this during spec review and pushed for a different approach. The agent agreed immediately, which is both nice and a little unnerving (more on that below).

It assumed iOS background execution was reliable. Built the scheduler around the idea that the app could do meaningful work in the background on a consistent schedule. That's not how iOS works. Background execution is best-effort — the system can throttle or kill it at any time. I had to flag this and redesign the approach.

For a pipeline project using Google Cloud Storage, I needed atomic batch file moves. The agent didn't know about GCS Hierarchical Namespace (HNS), a relatively new feature that supports this. I pointed it to the docs and it picked it up immediately. But without me knowing HNS existed, it would have built a fragile workaround.

**UX design.** The AI is great at implementing UI details — a nicely formatted table cell, a smooth animation, proper loading states. But overall navigation structure needed me. When to use a modal vs. a navigation push, how screens flow together, what the information hierarchy should be — those are judgment calls that require understanding what the user is actually trying to do. Best approach: let the agent take a first pass, then fix what feels wrong rather than over-specifying upfront.

**Product design.** The agent doesn't know why you're building the thing. It can build what you ask for, but deciding *what* to build — the "what" and "why" — is still entirely on you.

## Sycophancy is Real

AI agents will do what you tell them. To a fault. Even if it's a bad idea.

In my spec, I had a one-line requirement about notification recurrence. The agent dutifully designed an entire system around it — one that would have been fragile in both code and reliability. It never once said "hey, this is going to be a pain and might not work well." It just built what I asked for.

The model wants to be helpful, which means agreeing with you and executing your plan. It won't push back unless you explicitly ask.

The mitigation is simple but easy to forget: tell the agent to [challenge your assumptions](#challenge-the-ai-early). "Push back on my ideas. Tell me what's wrong with my approach. Suggest alternatives." When you do this, it actually gives good feedback — it just won't volunteer it.

The catch: you have to remember to do this every time. The one time you forget is probably the time you needed it most.

## AI-Driven Manual Testing

Not everything can be unit tested. UI flows, system integrations (AlarmKit, notifications, live activities), hardware-dependent features — these need human eyes and hands. But the AI can still do most of the work.

The concept: when a phase includes UI or system integration work, the agent writes a manual test plan as part of its build loop. It creates a diagnostic view inside the app — a hidden screen with buttons to trigger specific behaviors, display internal state, and exercise edge cases. Then it writes step-by-step test instructions: "tap this button, verify this label shows the next alarm time, check that the notification fires after 30 seconds."

This found real bugs. Alarm schedules were being set incorrectly — off by a time zone conversion. Intents weren't showing up in the system UI because of a missing Info.plist entry. These are exactly the kinds of things unit tests can't catch.

The best part: I ended up doing more testing than I would have on my own. Writing test plans is tedious, so normally I'd test the happy path and move on. Here, the agent wrote thorough plans covering edge cases I would have skipped, and I just followed the instructions. Repeatable test plans as a free side effect of the build process.

(Prompt: [manual_test_plan_guide.md](prompts/manual_test_plan_guide.md).)

## More Polish Than You'd Bother With

Small improvements become free. Not free-as-in-no-cost — the tokens cost something — but free in terms of your time and attention.

"Make it so the return key on the keyboard advances to the next input field." That's a 15-second prompt. In a normal project, I'd add it to a backlog and probably never get to it. Here, it's done before I finish my coffee.

On another project (a data pipeline in Python), the Google Cloud SDK didn't support async operations... in 2026... come on, folks. The agent rewired the integration to use async REST API calls instead of the threaded SDK approach. That's not a trivial change — it touched the HTTP layer, error handling, retry logic — but from my perspective it was one prompt and a code review. The codebase is faster and better for it.

You end up with a more polished product because the marginal cost of small improvements drops to near zero.

## It's Not Great at Speccing

The AI can draft specs, but it's not great at designing systems from scratch. A few patterns I noticed:

**Multiple competing sources of truth.** As the spec grew across files (functional spec, architecture, component designs, phase plans), details would drift. A decision in the architecture doc wouldn't fully propagate to component specs. The agent would sometimes reference an outdated assumption from an earlier conversation. Keeping everything consistent was ongoing work.

**Confident assumptions about things it hasn't verified.** The iOS background runtime example is a good one — the agent assumed it worked reliably because the API exists. It didn't think to check the real-world constraints. Similarly, it designed a non-defensive scheduler: no fallback for missed executions, no recovery path. These are design decisions that come from experience shipping software, not from reading documentation.

**Design weak spots.** When the agent had to make genuine design tradeoffs — not "which pattern fits here" but "what's the right behavior when two requirements conflict" — it would pick the most obvious path without flagging the tradeoff. You have to actively probe for these.

This is why the process [front-loads human attention in the planning phase](#build-spec--implementation-plan-interactive-agent). The specs are a collaboration, not a delegation.

## Example Projects and Costs

Two real projects, to give a sense of scale.

**iOS App** — the main project described in this post. Offline-first iOS app in SwiftUI with notifications, alarms, live activities, and a full test suite.

- ~185M tokens during the coding phase (autonomous agent)
- On the z.ai Coder plan: **$30/month** for essentially unlimited usage. I only hit the temporary rate limit once across the entire project.
- The same token volume on Sonnet would cost roughly **$2,400**
- The same on GLM 4.7 API pricing: **~$320**

**Pipeline Project** — a data pipeline / backend project in Python with Google Cloud integrations. Similar pattern: bulk of the tokens in the coding phase, minimal spend during planning.

The z.ai Coder plan is what made this practical for me. $30/month for what would otherwise be hundreds or thousands in API costs. Your mileage will vary depending on model choice and provider, but the key insight is: separate your planning model (best available, low token volume) from your coding model (good enough, high token volume).

> One last note: all those tokens running on GPUs use electricity. Carbon credits exist and are cheap. Worth considering if you're burning through millions of tokens regularly.

## The Hard Part: Tools and Sandboxing

The hardest part of this process wasn't the AI or the prompts — it was the tooling setup so the agent could actually work autonomously.

Autonomous agents need to build, test, lint, and format code. But you can't just give them unrestricted system access. Sandboxing is essential — you don't want an AI agent with free reign over your filesystem, network, and credentials. The tension: the agent needs enough access to do its job, but not so much that a hallucinated `rm -rf` ruins your day.

I spent more time configuring MCP servers for autonomous tooling for the iOS project than I spent on any individual coding phase. Build commands, test runners, linters, formatters — each one needed to be exposed to the agent in a controlled way. This is a one-time cost per project, but it's real.

**Some toolchains just work.** Go and Python (with uv) were painless. The CLIs work in sandboxed environments, the build and test tools are straightforward, everything's fine.

**Xcode is pain.** iOS development was a different story.

- Xcode's CLI tools (`xcodebuild`, etc.) don't work properly in a sandboxed environment. I had to write a custom MCP server just to bypass the sandbox for builds and compiler warnings.
- Xcode's CLI is second-class to its GUI. Occasionally the agent would get stuck in a loop, and I'd have to open Xcode manually to clear the state.
- Some iOS APIs — AlarmKit, certain notification types — require a physical device. Simulator testing wasn't sufficient. This is where the manual test plans became essential.

The tooling story will get better as the ecosystem matures. Right now, budget extra setup time if your toolchain isn't natively CLI-friendly.

# Prior Art

I iterated my way to this process, but I'm not the only one thinking about spec-driven agentic development. A few tools have emerged around the same ideas:

- [Spec Kit](https://speckit.org) — makes specifications executable: define requirements, generate plans, auto-generate implementations.
- [GSD](https://github.com/gsd-framework) — a "get shit done" framework for structured agentic workflows with spec-first planning.
- [Traycer](https://traycer.ai) — a commercial version of spec dev

I personally just use a few homegrown prompts and markdown files, but these may be worth exploring if you're adopting this kind of approach.

# Conclusion

I'm not writing code anymore, so you'd be tempted to call it vibe coding. But I'm going deeper into architecture than I normally would, writing more tests, running more manual tests, and producing better code than when I was constrained by my typing speed and focus. The results are genuinely better — not just faster, but higher quality.

288 unit tests. A full UI test suite. Manual test plans for every system integration. Thorough code review from multiple angles. I would never have done all of that on my own. Not because I don't know how — because I'm lazy and there are only so many hours. With this process, the marginal cost of doing it right dropped to near zero.

Vibe crafting is a silly name, but it gets the idea across. I'm spending more time steering and reviewing results than coding. But the process underneath is deliberate, structured, and uncompromising on quality.

## All Prompts and Artifacts

Every prompt referenced in this post is a real file from various projects I built. Here's the full set:

**Planning (Interactive Agent):**
- [spec_guide.md](prompts/spec_guide.md) — Prompt for building functional specs, architecture, and component designs
- [ui_spec_guide.md](prompts/ui_spec_guide.md) — Prompt for UI-specific component specs
- [implementation_plan_prompt.md](prompts/implementation_plan_prompt.md) — Prompt for creating the phased implementation plan
- [build_phase_plans.md](prompts/build_phase_plans.md) — Prompt for writing detailed per-phase plans

**Building (Autonomous Agent):**
- [phase_instructions.md](prompts/phase_instructions.md) — The autonomous loop: full instructions the coding agent follows for every phase

**Testing & Review:**
- [manual_test_plan_guide.md](prompts/manual_test_plan_guide.md) — Prompt for generating manual test plans and diagnostic UI
- [agentic_cr.md](prompts/agentic_cr.md) — Prompt for end-to-end agentic code review

**Example Artifacts:**
- [example phase plan](example_specs/phase13.md) — A real per-phase plan from the pipeline project, showing the level of detail the coding agent works from

Note: this blog post was vibe-crafted! See the [specs](/specs/) folder for how it was built.
