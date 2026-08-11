## Build Phase by Phase

For each phase, the loop is: build autonomously, review, iterate, commit, then hands-on test anything the tests can't reach. Here's what each step looks like.

### Build [Autonomous Agent]
`[░░░░░░░░░░] Human` · `[██████████] Agent`

The prompt is almost comically simple: "implement phase N." That's it. The agent reads the phase plan, knows what to build, and gets to work. You walk away.

Before each phase, clear the agent's context (`/clear` in Claude Code, or start a new session). Everything the agent needs is in the planning files — the phase plan is self-contained by design. A clean context means the agent isn't confused by stale conversation history from three phases ago.

The real magic isn't the prompt — it's the standing instructions the agent follows every time. I have a [`CLAUDE.md`](../prompts/CLAUDE.md) that points to [`phase_instructions.md`](../prompts/phase_instructions.md), which defines the full autonomous loop: read the plan, ask clarifying questions (only at the start), build, lint and format until clean, self code review, write tests, run tests until passing, mark phase complete, stop.

That loop is what makes "low attention" possible. You're not trusting the agent on every individual step — you're trusting the loop. The agent must not exit until all quality gates pass. If tests fail, it iterates. You come back to a clean result, or a stuck agent you can help.

If it ever gets really stuck, you probably have a spec problem. Throw it out, improve your spec, and start again.

### Review [Interactive Agent]
`[█████░░░░░] Human` · `[██░░░░░░░░] Agent`

Every phase gets a quick review before you move on. Check the code it generated, check the list of tests, make sure it looks good. Not line-by-line — that comes later in the end-to-end review.

For substantial phases, I'll feed the `git diff` and phase plan to the Interactive Agent and ask it to check the implementation against the spec. You're using the smarter model to code-review the cheaper model. It's also good for catching structural mismatches.

### Commit

One commit per phase. Keeps the history clean and makes it easy to bisect if something breaks later.

Code review is the gate here, not your eyeballs on the UI. That comes next.

### Agent-Led Manual Testing (UI Only) [Interactive Agent]
`[█████░░░░░] Human` · `[█████░░░░░] Agent`

Not everything can be unit tested. UI components, system API integrations (notifications, alarms, live activities) — these need human help. When unit tests aren't sufficient, the agent writes a manual test plan as part of the phase ([manual_test_plan_guide.md](../prompts/manual_test_plan_guide.md)). It creates a diagnostic view in the app and writes step-by-step instructions: "tap this button, verify this appears, check that the notification fires." 

This happens *after* the commit, which is deliberate. The agent is increasingly not running on my machine — containers, remote sandboxes, worktrees. Asking me to go look at the UI before it commits is asking for something I often can't do: there's nothing to sync yet. Once it's committed, I pull and look. If something's off, that's a follow-up commit, which is fine — it goes through the same review loop as everything else.

See the section below for more details on how this works.
