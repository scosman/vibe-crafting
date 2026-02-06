## Code Review
`[█████░░░░░] Human` · `[█████░░░░░] Agent`

Once all phases are built, I run a full codebase review — not a per-phase diff, but a comprehensive audit of the entire codebase against the spec.

### End-to-End Agentic Review [Autonomous Agent]

The prompt ([agentic_cr.md](../prompts/agentic_cr.md)) tells the agent to read the full spec, create a review plan, go through it systematically producing several smaller CR reports, and produce an `issues_summary.md` labeled by severity. The instruction is explicit: "be critical like a senior architect." 

This runs autonomously — kick it off and come back to the summary. Then switch to the Interactive Agent: read through the findings, decide what's real, fix what matters. Some findings are noise, but some are genuinely good catches — architectural inconsistencies, missing error handling, tests that don't actually test what they claim to. 

### External Agent Code Review

I also run external reviews: Gemini code review and CodeRabbit on pull requests for an additional look.  Don't open the PR until you're actually ready — running review tools on half-finished code just drowns you in noise. In general you should stump these, and get only nits back if your last review was any good.

### Human Code Review

We still do a full human code review pass before shipping. This isn't vibe-coding, we want to know what we've build and stand by it.
