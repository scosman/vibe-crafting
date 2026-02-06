## Build Spec & Implementation Plan [Interactive Agent]

This is where most of the real thinking happens. You're going to iteratively refine from a broad project overview into a detailed, step-by-step build plan. Five steps, each one more specific than the last.

The human interaction curve is steep at the start and tapers off. Steps 1 and 2 involve a lot of back-and-forth — you're making architecture calls, pushing back on suggestions, asking "what about X?". By steps 4 and 5, the agent mostly has what it needs and you're reviewing output rather than co-authoring it.

I'd call this "vibe-speccing." You're not reading every line of the spec like a legal contract. You're asking questions, forcing the agent to think end-to-end, and checking that the big decisions feel right. The specs are a collaboration — the AI drafts, you steer.

### The Five Steps

**Step 1: Functional Spec** (`functional_spec.md`)
`[█████░░░░░] Human` · `[░░░░░█████] Agent`
Start from your project overview and build out a full description of what the app does. Features, user flows, edge cases. The agent drafts, you iterate until it captures what you actually want. ([spec_guide.md](../prompts/spec_guide.md))

**Step 2: Architecture** (`architecture.md`)
`[█████░░░░░] Human` · `[░░░░░█████] Agent`
High-level technical design — data model, key components, how they interact. This is where you make the big calls: what frameworks, what patterns, what tradeoffs. ([spec_guide.md](../prompts/spec_guide.md))

**Step 3: Component Designs** (`components/[name].md`)
`[██░░░░░░░░] Human` · `[████████░░] Agent`
Individual specs for each component, including public interfaces, design patterns, API usage, and test plans. For UI components, I used a separate pass with its own prompt. ([spec_guide.md](../prompts/spec_guide.md), [ui_spec_guide.md](../prompts/ui_spec_guide.md))

**Step 4: Implementation Plan** (`implementation_plan.md`)
`[█░░░░░░░░░] Human` · `[█████████░] Agent`
Order the work into phases. What gets built first, what depends on what. Each phase should be roughly one component — small enough to review in one sitting. ([implementation_plan_prompt.md](../prompts/implementation_plan_prompt.md))

**Step 5: Per-Phase Plans** (`phase_plan/phase_N.md`)
`[█░░░░░░░░░] Human` · `[█████████░] Agent`
Detailed plans for each phase, pulling relevant details from the spec into a self-contained reference. This is what the coding agent actually reads when it starts a phase — it shouldn't need to go back to the full spec. ([build_phase_plans.md](../prompts/build_phase_plans.md), [example plan](../example_specs/phase13.md))

### Challenge the AI Early

One thing I do in steps 1 and 2: explicitly ask the agent to challenge my assumptions. Push back on my ideas. Tell me what's wrong with my approach. Suggest alternatives. Keep pushing for feedback on the top issues, until it's feedback feels like nits you can skip.

This matters because AI agents are sycophantic by default — they'll agree with whatever you say and build exactly what you asked for, even if it's a bad idea. If you ask for pushback early, you surface problems before they're baked into the spec. Much cheaper to fix a bad assumption in step 1 than to rewrite three components in step 4.
