# Two Tools: Interactive vs Autonomous Agents

The whole process uses two tools — really two modes of working with AI. They're different in cost, attention level, and what they're good at.

**Key Insight**: About 90% of the important decisions happen during planning. About 90% of the token usage happens during coding. Use two different tools.

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

My setup: Claude Code using GLM 4.7 on the z.ai Coder plan, web-search MCP (Z.ai), [Hooks MCP](https://github.com/scosman/hooks_mcp) setup with project-specific build, test, lint, format, coverage, and UI test commands. 

Note: yes it's ironic I'm using a Claude model in Cursor and non-Claude model in Claude Code. But it works and I like it.