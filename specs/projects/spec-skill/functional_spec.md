---
status: complete
---

# Functional Spec: Spec Skill

A skill (following the [Agent Skills](https://agentskills.io/specification) format) that implements a structured, spec-driven development process. The skill works on any codebase, in any language. It guides users from a blank-page project idea through functional design, architecture, phased implementation, and code review — with the human focused on decisions and review while AI agents handle drafting and building.

## Core Concept

Every batch of work ("project") lives under `/specs/projects/PROJECT_NAME/`. A project is a unit of change to the repo — typically one PR's worth of work. The skill walks the user through creating spec artifacts for that project, then uses those artifacts to drive autonomous implementation and structured code review.

The skill itself is project-agnostic. It provides the process; project-specific conventions (test commands, linting, formatting, style guides, CR standards) come from the user's system prompt configuration (CLAUDE.md, AGENTS.md, etc.).

## Commands

All commands are routed through `/spec`. The user types `/spec <subcommand>` and the skill routes accordingly. The skill description advertises available subcommands so agents can route without asking.

Available subcommands: `new_project`, `continue`, `implement`, `implement phase N`, `implement all`, `cr` / `code_review`, `setup`.

A bare `/spec` with no subcommand acts as a router: it reads the current state (active project, artifact statuses) and presents the most relevant options. It never requires the user to go through the router — direct subcommands always work. It can also handle open-ended requests, not just the listed commands.

### `/spec setup`

One-time (or incremental) setup for using the skill in a repo.

**Steps:**

1. Add `.specs_skill_state/` to `.gitignore` (creates file if needed, appends if not already present).
2. Create `/specs/projects/` directory if it doesn't exist.
3. **Monorepo detection:**
   - Search for root markers in subdirectories: `pyproject.toml`, `package.json`, `go.mod`, `Cargo.toml`, `build.gradle`, `pom.xml`, etc.
   - Ask the user: "Is this a monorepo? I found these potential sub-projects: [list]. Are there others?". Still ask if you found none.
   - If monorepo: create `/specs/projects/` in each confirmed sub-project root. Create `/specs/monorepo.md` at repo root describing the sub-projects and their paths. You may need to ask the user to get enough details about each project for monorepo.md to be a helpful file. We don't need detailed descriptions, but we need to know what each sub repo is.
4. **External knowledge check:** Scan the system prompt config for commonly needed references and suggest additions if missing:
   - Automated code checks / CI command
   - Test framework and run command
   - Linting / formatting commands
   - Code review guidelines
   - Code style conventions
   
   For each missing item, suggest where to add it and provide a template snippet.

**Idempotent:** Won't clobber existing setup. Only extends what's missing.

### `/spec new_project`

Create a new project from scratch. This is the primary planning flow.

**Pre-check:** If there's an active project in progress, confirm the user wants to start a new one (suggest `/spec continue` if they might mean to resume).

**After starting:** Set this as the active project in `.specs_skill_state/current_project.md`. Do not wait until complete to set as current project.

**Flow:**

#### Step 1: Project Overview
- Ask the user to describe what they want to build. Prompt for: what the thing does, who it's for, why they're building it, technical requirements, constraints.
- Create `/specs/projects/PROJECT_NAME/project_overview.md` from the user's description.
- Keep this very close to what the user wrote. Minor spelling/grammar edits only. This is their document.
- Mark `status: complete` in frontmatter once the user confirms it's done.

#### Step 2: Functional Spec
- Read the project overview.
- Ask clarifying questions in rounds to fill gaps. Start high-level, refine to details. 
- **Pushback opportunity:** Challenge feature decisions, unnecessary complexity, scope that doesn't match stated goals.
- Some examples, but the sections/document you define should best reflect the specific project:
  - **Features and behavior:** What does this do? What are the user flows or usage patterns?
  - **Edge cases and error handling:** What happens when things go wrong? What are the boundaries?
  - **Input/output contracts:** For APIs, CLIs, libraries — what goes in, what comes out, what are the formats?
  - **Configuration and defaults:** What's configurable? What are sensible defaults?
  - **Constraints:** Performance requirements, compatibility, security considerations.
  - **For projects with UI:** High level user-facing screens/views, navigation structure. Details will be in step 3. Spec here at functional level.
- Draft `functional_spec.md` and present for review.
- Iterate with user until confirmed complete.
- **Pushback opportunity:** After writing, review the overview and challenge anything that seems risky, underspecified, or likely to cause problems downstream. Suggest alternatives. The user gets final say but don't blindly follow.
- Mark `status: complete` when done and user has approved

#### Step 3: UI Design (conditional)
- Only runs if the project has a user-facing interface (detected from overview and functional spec).
- For UI projects: design the interface structure, screen/view inventory, navigation flow, component breakdown, interaction patterns.
- Follows the iterative Q&A approach: propose, get feedback, refine.
- Output depends on project type:
  - Web: page layout, page content, format (modal, sidebar, etc)
  - Mobile: screen flow, screen content, format (modal, sheet, etc) 
  - CLI: command structure, output formatting, interactive prompts
- Written to `ui_design.md`
- **Pushback opportunity:** Challenge UX decisions — discoverability, cognitive load, progressive disclosure, platform conventions.
- Mark `status: complete` when done and user has approved

#### Step 4: Architecture
- Ask technical questions to design the system. Cover these (and more as needed):
  - **Data model:** Entities, relationships, storage approach.
  - **Component breakdown:** Major classes/modules, their responsibilities, how they interact.
  - **Public interfaces:** Function signatures, API contracts, protocols/interfaces between components.
  - **Design patterns:** What patterns and why. Framework and library choices.
  - **Technical challenges:** Identify anything non-trivial and design the solution now. Don't leave hard problems for the coding phase.
  - **Error handling strategy:** How errors propagate, what's recoverable, logging approach.
  - **Testing strategy:** What kinds of tests, what coverage targets, what testing frameworks. Test plans per component.
- Decide if we're taking a 2-phase approach here (with component designs or not), based on the complexity of the project and if 1 file or overview+sub-files is best. Both expect the same level of technical detail, it's just a matter of if it's better in 1 file or many.
- Draft `architecture.md` and present for review.
- The architecture must be deep enough that no significant technical decisions remain for the coding agent. Classes, functions, overall flow, key algorithms, test cases, key deps — all here if we're doing a 1-phase approach with no components. That said, if doing a 2-stage design with components, some of these details may be covered in the next step in components; split the decisions and details across the architecture.md and component files as best makes sense for this project.
- **Pushback opportunity:** Challenge technical decisions, overengineering, missing error handling, inadequate test plans, framework choices that don't fit the use case, etc. Can also push back on a functional requirement that seems to be less important, but adds significant technical complexity and may not be worth the tradeoff (can offer alternatives spanning functional spec and architecture).
- Mark `status: complete` when done and user has approved

#### Step 5: Component Designs (conditional)
- The agent decides whether component designs are needed based on project size:
  - **Small projects:** Everything fits in `architecture.md`. Skip this step.
  - **Larger projects:** Create `/components/component_name.md` files for major components that need their own detailed design.
- Each component doc covers: purpose, public interface (full signatures), internal design approach, dependencies, test plan with specific test cases.
- Quick user review. "Component designs written to [folder]. Ready to continue to implementation plan?"
- Mark `status: complete` when done and user has approved

#### Step 6: Implementation Plan
- Read all completed spec artifacts.
- Design a phased build order:
  - Logical based on dependencies.
  - Each phase is roughly one component or one coherent unit of work — small enough to review in one sitting.
  - For small projects: "single phase" is fine.
  - Not too small. A user may have to stop what they are doing to review, so many tiny CRs are not effective.
- Write `implementation_plan.md` with markdown checklist style for progress tracking.
- Keep this file short — it's an ordered checklist, not a re-statement of specs.
- Mark `status: complete` when done and user has approved

**The `new_project` flow ends here.** Phase plans (`/phase_plans/phase_N.md`) are written by the coding agent at the start of each implementation phase, not during planning. See [Single Phase Implementation](#single-phase-implementation).

### `/spec continue`

Resume work on the active project. This is the "just do the next thing" command.

**Flow:**

1. Read current project from `.specs_skill_state/current_project.md`.
2. If no active project: list projects under `/specs/projects/`, ask which to work on.
3. Determine current state by checking artifact frontmatter statuses:
   - If in speccing phase (artifacts missing or `status: draft`): resume the spec flow at the next incomplete step. Tell the user: "Project X — continuing with [architecture / component designs / etc.]."
   - If all specs complete but phases remain unimplemented: treat as `/spec implement` — ask whether to implement next phase or all remaining.
4. Confirm with user before proceeding.

### `/spec implement`

Implement the active project. Routes to phase-specific or full implementation.

**Pre-checks:**
- Determine active project (from state file, or ask).
- Verify spec is complete: all artifacts through `implementation_plan.md` must have `status: complete`.
- If spec is incomplete: tell the user and suggest `/spec continue` to finish speccing.

**Routing:**
- If no arguments: ask "implement next phase or all remaining phases?"
- `/spec implement next`: implement a single phase, the next one.
- `/spec implement all`: implement all remaining phases.

#### Single Phase Implementation

Runs in a clean agent context. The coding agent receives a simple instruction to implement a specific phase (pointing to `implementation_plan.md`). Minimal instruction is passed in — most context is loaded from repo state.

The repository state includes: implementation plan, spec artifacts, and standing instructions. No conversation history from planning.

**The coding phase is autonomous and non-interactive.** The agent works without user assistance from start to finish. Stopping to ask the user for help is almost never acceptable — the spec and architecture should contain everything needed. The one rare exception: the agent discovers a genuinely new technical constraint that was not known at design time, which materially changes the plan (e.g., an API doesn't support the assumed operation, a framework has an undocumented limitation). In this case — and only this case — the agent may pause and surface the issue to the user for a decision.

**Loop:**
1. Read the implementation plan and identify the target phase.
2. Read the spec and architecture docs. Write a detailed phase plan to `/phase_plans/phase_N.md`:
   - **Overview:** What this phase accomplishes and why.
   - **Steps:** Ordered, specific. File(s) to change, exact changes, code snippets for signatures/key logic. Pull relevant details from architecture and component docs so the plan stands alone.
   - **Tests:** Specific automated test cases by name and what they verify. Manual test instructions if needed (UI, system integrations).
   - **Completion criteria:** Checklist of what must be true when done.
3. Build the code per the phase plan.
4. Run automated checks (lint, format, type-check, build) until clean. Follow project-specific commands from system prompt. Iterate with code until all pass.
5. Write tests per the phase plan's test section.
6. Run tests. Iterate until all pass.
7. Run automated checks again until clean (tests and fixes may introduce lint/format issues).
8. Self code-review via sub-agent. Fix issues and iterate per the [CR Iteration Loop](#cr-iteration-loop).
9. Run automated checks a final time until clean (CR fixes may introduce lint/format issues).
10. Mark the phase complete in `implementation_plan.md` (toggle checkbox only).
11. Stop and present summary of what was built.

If the agent encounters an issue, it fixes it and keeps going. The spec was reviewed and approved — trust the plan and execute.

#### Implement All

A lightweight coordinator agent runs the loop:

1. Get next incomplete phase from `implementation_plan.md`.
2. Spawn a sub-agent with clean context to implement the phase (single phase flow above, including its CR iteration).
3. Auto-commit: `"Phase N implementation of [project name]\n\n[description of work in phase]"`.
4. Show the phase summary from the subagent to the user, but don't stop. Continue to next phase.
5. Loop until all phases complete.

The coordinator has minimal context — it just manages the loop. Each implementation sub-agent gets a clean context. Code review happens inside each phase's implementation loop, not at the coordinator level.

#### CR Iteration Loop

Code review runs as a sub-agent during implementation (step 8 above). The loop ensures all feedback is addressed before the phase is considered done.

**Flow:**
1. Spawn a CR sub-agent with clean context (no coding agent history), and a simple status message "A coding agent just implemented phase X, please review using our code-review guidelines ...". It can use `git diff` to see code changes, and access specs from the repo. None of this needs to be passed in context.
2. CR produces feedback with severity labels: **critical** / **moderate** / **mild**.
3. If there are issues:
   - The coding agent addresses each one. Typically by fixing, and in rare cases with a comment (see note below)
   - After fixes, re-run CR as a new sub-agent, passing the prior feedback in a `<prior_cr_feedback>` block.
4. The re-review CR agent has two jobs:
   - **Verify prior issues:** Check that each previously flagged issue was addressed (resolved, or documented with justification).
   - **Check for new issues:** The fixes themselves may introduce new problems. It's a full review.
5. Loop steps 3-4 until CR comes back clean.

The `<prior_cr_feedback>` block is passed as structured input to the CR sub-agent. The CR prompt includes: "You may receive prior CR feedback. If present, verify each previously flagged issue has been addressed in the current code. Report any that remain unresolved and any new issues introduced by the fixes."

This same loop is used both during single phase implementation (step 8) and when `/spec cr` is invoked manually with prior feedback.

**Note:** The loop continues until no issues remain. If the coding agent doesn't want to fix an issue (for a good technical reason), it can add a code comment explaining why it's done this way, which should address the reviewer's concern (not written for the reviewer specifically, but as a genuinely useful code comment). When re-reviewing, the reviewer can choose to accept the comment and drop the issue, or push back with a feedback item that addresses the explanation: "The coder documented X, but we still feel this is an issue that needs resolution because Y. Consider Z." This loop continues until they agree.

### `/spec cr` / `/spec code_review`

Structured, spec-aware code review. Two aliases for the same command.

**Key principle:** The CR agent never has the coding agent's context. Decisions must be captured in code and specs, not in context windows. If something important is only in conversation history, it's a bug in the process.

**Flow:**

1. Determine what to review:
   - If no instructions: review `git diff` (unstaged + staged changes).
   - If given scope (e.g., "review file X" or "review phase 3"): review that scope.
2. Always runs as a sub-agent — spawned fresh, no prior context from coding.
3. Read the functional spec and relevant architecture docs for the active review.
4. Create a review plan.
5. Review systematically, producing feedback with severity labels: **critical** / **moderate** / **mild**.
6. Produce a summary of findings.

**Review dimensions:**
- **Spec compliance:** Does the implementation match what the spec says? Missing features, wrong behavior, incomplete edge case handling.
- **Code quality:** Architecture, naming, composition, error handling, test quality, code smells.
- **Consistency:** Does new code match existing patterns and conventions?
- **Project-specific standards:** Follow any code review guidelines defined in the project's system prompt config.

**Re-review (feedback verification):**

The CR skill may be passed an optional `<prior_cr_feedback>` block. When present, the review has an additional responsibility: verify that each previously flagged issue has been addressed. The output includes:
- Previously flagged issues that are now resolved (short confirmations)
- Remaining Issues
  - Previously flagged issues that are NOT resolved (with explanation, focusing on why change or comment is not acceptable).
  - New issues found

This enables a loop: CR → fix → CR with prior feedback → confirm all addressed.

The implement steps will manage this loop. The coordinator passes CR feedback from the last run back into the next CR pass until the review comes back clean.

### `/spec setup`

(Described above in its own section.)

## Project Artifacts

All spec artifacts live under `/specs/projects/PROJECT_NAME/`.

### YAML Frontmatter Convention

Every spec file has YAML frontmatter with a `status` field:

```yaml
---
status: draft
---
```

Valid statuses: `draft`, `complete`.

- Artifacts start as `draft` when first created.
- The agent marks `complete` after user confirmation.
- If a completed artifact is later edited by the agent, ideally update downstream specs (if so you can leave them all as `complete`). If making an edit and not updating downstream specs, it must be marked back to `draft`. Any downstream artifacts that depend on it should also cascade to `draft` (e.g., editing the functional spec cascades architecture, components, implementation plan, phase plans back to draft). 
- If the user edits files directly, the agent can't enforce this — but if it discovers inconsistencies, it should work with the user to resolve them. 

### File Inventory

| File | Created During | Purpose |
|------|---------------|---------|
| `project_overview.md` | new_project Step 1 | User's description of what to build. Seed document. |
| `functional_spec.md` | new_project Step 2 | Complete description of what the project does. Features, behaviors, edge cases, contracts. |
| `ui_design.md` | new_project Step 3 | UI structure, screens, navigation, interactions. Only for projects with UI. May be folded into functional_spec for small UI surfaces. |
| `architecture.md` | new_project Step 4 | Technical design. Data model, components, interfaces, patterns, test strategy. Deep enough that no technical decisions remain for coding. |
| `/components/NAME.md` | new_project Step 5 | Per-component detailed design. Optional — used when architecture.md alone isn't sufficient for larger projects. |
| `implementation_plan.md` | new_project Step 6 | Phased build order as a markdown checklist. Short — references specs for details. |
| `/phase_plans/phase_N.md` | Implementation | Self-contained plan for each phase. Written by the coding agent at the start of each phase, derived from specs and architecture. |

### Cascade Rules

When a spec artifact is modified after completion:
- `project_overview.md` → cascades everything below
- `functional_spec.md` → cascades architecture, components, implementation plan
- `architecture.md` → cascades components, implementation plan
- `components/` → cascades implementation plan
- `implementation_plan.md` → no downstream cascade (phase plans are derived fresh during implementation, either complete and code is done or draft and still draft)

Phase plans (`/phase_plans/`) are not part of the cascade chain. They are generated at implementation time from the current state of the spec and architecture docs, so they always reflect the latest approved designs.

"Cascade" means marking downstream artifacts as `draft`. They may or may not need actual changes — the agent should check and confirm with the user.

## Interaction Protocols

### Question-Asking Format

When the agent needs multiple answers from the user, questions are numbered and grouped by topic. This lets the agent ask many questions at once and the user answer them all in one message.

Format:
```
### [Topic Group]
1. [Question with enough context to answer without re-reading the spec]
2. [Question — offer concrete options where possible: "A, B, or C?"]

### [Another Topic Group]
3. [Question]
4. [Question]

Answer each question on a line, preceded by its number: `1. answer\n2. answer...`. Add as much detail as needed.
```

Guidelines:
- Group related questions under descriptive headers.
- Number sequentially across groups.
- Offer concrete options when possible ("Should we use X or Y?" not "What should we use?").
- Keep questions specific enough to answer in a sentence, but allow for longer answers.
- Don't ask questions whose answers are already in the provided context, unless pushing back.

### Pushback Protocol

The skill produces the best possible result, not just what the user asked for. Agents push back when they believe the user is making a mistake.

**When to push back:**
- **Project overview:** Risky scope, unclear goals, contradictory requirements.
- **Functional spec:** Unnecessary complexity, features that don't serve the stated goal, missing edge cases that will bite later.
- **Architecture:** Overengineering, wrong tool for the job, patterns that hurt testability or maintainability, insufficient error handling.
- **During implementation:** When coding reveals that a spec decision leads to bad technical outcomes. This is about new information — not re-litigating the plan.

**When NOT to push back:**
- Plan-level decisions during implementation (that opportunity was in the planning phase).
- After the user has already explicitly confirmed a decision the agent previously pushed back on.

**Pushback format:**
```
**Concern:** [Clear statement of the problem]

[Explanation of the tradeoff — why this matters, what it costs]

- **Option A:** [Alternative approach] — [tradeoff]
- **Option B:** [Another alternative] — [tradeoff]
- **Option C:** [Proceed as planned] — [what you accept by doing so]

What's your preference?
```

Note: may be preceded by a number if included in a question set, which is common.

The user always gets the final say. After pushback, if the user confirms their original approach, the agent proceeds. However, the pushback should be relative to the risk, and the clarity of the user's answer. A "yes" isn't enough to do something high risk or very poorly thought out, whereas "I understand, but I want X" is. A "yes" or "B" is fine for a lower risk decision.

### Agent Personas

Different phases of the process call for different thinking. The agent adopts the appropriate persona for each phase.

**Planning phases (functional spec, architecture, components):**
- Senior engineering lead/architect at a top tech company. Cares about long-term maintainability and correctness. Knows the difference between a trend and a best practice.
- Also considers: are we building the right thing for users? Not just technically sound, but actually useful.
- For UI projects: additionally thinks like a senior designer — intuitive, discoverable, low cognitive load, correct progressive disclosure for different skill levels.

**Coding phases (implementation):**
- Very skilled senior engineer IC.
- Code should explain itself through great naming and composition. Comments are for external constraints, not describing poorly structured code.
- Test-driven: writes tests that catch real breakage, don't need constant refactoring when code changes, target 95%+ coverage, reuse test helpers instead of copy-pasting across tests.
- Willing to flag when a requirement leads to bad technical outcomes — raises it rather than blindly implementing.

**Code review phases:**
- Detail-oriented senior IC who will own this code long-term.
- Direct and specific when there's a problem. Not harsh, but doesn't soften real issues.
- Reviews each file in detail, then zooms out to the whole project.
- Reads the spec to verify the right thing was implemented, not just that something was implemented.

## State Management

### Active Project Tracking

The file `.specs_skill_state/current_project.md` (git-ignored) tracks the user's active project:

```
Current Project: /specs/projects/project_name
```

- Set automatically by `/spec new_project`.
- Updated when user switches projects (new project or instructions).
- Per-worktree: since the directory is git-ignored, each worktree maintains its own state independently. This supports parallel work across worktrees.

### Progress Tracking

Progress is tracked via two mechanisms:
1. **Frontmatter status** on spec artifacts: `draft` / `complete`. Tells the agent where in the spec flow the project is.
2. **Checkboxes in `implementation_plan.md`**: Track which phases have been built.

No separate progress file — the source of truth is the artifacts themselves.

## Extensibility

The skill provides the process. Project-specific details come from the user's environment.

### What the Skill Provides
- The full spec-driven workflow.
- General guidance: "run automated checks before finishing," "write tests using the project's conventions," "follow the project's style guidelines."
- Persona-driven quality standards.

### What the User's System Prompt Provides
- Automated check commands (lint, format, type-check, test, build).
- Test framework and conventions.
- Code style guidelines.
- Code review standards (linked from system prompt config to a dedicated file, e.g., `.agents/code_review_guide.md`).
- Any project-specific constraints (async-only, specific API patterns, documentation requirements).

The skill's prompts reference these without knowing the specifics: "Run the project's automated checks as defined in your system prompt" rather than "run `uv run ./checks.sh`."

### Setup Assistance

`/spec setup` checks for commonly needed external knowledge and suggests additions:
- "I don't see automated check commands in your config. Add something like: `Run automated checks with [command]`."
- "No code review guidelines found. Consider creating `.agents/code_review_guide.md` and referencing it from your CLAUDE.md." and similar for the other cases mentioned above.
- Template snippets provided for each suggestion.

## Monorepo Support

For monorepos (multiple major sub-projects in one repository):

- `/spec setup` discovers sub-projects by scanning for root markers and confirming with the user.
- Each sub-project gets its own `/specs/projects/` directory.
- A `/specs/monorepo.md` at repo root describes the sub-project layout (paths and descriptions).
- Projects that span sub-projects live under the repo root's `/specs/projects/`.
- Projects scoped to one sub-project live under that sub-project's `/specs/projects/`.

This is lightweight — just folder structure and a description file. No special tooling beyond discovery.

## Out of Scope (V2)

The following are explicitly NOT part of V1:

- **Repo-wide specs:** Maintaining a `functional_spec.md` and `architecture.md` at the repo root that describe the entire codebase (not just projects).
- **`specs_to_update.md`:** Tracking which repo-wide specs need updating after a project ships.
- **Spec maintenance:** Guidance for keeping specs up to date as code evolves outside of projects.
- **One-time repo spec generation:** Speccing an existing codebase from scratch.
- **Nested spec hierarchy:** How to structure specs in deeply nested or complex monorepo layouts.

V1 is focused entirely on `/specs/projects/` — speccing and implementing discrete batches of work.
