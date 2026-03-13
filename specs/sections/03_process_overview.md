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

The key idea: you go broad to specific before any code is written. By the time the coding agent starts, it has a functional spec, architecture doc, component designs with test plans, a phased implementation plan, and [detailed per-phase plans](../example_specs/phase13.md). It's not winging it — it's executing a well-defined plan.

This front-loaded planning is where most of the important decisions happen. It's also where the human does most of their work. Once you're in the build loop, you're mostly reviewing output and giving feedback.
