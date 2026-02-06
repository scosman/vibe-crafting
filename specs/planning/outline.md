# README Outline — Vibe Crafting

## 1. Introduction
- Title: "Vibe Crafting: High Quality App Development without Coding"
- What is vibe crafting? Distinguish from "vibe coding"
  - Not just prompting and hoping — deliberate process, human judgment, AI execution
  - Thesis: with the right process, AI can build production-quality apps — but it still needs you
- What this repo is (blog-post-as-repo, real project artifacts, real prompts)
- What was built (brief description of the iOS app project)
- Compact TOC — one line per major section, sub-links inline. Format:
  > **The Two Tools:** Interactive Agent | Autonomous Agent
  > **The Process:** Overview | Project Overview | Spec & Plan | Build Phases | Review | Testing & Iteration | Code Review | External Review
  > **Lessons Learned:** It Still Needs You | Sycophancy | AI-Driven Manual Testing | Free Polish | Speccing | Projects & Costs | Autonomous Development

## 2. The Two Tools
- Concept: two tools, two levels of attention and intelligence
  - Why two? Cost and interaction mode
  - 90% of important decisions in planning (best model, high user attention)
  - 90% of token usage in coding (cheaper model, low user attention)
- **Interactive Agent** — human-in-the-loop, planning and review
  - High user attention: you're actively collaborating with the agent
  - Best possible model for critical decisions and saving human time
  - IDE-based for easy line-by-line change review, quick edits
  - My toolset: Cursor + Opus
- **Autonomous Agent** — long uninterrupted coding sessions
  - Low attention: agent works independently, you review results
  - Smart but much cheaper model (about ~2% cost of Opus at similar benchmark to Sonnet)
  - Critical: built-in tools for autonomy -- tests, linting, formatting, web search, etc
  - My toolset: Claude Code + GLM 4.7 (z.ai Coder plan) + Web Search MCP + Custom HooksMCP for test/lint/format/ui-tests/etc

## 3. The Process
- ### 3.0 Overview
  - High-level diagram/summary of the full flow (from `overview.md`)
  - Key idea: iterative planning narrows from broad to specific before any code is written

- ### 3.1 Human Writes `project_overview.md`
  - Don't skimp on details, motivations, technical requirements
  - This is the one artifact you write from scratch

- ### 3.2 Build Spec & Implementation Plan [Interactive Agent]
  - Iterative refinement: high-level overview → detailed step-by-step plan
  - Heavy human interaction early, minimal by later steps
  - Prompt: ask it to challenge you — get pushback out of the way early
  - Steps:
    - Step 1: `functional_spec.md`
    - Step 2: `architecture.md`
    - Step 3: Component designs + test plans (`components/[name].md`)
    - Step 4: Implementation ordering — N phases (`implementation_plan.md`)
    - Step 5: Per-phase plans (`phase_plan/phase_N.md`)
  - Still "vibe-speccing" — not reading specs line by line, asking questions and forcing end-to-end thinking

- ### 3.3 Build Phase by Phase [Autonomous Agent]
  - Prompt: "implement phase N" and let agent work
  - `/clear` before each phase — needed context lives in planning files
  - `phase_instructions.md` — a standing set of instructions the agent follows for every phase (link to prompt)
    - Key concept: defines the full autonomous loop so the agent is self-sufficient
    - The loop: read plan → confirm next step → write detailed phase plan if needed → work without assistance until done → build → lint/test/format until clean → self code review → write tests → run tests until passing → mark phase complete → stop
    - Agent can ask questions at start only — after that, works uninterrupted
    - Agent must not exit until all quality gates pass (build, tests, lint, format)
    - This is what makes "low attention" possible: you trust the loop, not the individual steps
  - Manual test plans for UI (`phase_N_manual_test_plan.md`) — written by agent as part of the loop when unit tests aren't possible

- ### 3.4 Review Each Phase
  - Optional: agent phase code review (smart model on `git diff` vs spec)
  - Human: run manual test plan if applicable (back in Interactive Agent)
  - Human: optional quick manual code review of major decisions
  - Check in the phase (git commit)

- ### 3.5 Human Testing & Iteration [Interactive Agent]
  - Use the feature, give feedback, iterate
  - More "vibey" here, but spec keeps it on rails
  - Keep updating spec as you go

- ### 3.6 End-to-End Agentic Code Review [Autonomous then Interactive]
  - Run full agentic code review with Autonomous Agent (link prompt)
  - Interactively review `issue_summary` and fix with Interactive Agent

- ### 3.7 External Code Review
  - Don't open PR until ready for full review (Gemini/CodeRabbit)
  - Continuing human code review — no unreviewed agentic code ships

## 4. Lessons Learned

- ### 4.1 It Still Needs You
  - Technical design flaws the AI pushed for:
    - JSON serialization for hashing (non-deterministic)
    - Relying on iOS background execution
    - Missing cloud storage features (GCS HNS atomic moves)
  - UX design: modals vs nav stacks, overall plan needed human judgment
  - Product design: human owns the "what" and "why"

- ### 4.2 Sycophancy is Real
  - Will do what you say to a fault, even bad ideas
  - Example: notification recurrence design that would have been fragile
  - Mitigation: explicitly ask it to push back (and remember to do so every time)

- ### 4.3 AI-Driven Manual Testing [Interactive Tool]
  - Concept: agent writes manual test plans + diagnostic UI
  - Mostly UI, but also for system integrations (AlarmKit, notifications, etc). Give examples
  - Found genuine bugs (alarm schedules, intents not in system UI)
  - More testing than you'd do manually, and it's repeatable
  - Link: prompt, example test plan, example diagnostic UI

- ### 4.4 More Polish Than You'd Bother With
  - Tiny improvements become free: "make return key advance to next step" — 15 seconds
  - Example from another project: async REST calls instead of threaded SDK calls

- ### 4.5 It's not great at speccing (a bit)
  - Multiple competing sources of truth
  - AI assumptions: iOS background runtime is reliable, non-defensive scheduler
  - Design weak spots

- ### 4.6 Example Projects and Costs
  - Two real projects summarized with token/cost breakdowns
  - **iOS App** — the main project described in this post
    - Token breakdown, model costs, z.ai Coder plan value
  - **Pipeline Project** — a data pipeline / backend project
    - Token breakdown, model costs
  - z.ai Coder plan: ~$30/mo for essentially unlimited (only hit temp limit once)
  - What comparable usage would cost on other models/plans
  - Note on electricity / carbon credits

- ### 4.7 True Autonomous Development With Security
  - The real hard part: autonomous agents need a secure sandbox
  - More time setting up MCP servers to let it work autonomously than working on code. This is okay, one time cost.
  - Sandboxing is essential but painful — agents need to build, test, lint, but you can't just give them full system access
  - Some toolchains work fine: golang, uv+Python.
  - xCode/iOS particularly awful
    - CLIs don't work in sandbox — needed MCP server to bypass for builds/warnings
    - Xcode CLI is second class to UI — occasionally had to run manually
    - Some APIs require physical device testing. Manual testing needed [link].

## 5. Prior Art
- Other spec-driven development tools and approaches
- TODO: research and fill in details

## 6. Conclusion
- Summary: vibe crafting = deliberate process + right tools + human judgment
- When to use this approach / who it's for
- Links to all prompts and artifacts in the repo
