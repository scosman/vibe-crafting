---
status: draft
---

# Architecture: Spec Skill

## Overview

The spec skill is a directory of markdown files following the [Agent Skills](https://agentskills.io/specification) format. `SKILL.md` is the entry point and command router. All substantive instructions live in `references/` files, loaded on demand via progressive disclosure.

The skill contains no executable code — it is entirely prompt/instruction engineering packaged as an agent skill. The "architecture" is the file structure, the content design of each file, and how they reference each other.

### Core Pattern

SKILL.md stays loaded for the entire session. It handles command routing and holds conventions that every command needs (artifact format, state management, project structure). When a command is invoked, the agent loads the corresponding reference file, which contains the full instructions for that command. Some commands load additional reference files (sub-step files, sub-agent prompts, shared protocols) as needed.

This is two-level progressive disclosure: SKILL.md (always loaded) → command reference (loaded per command) → sub-references (loaded as needed within a command). Matches the "nested progressive disclosure" the agent skills spec supports.

## Skill Directory Structure

```
spec/
├── SKILL.md
└── references/
    ├── cmd_setup.md
    ├── cmd_new_project.md
    ├── step_functional_spec.md
    ├── step_ui_design.md
    ├── step_architecture.md
    ├── step_component_designs.md
    ├── cmd_continue.md
    ├── cmd_implement.md
    ├── cmd_code_review.md
    ├── coding_phase_prompt.md
    ├── cr_agent_prompt.md
    ├── pushback.md
    └── spawning_subagents.md
```

**Naming conventions:**
- `cmd_` prefix: top-level command reference files. One per command.
- `step_` prefix: sub-step files for the `/spec new_project` flow. Loaded by `cmd_new_project.md` as the user progresses through planning steps.
- No prefix: sub-agent prompts (passed to spawned agents) and shared protocol files.

All reference files are flat in `references/` — no subdirectories. The prefixes provide visual grouping.

## SKILL.md Design

**Target size:** 200-300 lines. Well within the 500-line recommendation.

SKILL.md has two jobs: (1) route commands to reference files, and (2) hold shared context that every command needs.

### Frontmatter

```yaml
---
name: spec
description: >
  Commands: new_project, continue, implement, cr (code review), setup, or open guidance
  
  Spec-driven development: process for planning, building, and reviewing
  code projects using structured specifications. Guides users from project
  idea through functional spec, architecture, phased implementation, and
  code review — with human focused on decisions and AI agents handling
  drafting and building.


  Use when the user wants to: start a new project with specs, continue
  speccing or implementing a project, implement a planned phase, review
  code against specs, or set up spec-driven development in a repo.
---
```

### Body Sections

**1. Process Overview** (~20 lines)
Brief description of the spec-driven development process. What it is, what it produces, how the human and agent collaborate. The functional spec's "Core Concept" section, condensed.

**2. Command Reference** (~80 lines)
A section per command, each containing:
- Trigger phrase(s) and aliases
- 2-3 line description of what it does
- Instruction to read the corresponding reference file
- Example: `### /spec new_project` / `Plan a new project from scratch...` / `→ Read [cmd_new_project.md](references/cmd_new_project.md)`

Commands listed: `setup`, `new_project`, `continue`, `implement` (with `next`, `all`, `phase N` variants), `cr` / `code_review`.

**Bare `/spec` handling:** Read state (current project, artifact statuses). If no project: suggest `new_project` or `setup`. If project in progress: show current state and suggest the logical next action (continue speccing, implement next phase, etc.). Never requires routing through bare `/spec` — direct commands always work. Can also handle open-ended requests by interpreting intent and routing.

**3. Project Structure** (~25 lines)
The `/specs/projects/PROJECT_NAME/` layout and file inventory table from the functional spec. This is here (not in a reference) because every command needs to know what artifacts exist and where they live.

File inventory table:
| File | Created During | Purpose |
|------|---------------|---------|
| `project_overview.md` | new_project Step 1 | User's description of what to build |
| `functional_spec.md` | new_project Step 2 | Features, behaviors, edge cases, contracts |
| `ui_design.md` | new_project Step 3 | UI structure, screens, navigation (conditional) |
| `architecture.md` | new_project Step 4 | Technical design, deep enough for coding |
| `/components/NAME.md` | new_project Step 5 | Per-component detailed design (conditional) |
| `implementation_plan.md` | new_project Step 6 | Phased build order as checklist |
| `/phase_plans/phase_N.md` | Implementation | Per-phase plan written by coding agent |

**4. Artifact Conventions** (~30 lines)
- YAML frontmatter with `status: draft | complete`.
- Lifecycle: created as `draft`, marked `complete` after user confirms, reverted to `draft` if modified post-completion.
- Cascade rules: editing a completed artifact cascades `draft` status to all downstream artifacts. The dependency chain: `project_overview → functional_spec → architecture → components → implementation_plan`. Phase plans are outside the cascade (generated fresh at implementation time).

**5. State Management** (~15 lines)
- `.specs_skill_state/current_project.md` (git-ignored) tracks active project.
- Format: `Current Project: /specs/projects/project_name`
- Set by `new_project`, updated on project switch.
- Per-worktree since git-ignored.

**6. Monorepo Support** (~15 lines)
Brief description: sub-projects get their own `/specs/projects/`, cross-project work lives at root, `/specs/monorepo.md` describes layout. Reference `/spec setup` for discovery.

**7. Extensibility Note** (~15 lines)
The skill provides the process. Project-specific details (test commands, lint commands, style guides, CR standards) come from the user's system prompt configuration. The skill's instructions reference these generically: "run the project's automated checks" not "run `uv run ./checks.sh`."

## Reference File Designs

### cmd_setup.md

**Purpose:** One-time repo setup for using the skill.

**Size:** ~60-80 lines.

**Content:**
1. **Gitignore step:** Add `.specs_skill_state/` to `.gitignore`. Create file if needed, append if not present. Don't duplicate.
2. **Directory creation:** Create `/specs/projects/` if missing.
3. **Monorepo detection:**
   - Scan for root markers in subdirectories: `pyproject.toml`, `package.json`, `go.mod`, `Cargo.toml`, `build.gradle`, `pom.xml`, `.csproj`, etc.
   - Ask user to confirm: "Is this a monorepo? I found these potential sub-projects: [list]. Are there others?" Always ask even if none found.
   - If monorepo: create `/specs/projects/` in each sub-project, write `/specs/monorepo.md` with sub-project descriptions. Ask user for enough detail about each sub-project for the file to be useful.
4. **External knowledge check:**
   - Scan system prompt config for: automated check commands, test framework/commands, lint/format commands, code review guidelines, code style conventions.
   - Try to detect the agent tool in use (look for `.cursor/`, `CLAUDE.md`, `AGENTS.md`, `Codex`, etc.). Use the detected tool name in suggestions when possible, fall back to generic "your agent's system prompt configuration" if unclear.
   - For each missing item: suggest where to add it, provide a template snippet.
5. **Idempotent:** Won't clobber existing setup. Only extends what's missing.

**References loaded:** None — self-contained.

### cmd_new_project.md

**Purpose:** The primary planning flow. Orchestrates steps 1-6 of project creation.

**Size:** ~120-150 lines.

**Content:**

**Pre-check:** If there's an active project in progress, warn and suggest `/spec continue`.

**Set active project:** Write to `.specs_skill_state/current_project.md` immediately after creating the project folder. Don't wait until complete.

**Question-asking format** (inline, ~15 lines): Numbered questions grouped by topic. Sequential numbering across groups. Offer concrete options. Instruction at the end: "Answer each by number." This format is used by all planning steps.

**Planning persona** (inline, ~10 lines): Senior engineering lead/architect at a top tech company. Cares about long-term maintainability. Also considers: are we building the right thing for users? For UI projects, additionally thinks like a senior designer. This persona applies to all planning steps.

**Step 1: Project Overview** (inline, ~20 lines):
- Ask user to describe what they want to build. Prompt for: what it does, who it's for, why, technical requirements, constraints.
- Create `project_overview.md` with `status: draft` frontmatter.
- Keep very close to what the user wrote. Minor spelling/grammar only.
- Confirm with user, mark `status: complete`.

**Step 2: Functional Spec:**
- → Read [step_functional_spec.md](references/step_functional_spec.md) and follow it.

**Step 3: UI Design (conditional):**
- Only if the project has a user-facing interface (detected from overview + functional spec). Offer to skip if user prefers ("just build backend/APIs").
- → Read [step_ui_design.md](references/step_ui_design.md) and follow it.

**Step 4: Architecture:**
- → Read [step_architecture.md](references/step_architecture.md) and follow it.

**Step 5: Component Designs (conditional):**
- The agent decides based on project size. Skip for small projects where architecture.md covers everything.
- → Read [step_component_designs.md](references/step_component_designs.md) and follow it.

**Step 6: Implementation Plan** (inline, ~25 lines):
- Read all completed spec artifacts.
- Design phased build order: logical by dependencies, each phase roughly one coherent unit of work, small enough to review in one sitting but not so small that many tiny CRs burden the user.
- Small projects: single phase is fine.
- Write `implementation_plan.md` with markdown checkboxes for progress tracking.
- Keep short — ordered checklist referencing specs for details, not a re-statement.
- Confirm with user, mark `status: complete`.

**End-of-flow note:** The `new_project` flow ends after step 6. Phase plans are written by the coding agent at the start of each implementation phase, not during planning.

**References loaded:** `step_functional_spec.md`, `step_ui_design.md`, `step_architecture.md`, `step_component_designs.md` — each loaded only for its step. Also loads `pushback.md` (loaded once, used across steps).

### step_functional_spec.md

**Purpose:** Instructions for writing a functional spec through iterative Q&A with the user.

**Size:** ~100-120 lines.

**Content:**
1. **Process:** Read project overview → identify gaps → ask clarifying questions in rounds (high-level first, then details) → draft functional_spec.md → present for review → iterate → confirm → mark complete.
2. **What to cover.** Examples of sections (not rigid — adapt to the project):
   - Features and behavior: what it does, user flows, usage patterns.
   - Edge cases and error handling: what happens when things go wrong, boundaries.
   - Input/output contracts: for APIs, CLIs, libraries — formats, validation.
   - Configuration and defaults: what's configurable, sensible defaults.
   - Constraints: performance, compatibility, security.
   - For UI projects: high-level screens/views and navigation at functional level (details in step 3).
3. **Quality bar:** The spec should be complete enough that someone unfamiliar with the project could understand what is being built and why, from this document alone. Every behavior and decision should be explicit — don't leave gaps for the coder to fill.
4. **Pushback:** → Load [pushback.md](references/pushback.md). Challenge feature decisions, unnecessary complexity, scope that doesn't match stated goals. After writing the draft, review the overview and challenge anything risky, underspecified, or likely to cause downstream problems.
5. **Completion:** Create with `status: draft`, mark `status: complete` after user confirms.

**References loaded:** `pushback.md` (if not already loaded by a prior step).

### step_ui_design.md

**Purpose:** UI/UX design for projects with user-facing interfaces.

**Size:** ~60-80 lines.

**Content:**
1. **Process:** Read overview + functional spec → propose UI structure → iterative Q&A → draft `ui_design.md` → review → iterate → confirm.
2. **What to cover** (varies by project type):
   - Web: page layout, page content, navigation, component breakdown, responsive behavior, modals/sidebars.
   - Mobile: screen flow, screen content, navigation patterns, modals/sheets/bottom sheets, platform conventions (HIG/Material).
   - CLI: command structure, output formatting, interactive prompts, help text.
3. **UX lens:** Intuitive, discoverable, low cognitive load, correct progressive disclosure for different skill levels. Follow platform conventions. Don't reinvent standard patterns.
4. **Pushback:** → Load [pushback.md](references/pushback.md). Challenge UX decisions — discoverability, cognitive load, progressive disclosure, navigation structure, platform convention violations.
5. **Completion:** `status: draft` → `status: complete` after user confirms.

**References loaded:** `pushback.md`.

### step_architecture.md

**Purpose:** Instructions for writing a technical architecture doc through iterative Q&A.

**Size:** ~100-120 lines.

**Content:**
1. **Process:** Read overview + functional spec (+ ui_design if present) → ask technical questions → draft `architecture.md` → review → iterate → confirm.
2. **What to cover:**
   - Data model: entities, relationships, storage approach.
   - Component breakdown: major classes/modules, responsibilities, interactions.
   - Public interfaces: function signatures, API contracts, protocols between components.
   - Design patterns: which ones and why. Framework and library choices with rationale.
   - Technical challenges: identify anything non-trivial and design the solution now. Hard problems get solved here, not during coding.
   - Error handling strategy: how errors propagate, what's recoverable, logging approach.
   - Testing strategy: kinds of tests, coverage targets, frameworks. Test approach per component.
3. **Depth requirement:** The architecture must be deep enough that no significant technical decisions remain for the coding agent. Classes, functions, overall flow, key algorithms, test cases, key dependencies — all specified. The coding agent executes a well-defined plan, it doesn't design.
4. **1-phase vs 2-phase decision:** The agent decides whether the project needs component designs (step 5) or if everything fits in architecture.md. For projects where a single architecture doc would exceed ~300 lines of technical content, or where individual components have enough internal complexity to warrant their own docs, recommend component designs. Communicate this decision to the user.
5. **Pushback:** → Load [pushback.md](references/pushback.md). Challenge technical decisions: overengineering, wrong tool for the job, patterns that hurt testability/maintainability, insufficient error handling, framework choices that don't fit. Can also push back on functional requirements that add disproportionate technical complexity — offer alternatives spanning functional spec and architecture.
6. **Completion:** `status: draft` → `status: complete` after user confirms.

**References loaded:** `pushback.md`.

### step_component_designs.md

**Purpose:** Per-component detailed designs when architecture.md alone isn't sufficient.

**Size:** ~50-60 lines.

**Content:**
1. **When to use:** Agent decided during architecture step that component designs are needed.
2. **What each component doc covers:**
   - Purpose and scope.
   - Public interface: full function/method signatures with parameter types and return types.
   - Internal design approach: key algorithms, data flow, state management.
   - Dependencies: what this component depends on, what depends on it.
   - Test plan: specific test cases by name and what they verify.
3. **One file per component** in `/components/component_name.md`.
4. **Review:** Quick user review. "Component designs written to [folder]. Ready to continue to implementation plan?"
5. **Completion:** `status: draft` → `status: complete` after user confirms.

**References loaded:** None additional.

### cmd_continue.md

**Purpose:** Resume work on the active project. The "what should I do next?" command.

**Size:** ~40-50 lines.

**Content:**
1. **Read state:** Load `.specs_skill_state/current_project.md` to find active project.
2. **No active project:** List project directories under `/specs/projects/`. Ask which to work on. Set it as active.
3. **Determine current state:** Check frontmatter `status` on each artifact in dependency order:
   - If any spec artifact is missing or `draft`: resume the `new_project` flow at the next incomplete step. Load the corresponding step reference file and proceed.
   - If all specs are `complete` but implementation phases remain: treat as `/spec implement` — ask whether to implement next phase or all remaining.
4. **Confirm** before proceeding to the next action.

**References loaded:** Routes to other command/step references based on state. Doesn't load them preemptively — determines the right one first, then loads it.

### cmd_implement.md

**Purpose:** Implement the active project. Routes to single-phase or full implementation.

**Size:** ~120-150 lines.

**Content:**

**Pre-checks:**
- Determine active project from state file, or ask.
- Verify all spec artifacts through `implementation_plan.md` have `status: complete`. If not: tell user and suggest `/spec continue`.

**Routing:**
- `/spec implement`: ask "next phase or all remaining?"
- `/spec implement next`: single phase.
- `/spec implement all`: all remaining phases.
- `/spec implement phase N`: specific single phase.

**Single Phase Implementation:**
Instructions for running one phase. This is an autonomous loop — the agent works without user assistance from start to finish.

The coding persona (inline, ~8 lines): very skilled senior IC. Code explains itself through naming and composition. Comments for external constraints only. Test-driven: high coverage, reusable test helpers, tests that catch real breakage. Willing to flag when a requirement leads to bad technical outcomes.

Steps (the loop):
1. Read `implementation_plan.md`, identify target phase.
2. Read spec and architecture docs. Write phase plan to `/phase_plans/phase_N.md`:
   - Overview: what and why.
   - Steps: ordered, specific. Files to change, exact changes, code snippets for signatures.
   - Tests: specific automated test cases by name. Manual test instructions if needed.
   - Completion criteria: checklist.
3. Build the code per the plan.
4. Run automated checks (lint, format, type-check, build). Follow project-specific commands from system prompt. Iterate until clean.
5. Write tests per the phase plan's test section.
6. Run tests. Iterate until passing.
7. Run automated checks again (tests/fixes may introduce lint/format issues). Iterate until clean.
8. Self code-review via sub-agent: → Read [spawning_subagents.md](references/spawning_subagents.md) for how to spawn. Pass the prompt from [cr_agent_prompt.md](references/cr_agent_prompt.md) to the sub-agent. Include a status message: "A coding agent just implemented phase N of [project]. Review the changes using `git diff`." See CR Iteration Loop below.
9. Run automated checks one final time (CR fixes may introduce issues). Iterate until clean.
10. Mark phase complete in `implementation_plan.md` (toggle checkbox only).
11. Stop. Present summary of what was built.

**CR Iteration Loop:**
1. Spawn CR sub-agent with clean context. Pass the CR prompt from `cr_agent_prompt.md`.
2. CR returns feedback with severity labels.
3. If issues exist: the coding agent fixes each one (or in rare cases, adds a code comment explaining the technical rationale for the current approach — a genuinely useful comment, not written for the reviewer).
4. After fixes: spawn a new CR sub-agent, passing the same CR prompt plus the prior feedback in a `<prior_cr_feedback>` block.
5. The re-review agent verifies prior issues are addressed AND checks for new issues from fixes.
6. Loop until CR returns clean.

**Non-interactive rule:** The coding phase is autonomous. The agent does not stop to ask the user for help except in one rare case: a genuinely new technical constraint not known at design time that materially changes the plan (e.g., an API doesn't support an assumed operation). This is the only acceptable reason to pause.

**Implement All:**
A lightweight coordinator that:
1. Gets next incomplete phase from `implementation_plan.md`.
2. Spawns a sub-agent with clean context to run the single-phase flow above (→ [spawning_subagents.md](references/spawning_subagents.md)). Pass: phase number, project path, instruction to follow the single-phase implementation flow.
3. Auto-commits: `"Phase N implementation of [project name]\n\n[description of work]"`.
4. Shows phase summary to user but doesn't stop. Continues to next phase.
5. Loops until all phases complete.

The coordinator has minimal context — it manages the loop. Each phase sub-agent gets clean context. CR happens inside each phase, not at coordinator level.

**References loaded:** `spawning_subagents.md`, `cr_agent_prompt.md`. For implement-all, also loads `coding_phase_prompt.md` to pass to each phase sub-agent.

### cmd_code_review.md

**Purpose:** Standalone spec-aware code review. Aliases: `/spec cr`, `/spec code_review`.

**Size:** ~60-80 lines.

**Content:**

**Key principle:** The CR agent never has the coding agent's context. Decisions must be captured in code and specs, not in context windows.

**Determine scope:**
- No arguments: review `git diff` (unstaged + staged).
- Given scope: review that scope ("review file X", "review phase 3", etc.).

**Execution:** Always as a sub-agent. → Read [spawning_subagents.md](references/spawning_subagents.md) for how. Pass the prompt from [cr_agent_prompt.md](references/cr_agent_prompt.md), plus scope description.

**Re-review:** If the user provides prior feedback (or this is called from a loop), pass it as `<prior_cr_feedback>` to the CR sub-agent. The CR prompt handles verification of prior issues.

**Post-review:** Present findings to user. If issues exist and user wants fixes: user or coding agent addresses them, then re-run CR with prior feedback.

**References loaded:** `spawning_subagents.md`, `cr_agent_prompt.md`.

### coding_phase_prompt.md

**Purpose:** The self-contained prompt passed to a coding sub-agent when spawning it to implement a phase. This is what the sub-agent sees — nothing else from the conversation.

**Size:** ~80-100 lines.

**Content:**
This file IS the prompt. It is written in the second person ("you are..."), addressed to the coding sub-agent. It should be passable directly as the sub-agent's task description.

Key sections:
1. **Role:** You are a senior software engineer implementing phase N of [project]. (The caller fills in N and project path.)
2. **Context loading:** Read `implementation_plan.md` to identify the phase. Read the spec artifacts (`functional_spec.md`, `architecture.md`, components if they exist) for context.
3. **Phase plan:** Write a detailed phase plan to `/phase_plans/phase_N.md` before coding. Structure: overview, ordered steps with file-level specifics, test cases, completion criteria.
4. **Build loop:** Build → automated checks → tests → automated checks → self-review → automated checks. Each step iterates until clean. Use project-specific commands from system prompt.
5. **Self-review via sub-agent:** → Reference [cr_agent_prompt.md](references/cr_agent_prompt.md). Spawn a CR sub-agent. CR iteration loop until clean.
6. **Non-interactive:** Work autonomously. Don't ask the user. The one exception: genuinely new technical constraint not known at design time.
7. **Completion:** Mark phase checkbox in `implementation_plan.md`. Present summary.

**Design note:** This file duplicates some content from `cmd_implement.md` (the phase loop steps). That's intentional — this prompt must be self-contained because it's passed to a sub-agent that has no access to cmd_implement.md. The sub-agent only sees this prompt and the repo.

**References loaded by the sub-agent:** `cr_agent_prompt.md` (for self-review). The sub-agent reads this reference from the repo at runtime, it doesn't need to be passed in context. (This works because the skill is installed in the repo or accessible from it.)

### cr_agent_prompt.md

**Purpose:** The self-contained prompt passed to a CR sub-agent. Used by both `cmd_implement.md` (step 8 self-review) and `cmd_code_review.md` (standalone review).

**Size:** ~60-80 lines.

**Content:**
Written in second person, addressed to the CR sub-agent. Passable directly as the sub-agent's task description.

1. **Role and persona:** You are a detail-oriented senior IC who will own this code long-term. Direct and specific when there's a problem — polite but doesn't soften real issues. Reviews each file in detail, then zooms out to the whole change.
2. **Context loading:** Read the project's functional spec and relevant architecture docs. Use `git diff` to see code changes. Read any specs referenced by the implementation plan for the relevant phase.
3. **Review plan:** Create a review plan before starting. Identify files to review, specs to check against, areas of concern.
4. **Review dimensions:**
   - Spec compliance: does the implementation match the spec? Missing features, wrong behavior, incomplete edge cases.
   - Code quality: architecture, naming, composition, error handling, test quality, code smells.
   - Consistency: does new code match existing patterns and conventions?
   - Project-specific standards: follow any code review guidelines in the project's system prompt config.
5. **Severity labels:** Each issue gets one: **critical** (must fix), **moderate** (should fix), **mild** (consider fixing / nit).
6. **Output format:** Structured feedback. Group by file or concern. Each item: severity, description, suggestion.
7. **Re-review protocol:** If a `<prior_cr_feedback>` block is present:
   - Verify each previously flagged issue: resolved, or documented with justification.
   - Report: previously resolved (short confirmations), still unresolved (with explanation of why the fix or comment is insufficient), new issues from the fixes.
8. **Output:** Summary of findings. If clean: explicit "no issues found" confirmation.

**References loaded:** None — self-contained by design. The sub-agent reads spec files from the repo, not from skill references.

### pushback.md

**Purpose:** Shared protocol for how and when to push back on user decisions. Loaded by planning steps (functional spec, UI design, architecture) and potentially during implementation.

**Size:** ~40-50 lines.

**Content:**
1. **When to push back:**
   - Risky scope, unclear goals, contradictory requirements.
   - Unnecessary complexity, features that don't serve stated goals.
   - Overengineering, wrong tool, patterns that hurt testability/maintainability.
   - During implementation: only when coding reveals genuinely new information that changes the calculus.
2. **When NOT to push back:**
   - Plan-level decisions during implementation (that ship sailed in planning).
   - After the user has already explicitly confirmed a decision the agent previously pushed back on.
3. **Format:**
   ```
   **Concern:** [Clear statement of the problem]

   [Explanation of the tradeoff — why this matters, what it costs]

   - **Option A:** [Alternative] — [tradeoff]
   - **Option B:** [Another alternative] — [tradeoff]
   - **Option C:** [Proceed as planned] — [what you accept]

   What's your preference?
   ```
4. **Calibration:** Pushback intensity matches risk. Low-risk decision with a "yes" → proceed. High-risk decision needs explicit confirmation that the user understands the tradeoff ("I understand, but I want X"), not just "yes."
5. **Final say:** The user always decides. After pushback and explicit confirmation, the agent proceeds without relitigating.

**References loaded:** None — self-contained.

### spawning_subagents.md

**Purpose:** Explain the sub-agent pattern and how to use it across different tools. Short reference loaded once per session when sub-agents are needed.

**Size:** ~35-45 lines.

**Content:**
1. **What and why:** Sub-agents are fresh agent contexts with no conversation history from the current session. Used when clean context matters (CR shouldn't see coding thinking, each phase starts fresh). The sub-agent sees only what you pass it (a prompt) plus the repo.
2. **What to pass:** A prompt/task description (the caller specifies this — typically a reference file's content or a short instruction pointing to one). Optionally structured data like `<prior_cr_feedback>`. Never pass conversation history.
3. **Examples for common tools:**
   - Claude Code: `Task()` tool or similar sub-agent mechanism.
   - Cursor: sub-agent spawning capability.
   - General: "use your sub-agent or task-spawning capability."
4. **Fallback:** If the tool doesn't support sub-agents, approximate by: clearing context and starting a new conversation with only the sub-agent prompt. Note this is less ideal since it can't run in parallel with the parent.

**References loaded:** None — self-contained.

## Reference Loading Map

This shows which files reference which. Arrows mean "instructs the agent to load."

```
SKILL.md
├── cmd_setup.md
├── cmd_new_project.md
│   ├── step_functional_spec.md
│   │   └── pushback.md
│   ├── step_ui_design.md
│   │   └── pushback.md
│   ├── step_architecture.md
│   │   └── pushback.md
│   └── step_component_designs.md
├── cmd_continue.md
│   └── (routes to any step_* or cmd_implement based on state)
├── cmd_implement.md
│   ├── spawning_subagents.md
│   ├── coding_phase_prompt.md  (passed to coding sub-agent)
│   │   └── cr_agent_prompt.md  (sub-agent loads from repo)
│   └── cr_agent_prompt.md      (for CR iteration loop)
└── cmd_code_review.md
    ├── spawning_subagents.md
    └── cr_agent_prompt.md      (passed to CR sub-agent)
```

**Key observations:**
- `pushback.md` is loaded once and reused across planning steps. Three references but one load.
- `spawning_subagents.md` is loaded once when first needed (implement or CR).
- `cr_agent_prompt.md` is referenced by both implement and code_review. Content is identical — it's the same CR process.
- `coding_phase_prompt.md` is only used by implement-all (passed to phase sub-agents). For single-phase implement, the agent follows cmd_implement.md directly.

## Design Principles

### Self-Containment for Sub-Agent Prompts

`coding_phase_prompt.md` and `cr_agent_prompt.md` must be fully self-contained. They are passed to sub-agents that have no access to the parent conversation, SKILL.md, or other reference files (except what's in the repo).

This means some content is intentionally duplicated between cmd_implement.md and coding_phase_prompt.md. The command file describes the process from the orchestrator's perspective; the prompt file describes it from the sub-agent's perspective. Same behavior, different audience.

The sub-agent CAN read reference files from the repo at runtime (e.g., the CR sub-agent spawned by the coding sub-agent can read cr_agent_prompt.md from the skill directory). Prompts should reference skill files by relative path when the sub-agent needs them.

### Progressive Disclosure Token Budget

Rough token budgets per session type:
- **Routing only** (bare `/spec`): SKILL.md only. ~200-300 lines.
- **Planning step** (e.g., writing functional spec): SKILL.md + cmd_new_project.md + step file + pushback.md. ~400-550 lines total.
- **Implementation**: SKILL.md + cmd_implement.md + spawning_subagents.md. ~380-480 lines. Sub-agent prompts are passed out-of-context to sub-agents, not loaded into the parent.
- **Code review**: SKILL.md + cmd_code_review.md + spawning_subagents.md. ~330-430 lines. CR prompt passed to sub-agent.

All well within reasonable context budgets.

### Consistent Language

Use "project" to mean a unit of work under `/specs/projects/`. Use "sub-project" for monorepo children. Never use "project" ambiguously. Use "phase" for implementation phases. Use "step" for planning steps within new_project.

Use "sub-agent" consistently (not "sub-process", "child agent", "spawned agent" interchangeably).

## Cross-Tool Compatibility

The skill targets any agent that supports the [Agent Skills](https://agentskills.io/specification) format. It does not assume a specific tool.

**Tool detection** (best-effort, in cmd_setup.md): look for `.cursor/`, `CLAUDE.md`, `AGENTS.md`, `.codex/` etc. to guess the tool. Use detected tool name in setup suggestions. Fall back to "your agent's system prompt configuration" when unsure.

**Sub-agent spawning** is described generically in spawning_subagents.md with examples for known tools and a fallback for unknown tools.

**System prompt config** is referenced generically throughout: "follow the project's automated check commands as defined in your system prompt" not "as defined in CLAUDE.md."

## Testing Strategy

This is a writing project — the output is markdown files. Testing is:

1. **Structural validation:** Verify the skill passes the agent skills `quick_validate.py` or `skills-ref validate` tool (checks frontmatter, naming conventions).
2. **Token budget check:** Verify SKILL.md is under 500 lines. Verify no individual reference file exceeds ~150 lines (soft limit — longer is fine if justified, but signals a possible split).
3. **Reference integrity:** Every file referenced from SKILL.md or another reference exists. No broken links.
4. **Self-containment check:** `coding_phase_prompt.md` and `cr_agent_prompt.md` make sense when read in isolation, without any other reference file loaded.
5. **End-to-end walkthrough:** Manually trace through each command path (setup, new_project full flow, continue at various states, implement single phase, implement all, standalone CR) and verify the agent would load the right files in the right order and receive coherent instructions at each step.
6. **Real usage:** Use the skill on a real project and iterate based on what works and what doesn't.
