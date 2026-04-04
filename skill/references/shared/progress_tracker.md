# Progress Tracker

The manager must output a progress block after every sub-agent return. This is your program counter — it keeps you on track.

## Format

Wrap the block in `<progress>` tags. Use the label provided by your command file ("Phase [N] Progress" or "Task Progress").

Example (mid-flow, first pass, project mode):

```
<progress>
Phase 2 Progress:
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

- Output the progress block **after every sub-agent return**, before doing anything else.
- Mark each step as it completes. Show the current step as "in progress."
- **After outputting the progress block, immediately proceed to the next pending step.** Do not wait for user input. Do not ask what to do next. The progress block tells you what to do next — do it.
- The ONLY reasons to stop and wait for the user: (1) escalation/roadblock from the coding agent, (2) after Step 5 (the flow is complete).
- If a step sends you back to an earlier step (e.g., commit hook failure → back to Step 1b), update the block to reflect the reset, increment the round counter, and keep going.
