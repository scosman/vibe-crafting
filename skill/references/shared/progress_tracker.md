# Progress Tracker

The manager must output a progress block at key moments throughout the flow. This is your program counter — it keeps you on track and commits you to the process.

## When to Output

- **Before spawning the first sub-agent.** This is your commitment to the process. If you haven't output a progress block, you haven't started the process. Output it immediately after completing pre-work (clarification, pre-checks, fetching data) and before any sub-agent spawn.
- **After every sub-agent return.** Before doing anything else, output the updated block.
- **After any loop-back** (CR feedback, commit hook failure). Update the block to reflect the reset and incremented round counter.

## Format

Wrap the block in `<progress>` tags. Use the label provided by your command file ("Phase [N] Progress", "Task Progress", or "PR Feedback Progress").

## Step 0: Pre-Work

Every command has setup work before the implementation loop begins. Track this as Step 0 in your progress block. The specific Step 0 items vary by command — your command file defines them. Common patterns:

- **Task mode:** Step 0a: Clarify, Step 0b: Create task file
- **Implement mode:** Step 0: Pre-checks
- **PR mode:** Step 0a: Find PR, Step 0b: Fetch comments

Step 0 items may complete before the first progress block is output (e.g., clarification requires user interaction). That's fine — show them as already checked off in your first block. The point is that your first progress block includes them, giving you the full picture of where you are.

Example (task mode, first progress block after clarification and task file creation):

```
<progress>
Task Progress:
- [x] Step 0a: Clarify — complete
- [x] Step 0b: Task file — created
- [ ] Step 1: Coding — in progress
- [ ] Step 1b: Attestation — pending
- [ ] Step 2: Code review — pending
- [ ] Step 3: Commit — pending
- [ ] Step 4: Verify — pending
- [ ] Step 5: Summary — pending
</progress>
```

Example (implement mode, mid-flow):

```
<progress>
Phase 2 Progress:
- [x] Step 0: Pre-checks — complete
- [x] Step 1: Coding — complete
- [x] Step 1b: Attestation — complete
- [ ] Step 2: Code review — in progress
- [ ] Step 3: Commit — pending
- [ ] Step 4: Verify — pending
- [ ] Step 5: Summary — pending
</progress>
```

## Round Counters

When the flow loops back (CR feedback or commit hook failure), increment a round counter on the affected steps. Only show the counter starting at round 2 — no noise on the happy path.

After CR finds issues and coding agent is resumed:

```
<progress>
- [x] Step 1: Coding — complete (round 2)
- [ ] Step 1b: Attestation — in progress (round 2)
- [ ] Step 2: Code review — pending (round 2)
- [ ] Step 3: Commit — pending
- [ ] Step 4: Verify — pending
- [ ] Step 5: Summary — pending
</progress>
```

After a commit hook failure resets back to Step 1b:

```
<progress>
- [x] Step 1: Coding — complete (round 3, hook fix)
- [ ] Step 1b: Attestation — in progress (round 3)
- [ ] Step 2: Code review — pending (round 3)
- [ ] Step 3: Commit — pending (round 2)
- [ ] Step 4: Verify — pending
- [ ] Step 5: Summary — pending
</progress>
```

Note: Steps 3–5 have their own counter independent of the 1→1b→2 loop, since commit can fail separately.

## Rules

- **Output your first progress block BEFORE spawning the first sub-agent.** This is non-negotiable. No progress block = you haven't entered the process.
- Output the progress block **after every sub-agent return**, before doing anything else.
- Mark each step as it completes. Show the current step as "in progress."
- **After outputting the progress block, immediately proceed to the next pending step.** Do not wait for user input. Do not ask what to do next. The progress block tells you what to do next — do it.
- The ONLY reasons to stop and wait for the user: (1) escalation/roadblock from the coding agent, (2) after the final step (the flow is complete).
- If a step sends you back to an earlier step (e.g., commit hook failure → back to Step 1b), update the block to reflect the reset, increment the round counter, and keep going.
