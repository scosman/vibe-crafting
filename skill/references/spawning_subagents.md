# Spawning Sub-Agents

Explanation of the sub-agent pattern and how to use it across different tools.

## What and Why

Sub-agents are fresh agent contexts with no conversation history from the current session.

Use them when clean context matters:

- **Code review**: CR shouldn't see coding agent's thinking
- **Phase implementation**: Each phase starts fresh

The sub-agent sees only what you pass it (a prompt) plus the repo. No conversation history.

## The Return Path Is Not Optional

**Dispatch every sub-agent in a way that delivers its final message back into your
context.**

The manager's whole control flow depends on reading what the sub-agent returns — the
`<attestation>` block, the `<ui_review>` block, the CR findings. If the return payload
does not reach you, you cannot validate, cannot route feedback, and cannot decide whether
to commit.

Many agent tools offer more than one dispatch mode, and they are not equivalent:

| Dispatch mode | You get back |
|---|---|
| **Blocking / result-returning** (use this) | The sub-agent's full final text, in your context, when the call returns |
| Fire-and-forget, background, or "teammate" modes | A spawn receipt now. The payload arrives later, only if the agent explicitly sends one — and often only once you end your turn |

Before you dispatch, know which mode you are using. Two failure signatures mean you picked
the wrong one:

- The call returns instantly with an ID or a "started" message instead of the agent's work.
- You later receive a bare "finished" / "idle" / "available" signal with no content
  attached. **A completion notification is not a result.** The agent's own final text is
  not carried by it.

**Do not poll or sleep inside your turn waiting for a message-based payload.** On some
tools the message is only injected into your context at a turn boundary, so waiting in-turn
never delivers it no matter how long you wait. This is the single most confusing failure
mode: the agent finished, the send succeeded, and you still see nothing. A blocking
dispatch has no such problem, which is why it is the default here.

If you are stuck this way, do not proceed as if the step passed, and do not read an idle
notification as "still working." Go to
[Recovery](#recovery-when-a-payload-is-genuinely-lost).

Also record whatever handle the tool gives you at dispatch time (an agent ID, session ID,
or name). You need it to resume this agent later, and it is usually only offered once.

## Model: Critical

Spawn subagents with the same model you use. Don't use lesser models for speed or perf. We want the user's model selection for entire process.

## What to Pass

- A prompt/task description (caller specifies this — typically a reference file's content)
- Optionally structured data like `<prior_cr_feedback>`

**Never pass conversation history.** The point is a clean context.

## Examples by Tool

### Claude Code

Use the `Agent()` tool, and **do not pass `name`**:

```
Agent({
  description: "Code review phase 1",
  prompt: "Review this code using the guidelines in references/cr_agent_prompt.md"
})
```

`name` is the trap here. It makes the agent addressable, and when agent teams are enabled
(`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) a named agent launches as a **teammate**, which
communicates only through a mailbox. A teammate's plain final text is never returned to
you — per the `SendMessage` tool's own description, "Your plain text output is NOT visible
to other agents." What you get instead is:

```
{"type":"idle_notification","from":"coder-p1","idleReason":"available"}
```

which carries no payload. This is documented behavior, not a bug. `run_in_background: false`
does not save you — background is the default in recent versions, and a named agent is a
teammate regardless of the flag.

A teammate *can* report back, by calling `SendMessage({to: "team-lead", ...})`. Measured
behavior (2026-08-23, Claude Code 2.1.220, in-process mode): the send succeeds, the message
is written to the lead's inbox within a second, and it is then **injected into the lead's
context only when the lead's turn ends**. Polling or sleeping in-turn never surfaces it.
The lead is woken automatically once it yields.

That makes teams workable only for an event-driven manager that deliberately ends its turn
after spawning. This skill's flows are written to run straight through without stopping, so
they need the blocking path. You also cannot see whether teams are enabled on the user's
machine. Treat it as unconditional: **if you need the answer in the same turn, do not name
the agent.**

The result of an unnamed call contains the agent's full final text **and** its `agentId`.
Save the `agentId`.

### Cursor

Use Cursor's sub-agent spawning capability, in the mode that returns the sub-agent's result
to the parent.

### Generic / Unknown

If the tool doesn't have explicit sub-agent support:

Approximate by:
1. Clearing context
2. Starting a new conversation with only the sub-agent prompt

This is less ideal since it can't run in parallel with the parent, but works for the clean context requirement.

## Resuming Sub-Agents

Some workflows need to send follow-up messages to an existing sub-agent
rather than spawning a fresh one. This preserves the agent's context.

Use when: the sub-agent needs to continue work it already started
(e.g., a coding agent addressing CR feedback on code it just wrote).

This is where the handle you saved at dispatch time gets used. Whatever the mechanism, the
same rule applies: the resumed agent's reply must come back into your context.

### Claude Code

Use `SendMessage`, addressed to the `agentId` you saved at spawn time:

```
SendMessage({
  to: agentId,
  summary: "CR feedback to address",
  message: "[the CR Feedback Prompt template, filled in]"
})
```

The Agent tool has **no `resume` parameter**. `SendMessage` is the way to continue a
sub-agent. A completed sub-agent that receives a `SendMessage` auto-resumes, and its reply
routes back to the manager.

### Cursor

Use Cursor's `resume` options

### Generic / Unknown

Generic tools typically don't support resuming — use a fresh spawn with accumulated context if needed.

### When to Resume vs. Fresh Spawn

- **Resume**: coding agent receiving CR feedback or commit approval (needs its prior context)
- **Fresh spawn**: CR agent (must NOT have coding context)

## Recovery: When a Payload Is Genuinely Lost

If a sub-agent finished but its output never reached you — wrong dispatch mode, an errored
call, or an interrupted session:

0. **First, end your turn once and see if the payload arrives.** If the dispatch was
   message-based, this alone often resolves it — the message may be queued and waiting for
   a turn boundary. Cheapest possible fix; try it before anything below.
1. **Do NOT guess from file timestamps, `git log`, or mailbox files.** Probing mtimes to
   infer what an agent did is unreliable and will mislead you into re-spawning duplicate
   work or declaring a live agent dead. An empty inbox file is especially treacherous: it
   means "already consumed" just as often as "never sent," and you cannot tell which.
2. **Try to resume it and ask for the summary again.** If you saved a handle, resume the
   agent and ask it to re-send its summary. This is the cheap fix and preserves its
   context.
3. **Only if that fails, dispatch a fresh sub-agent with accumulated context** — using a
   result-returning mode this time. Give it:
   - The original phase/task instructions
   - `A prior coding agent already wrote this code. Review the current diff to understand what was done.`
   - The CR feedback or the next instruction it needs to act on

Step 3 is the fallback, not the default. It costs a full context rebuild and the new agent
cannot attest to work it did not do — expect to re-run checks.

If you cannot recover the payload at all, say so to the user and ask them to paste the
sub-agent's summary. Do not fabricate an attestation or proceed as if the step passed.

## Fallback Language

If unsure about tool capabilities, use:

> Use your sub-agent or task-spawning capability to start a fresh agent context with the following prompt. Dispatch it so that its final message is returned to you:
>
> [prompt content]

The agent will use whatever mechanism is available.
