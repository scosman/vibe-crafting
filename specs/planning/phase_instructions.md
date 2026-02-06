Each phase should follow this pattern:

- Read `specs/planning/implementation_plan.md`
- Choose the next unchecked phase based on the checkmarks. Confirm with the user it's the right one to work on (if they didn't already tell you which phase to do).
- Read `specs/planning/outline.md` and the relevant section(s) of `specs/project_overview/` for context on what this phase should cover.
- Read all existing section files in `specs/sections/`. The blog is short enough to fit in context, and you need to match tone, style, and avoid repeating what's already been said.
- Check `specs/planning/needed_prompts.md` for any prompt files this phase needs to reference. If they don't exist yet in `prompts/`, ask the user to provide them before drafting. You'll need these to write the section well — don't use placeholders for content you should have.
- Draft the section file(s) in `specs/sections/` as listed in the implementation plan.
  - Follow the outline's bullet points for structure, but write real prose — not bullet lists.
  - Tone: conversational, honest, technical but accessible. First person. Not preachy or salesy.
  - Link to prompt files in `prompts/` where the outline calls for it (use placeholder links if prompt file doesn't exist yet).
  - Keep sections self-contained — they should read well standalone and as part of the whole.
- Self-review the draft:
  - Does it cover everything the outline calls for?
  - Does it flow well from the previous section and into the next?
  - Spelling and grammar check.
  - Are examples concrete, not generic?
- Present the draft to the user for review. Do not continue to the next phase until the user approves.
- Once approved, mark the phase's tasks as complete in `specs/planning/implementation_plan.md`. Only edit this file to toggle checkboxes, nothing else.
- Stop after a single phase is done. Don't continue until asked.
