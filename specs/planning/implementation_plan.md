# Implementation Plan — Vibe Crafting Blog Post

## Approach

Each section of the README is drafted as its own file in `specs/sections/`. This keeps diffs clean, lets us iterate on sections independently, and avoids merge conflicts with ourselves. When all sections are polished, we cat them together to produce the final `README.md` at the repo root.

```
specs/sections/
  introduction.md
  two_tools.md
  process_overview.md
  process_project_overview.md
  process_spec_and_plan.md
  process_build_phases.md
  process_review.md
  process_testing_iteration.md
  process_code_review.md
  process_external_review.md
  lessons_learned.md
  prior_art.md
  conclusion.md
```

Assembly: `cat specs/sections/*.md > README.md`

## Phases

### Phase 1: Foundation — Introduction & Two Tools
- [x] Write `introduction.md`
  - Title, vibe crafting vs vibe coding, thesis, repo description, what was built
  - Compact TOC (will finalize links in assembly phase)
- [x] Write `two_tools.md`
  - Interactive Agent / Autonomous Agent framework
  - Cost and attention model (90/90 split)
  - Specific toolset as examples, not definitions
- [x] **Human review:** Does the intro hook? Is the two-tool concept clear to someone unfamiliar?

### Phase 2: The Process — Overview & Planning Steps
- [x] Write `process_overview.md`
  - High-level diagram or summary of the full flow
  - Key idea: broad → specific before any code
- [x] Write `process_project_overview.md`
  - Human writes `project_overview.md`, what goes in it
- [x] Write `process_spec_and_plan.md`
  - Iterative spec building (5 steps), human interaction curve
  - "Vibe-speccing" concept
  - Link to challenge prompt, spec prompts
- [x] **Human review:** Does the planning flow read as a clear sequence? Are the spec steps understandable without seeing the actual artifacts?

### Phase 3: The Process — Build, Review, Iterate
- [x] Write `process_build_phases.md`
  - "Implement phase N" pattern, `/clear`, phase instructions
  - Manual test plans for UI
  - Link to phase_instructions.md, implement prompt
- [x] Write `process_review.md`
  - Agent code review, human manual testing, human code review, git commit
- [x] Write `process_testing_iteration.md`
  - Human testing loop, keeping spec updated
- [x] Write `process_code_review.md`
  - End-to-end agentic review (Autonomous → Interactive)
  - Link to review prompt
- [x] Write `process_external_review.md`
  - External tools (Gemini/CodeRabbit), no unreviewed code ships
- [x] **Human review:** Does the build→review→iterate loop feel natural? Is it clear when to use which agent?

### Phase 4: Lessons Learned
- [ ] Write `lessons_learned.md`
  - 4.1 It Still Needs You — technical, UX, and product design examples
  - 4.2 Sycophancy is Real — examples and mitigation
  - 4.3 AI-Driven Manual Testing — concept, bugs found, links to prompts/test plans
  - 4.4 More Polish Than You'd Bother With — "free" improvements
  - 4.5 It's not great at speccing — competing sources of truth, AI assumptions
  - 4.6 Example Projects and Costs — two projects (iOS app, pipeline) with token/cost breakdowns
  - 4.7 True Autonomous Development With Security — sandboxing challenges, toolchain differences, Xcode pain
  - Concrete examples for each lesson
  - Links to relevant prompts and artifacts where applicable
- [ ] **Human review:** Are the anecdotes compelling? Anything missing from real experience? Tone check — honest, not preachy.

### Phase 5: Prior Art
- [ ] Write `prior_art.md`
  - Other spec-driven development tools and approaches
  - TODO: research and fill in details
- [ ] **Human review:** Coverage and fairness of comparisons.

### Phase 6: Conclusion & Prompts
- [ ] Write `conclusion.md`
  - Summary, audience, links to all prompts/artifacts
- [ ] Collect and add all prompt files to `prompts/` directory (per `needed_prompts.md`)
- [ ] **Human review:** Do prompt files make sense standalone? Are they all linked from the right sections?

### Phase 7: Examples & Links
 - [ ] Ask users for examples of key outputs of the process (specs, arhcitecture, etc)
 - [ ] link to the examples throughout our doc
 - [ ] setup appriopiate links between related sections (process -> lessons learned). Anchor links won't funtionally work yet but that's okay.

### Phase 8: Polish
- [ ] Review all complete sections from quality
- [ ] Fix any spelling issues carried from outline (see review notes)

### Phase 7: Assembly & Polish
- [ ] Cat all section files into `README.md`
- [ ] Fix TOC content and links to match actual headings/sections/anchors
- [ ] Final read-through for flow, tone, consistency
- [ ] **Human review:** Full end-to-end read of the assembled README. Final sign-off.

## Notes

- All phases use the **Interactive Agent** — this is a writing project, not a coding one. Every section benefits from human collaboration and iteration.
- Prompt files (Phase 5) may require pulling from external sources (chat history, other projects). Flag anything we can't find.
- Keep sections self-contained enough that reordering or cutting a section doesn't break the rest.
- Each process section (3.0 through 3.7) should have an effort bar right under the header showing human vs agent split: `[██████████] Human` · `[░░░░░░░░░░] Agent` (adjust fill to reflect the ratio).
