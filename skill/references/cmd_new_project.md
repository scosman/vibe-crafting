# `/spec new_project` — Create a New Project

The primary planning flow. Creates a new project spec under `/specs/projects/PROJECT_NAME/` and walks through all planning steps.

## Pre-Check

Before starting, check if there's an active project in `.specs_skill_state/current_project.md`.

If an active project exists and has incomplete work:

> You have an active project: [project_name]
>
> - [specs still in draft]
> - [implementation phases remaining]
>
> Use `/spec continue` to resume work on that project, or confirm you want to start a new project.

Wait for user confirmation before proceeding.

## Starting a New Project

### Set Active Project

After confirming the project name and creating its folder, **immediately** set it as the active project:

```
Current Project: /specs/projects/PROJECT_NAME
```

Write this to `.specs_skill_state/current_project.md` right away. Don't wait until the planning flow is complete.

### Project Folder

Create the project folder:

```bash
mkdir -p "specs/projects/PROJECT_NAME"
```

Use the project name provided by the user. Minor cleanup for filesystem safety (remove leading/trailing spaces, replace slashes with hyphens), otherwise keep it close to what they said.

## Question-Asking Format

When you need multiple answers from the user, group questions by topic and number them sequentially:

```
### Feature Clarification
1. Should the system support [X] or [Y]?
2. What happens when [edge case]?

### Technical Constraints
3. Do you have a preference for [technology choice]?
4. Any performance requirements I should know about?

Answer each question on a line, preceded by its number: `1. answer\n2. answer...`
Add as much detail as needed.
```

**Guidelines:**
- Group related questions under descriptive headers
- Number sequentially across all groups
- Offer concrete options where possible ("A, B, or C?" not "what do you want?")
- Keep questions self-contained enough to answer without re-reading context
- Don't ask questions whose answers are already in the provided context

## Planning Persona

For all planning steps, adopt this persona:

> You are a senior engineering lead/architect at a top tech company (FAANG-level). You care about long-term maintainability, code quality, and building the right thing.
>
> You've been around long enough to know the difference between a trend and a best practice. You push back on decisions that will cause problems down the line.
>
> You also think like a product manager: are we building the right thing for users? Will this actually solve their problem?
>
> For UI projects, you additionally think like a senior designer: intuitive, discoverable, low cognitive load, correct progressive disclosure.

This persona applies across all planning steps.

## Research (Conditional, Any Planning Step)

Planning is only as good as what you know. When the project depends on something external that neither you nor the user can describe accurately from memory — a standard, a third-party API, a library's real capabilities, how others have solved this — run [research](cmd_research.md) before you spec against a guess.

→ Read [references/cmd_research.md](cmd_research.md) and follow it.

**When to run it:**

| Before | When the unknown is |
|--------|---------------------|
| Step 1 (project overview) | What's even possible or worth building — the user is exploring ("should we support the OpenEnv standard?") |
| Step 2 (functional spec) | What the feature set must include — required behaviors defined by an external spec, protocol, or competitor baseline |
| Step 4 (architecture) | How to build it — library and framework choices, API contracts, protocol details, performance characteristics |

Don't research by reflex. Research when a wrong assumption would send the spec down the wrong path, not to pad the docs.

**How it runs here — split, in this context:**

- **Steps 0–2 of the research command run inline**, in this conversation: the web-access check, the subtopic plan, and **user approval of that plan**. The user sees the plan and approves both scope and cost — web tools aren't free.
- **Step 3 onward dispatches from here too**: this context spawns the subtopic research sub-agents and the summary sub-agent directly.
- Do **not** wrap research in a single "research manager" sub-agent — that hides the plan approval behind a handoff. One layer of sub-agents.
- Context stays small: research is written to files, and sub-agents return only short summaries.
- **Emit the research command's "Research Progress" block while it runs.** `new_project` isn't otherwise progress-tracked, so the block starts when research starts and ends when it does — then you're back in the planning step it was blocking.

**Working folder:** `specs/projects/PROJECT_NAME/research/[topic]/` — this is the `[working_folder]` the research command expects, inside the project rather than the standalone `specs/research/` location.

When research completes, read `specs/projects/PROJECT_NAME/research/[topic]/summary.md` and continue the planning step it was blocking. Cite the research in the artifact you write — link to the summary where a spec decision rests on a finding.

## Step 1: Project Overview

If the user is still deciding what's worth building and that turns on something external — is this standard real, does that platform allow it, has someone already solved it — run [Research](#research-conditional-any-planning-step) first, and write the overview against what it found.

Ask the user to describe what they want to build:

> Tell me about what you want to build. What does it do? Who is it for? Why are you building it? Any technical requirements or constraints I should know about?

Create `specs/projects/PROJECT_NAME/project_overview.md`:

```markdown
---
status: draft
---

# [Project Name]

[User's description, kept very close to what they wrote. Only minor spelling/grammar fixes. This is their document.]
```

Present it to the user for review. Ask:

> Here's the project overview based on what you described. Does this look right?

If they approve, mark `status: complete`. If they want changes, make them and ask again.

## Step 2: Functional Spec

If the feature set depends on an external spec, protocol, or product baseline you can't describe accurately, run [Research](#research-conditional-any-planning-step) first.

→ Read [references/step_functional_spec.md](references/step_functional_spec.md) and follow it.

## Step 3: UI Design (Conditional)

Only run if the project has a user-facing interface. Detect this from the overview + functional spec content.

If there's no UI (backend API, library, CLI tool, etc.), ask:

> This doesn't appear to have a user-facing interface. Should we skip the UI design step and proceed directly to architecture?

If they confirm, skip to Step 4.

If UI is needed:

→ Read [references/step_ui_design.md](references/step_ui_design.md) and follow it.

## Step 4: Architecture

If technical choices hinge on unknowns — which library, what an API actually supports, how a protocol behaves — run [Research](#research-conditional-any-planning-step) first. The architecture is supposed to leave no significant technical decisions to the coding agent; researching after you've written it is too late.

→ Read [references/step_architecture.md](references/step_architecture.md) and follow it.

## Step 5: Component Designs (Conditional)

During the architecture step, you'll decide whether component designs are needed:

- **Small projects**: Everything fits in `architecture.md`. Skip this step.
- **Larger projects**: Individual components need their own detailed docs.

If component designs are needed:

→ Read [references/step_component_designs.md](references/step_component_designs.md) and follow it.

If not needed, proceed directly to Step 6.

## Step 6: Implementation Plan

Read all completed spec artifacts (project_overview.md, functional_spec.md, ui_design.md if present, architecture.md, components if present).

Design a phased build order:

- Logical by dependencies — foundational things first
- Each phase is roughly one coherent unit of work
- Small enough to review in one sitting, but not so small that the user is burdened with many tiny CRs
- For small projects: single phase is fine

Write `specs/projects/PROJECT_NAME/implementation_plan.md`:

```markdown
---
status: draft
---

# Implementation Plan: [Project Name]

## Phases

- [ ] Phase 1: [Brief description]
- [ ] Phase 2: [Brief description]
- [ ] ...
```

Keep this file short — it's an ordered checklist referencing the specs for details. Don't restate the spec content.

Present to the user for review. If approved, mark `status: complete`.

## End of Flow

The `new_project` flow ends here. All spec artifacts are now complete.

Phase plans (`/phase_plans/phase_N.md`) are written by the coding agent at the start of each implementation phase, not during planning.

Next steps for the user:

> Your project spec is complete. Use `/spec implement` to start building, or `/spec implement phase 1` for the first phase.
