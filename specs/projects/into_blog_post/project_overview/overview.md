**Vibe Crafting Formula:**

Two Tools:
- Cursor + Opus: 
  - Usage style: Human interactive, IDE to view changes easily. Generally planning and review phases.
  - Model: Best possible model (Opus for me right now)
- Claude Code + GLM 4.7
  - Usage style: long coding sessions without human interaction. 
  - Model: GLM 4.7 on z.ai Coder plan. Very good, but roughly 2% the cost of Opus when you subscribe.
  - Essential: great tools for tests, linting, formatting.
- Why two? Cost and user interaction mode.
  - 90% of important decisions are made in planning with most intelligent model, and more human attention.
  - About 90% of token usage is during coding, testing and refactoring. This uses smart but much less expensive model (GLM 4.7)

Process:
- Human writes `project_overview.md` 
  - Don’t skimp on details, motivations, and technical requirements.
- Build Spec and Implementation Plan [Human + Tool 1]
  - Build your plan iteratively iteratively, going from a high level overview doc to detailed step by step plan. 
  - Heavy human interaction in earlier steps (lots of back and forth). Minimal or none by step 3/4/5. Using Cursor + Opus here.
  - Ask it to challenge you (prompt). Get these out of the way in step 1/2.
  - Steps
    - Step 1: `functional_spec.md` 
    - Step 2: `architecture.md`
    - Step 3: individual component designs (including test plans) `components/[comonent_name].md` + check
    - Step 4: order to implement project in describing N phases `implementation_plan.md` 
    - Step 4: per-phase implementation plans `phase_plan/phase_N.md`
- Build phase by phase.
  - Using Claude Code and CLM, literally just prompt “implement phase 7” and let it work. [tool 2]. `/clear` before each phase starts. Needed context is stored in our planning files.
  - General `phase_instructions.md` (link)
    - Agent can ask questions at start, but only at start. After that the agent should not exit or ask questions until it’s complete. That includes all tests passing, all linting/formating errors corrected, human review test plan ready (if needed)
    - For UI or other things that can’t have unit tests, write a `phase_N_manual_test_plan.md` to review with user
  - Optional: Agent Phase Code Review
    - Ask smarter model if `git diff` matches the spec and phase plan well, and look for issues. Optional depending on complexity of task.
  - Human Review When Done a phase: 
    - run manual testing plan if needed (back in Cursor+Opus here since it’s interactive)
    - Optional: Do a quick manual code review of major decisions
  - Check in phase
- Repeat until all phases are done
- Human testing and iteration [tool 1]
  - Use your new feature, give feedback and iterate. More “vibey” here, but it has a large spec to work off of so still very on rails. Don’t stop updating spec.
- End to End Code Agentic Code Review
  - Ask for end to end agentic code review with GLM 4.7 (prompt below)
  - Interactively review issue_summary and make fixes with your best model (Cursor+Opus)
- Code Review Agent Code Review
  - Don’t open Github PR until ready for Gemini/Code Rabbit to process whole thing.
  - Continuing human code review on everything. No non-reviewed agentic code yet.

