# Conclusion

I'm not writing code anymore, so you'd be tempted to call it vibe coding. But I'm going deeper into architecture than I normally would, writing more tests, running more manual tests, and producing better code than when I was constrained by my typing speed and focus. The results are genuinely better — not just faster, but higher quality.

288 unit tests. A full UI test suite. Manual test plans for every system integration. Thorough code review from multiple angles. I would never have done all of that on my own. Not because I don't know how — because I'm lazy and there are only so many hours. With this process, the marginal cost of doing it right dropped to near zero.

Vibe crafting is a silly name, but it gets the idea across. I'm spending more time steering and reviewing results than coding. But the process underneath is deliberate, structured, and uncompromising on quality.

## All Prompts and Artifacts

Every prompt referenced in this post is a real file from various projects I built. Here's the full set:

**Planning (Interactive Agent):**
- [spec_guide.md](../prompts/spec_guide.md) — Prompt for building functional specs, architecture, and component designs
- [ui_spec_guide.md](../prompts/ui_spec_guide.md) — Prompt for UI-specific component specs
- [implementation_plan_prompt.md](../prompts/implementation_plan_prompt.md) — Prompt for creating the phased implementation plan
- [build_phase_plans.md](../prompts/build_phase_plans.md) — Prompt for writing detailed per-phase plans

**Building (Autonomous Agent):**
- [phase_instructions.md](../prompts/phase_instructions.md) — The autonomous loop: full instructions the coding agent follows for every phase

**Testing & Review:**
- [manual_test_plan_guide.md](../prompts/manual_test_plan_guide.md) — Prompt for generating manual test plans and diagnostic UI
- [agentic_cr.md](../prompts/agentic_cr.md) — Prompt for end-to-end agentic code review

**Example Artifacts:**
- [example phase plan](../example_specs/phase13.md) — A real per-phase plan from the pipeline project, showing the level of detail the coding agent works from

Note: this blog post was vibe-crafted! See the [specs](/specs/) folder for how it was built.