---
name: spec
description: >
  Commands: new_project, continue, implement, task, cr (code review), deep cr (multi-phase code review), pr (address PR feedback), setup, or open guidance

  Spec-driven development: process for planning, building, and reviewing
  code projects using structured specifications. Guides users from project
  idea through functional spec, architecture, phased implementation, and
  code review — with human focused on decisions and AI agents handling
  drafting and building.

  Use when the user wants to: start a new project with specs, continue
  speccing or implementing a project, implement a planned phase, run a
  quick task without a full spec, review code against specs, or set up
  spec-driven development in a repo.
---

# Spec-Driven Development

A structured process for building software projects with specifications. You (the human) focus on decisions and review; AI agents handle drafting and building.

## How It Works

Every batch of work ("project") gets a spec folder under `/specs/projects/PROJECT_NAME/`. The skill guides you through:

1. **Planning**: Project overview → functional spec → architecture → implementation plan
2. **Building**: Implement phases autonomously, with code review built into each phase
3. **Reviewing**: Spec-aware code review that verifies implementation matches design

The skill is project-agnostic. It provides the process; your project-specific conventions (test commands, linting, style) come from your system prompt configuration.

## Command Reference

### `/spec setup`

One-time (or incremental) setup for using the skill in a repo. Adds `.specs_skill_state/` to gitignore, creates `/specs/projects/` directories, detects monorepo layout, and checks for commonly-needed configuration.

→ Read [setup command reference](references/cmd_setup.md)

### `/spec new_project` or `/spec new`

Create a new project from scratch. Walks through planning steps: project overview, functional spec, architecture (with optional component designs), and implementation plan. Sets this as your active project.

→ Read [new project command reference](references/cmd_new_project.md)

### `/spec continue` or `/spec cont`

Resume work on the active project. Shows current state and routes to the next logical action — continue speccing, implement next phase, or review code.

→ Read [continue command reference](references/cmd_continue.md)

### `/spec implement` or `/spec impl`

Implement the active project. Routes to phase-specific or full implementation. Use `/spec implement next` for one phase, `/spec implement all` for all remaining, or `/spec implement phase N` for a specific phase.

→ Read [implement command reference](references/cmd_implement.md)

### `/spec task`

Implement a one-off task without a full spec. Describe what you want inline and get the same implement loop (coding agent, code review, commit) without planning artifacts.

→ Read [task command reference](references/cmd_task.md)

### `/spec pr`

Address review feedback from a GitHub pull request. Finds the PR for the current branch, fetches unresolved comments, spawns a coding agent to address them, runs the standard CR loop, commits, pushes, and replies to each comment thread on GitHub.

→ Read [PR feedback command reference](references/cmd_pr.md)

### `/spec cr` or `/spec code_review`

Structured, spec-aware code review. Reviews `git diff` by default, or a specified scope. Always runs as a sub-agent with clean context.

→ Read [code review command reference](references/cmd_code_review.md)

### `/spec deep cr` or `/spec deep review`

Multi-phase agentic code review. Compares current branch against its fork point, designs review phases tailored to the diff, runs focused sub-agents per phase, and produces persistent review artifacts.

→ Read [deep code review command reference](references/cmd_deep_code_review.md)

### Bare `/spec` — Router

Reads current state (active project, artifact statuses) and presents relevant options. Never requires routing — direct commands always work. Can also interpret open-ended requests.

To check state: read `.specs_skill_state/current_project.md` and scan artifact frontmatter. Always show `/spec task` as an available option. If no project exists, suggest `new_project`, `task`, or `setup`. If project in progress, show state and suggest the next action.

## Project Structure

Every project lives under `/specs/projects/PROJECT_NAME/`:

| File | Created During | Purpose |
|------|---------------|---------|
| `project_overview.md` | new_project Step 1 | Your description of what to build |
| `functional_spec.md` | new_project Step 2 | Features, behaviors, edge cases, contracts |
| `ui_design.md` | new_project Step 3 | UI structure, screens, navigation (conditional) |
| `architecture.md` | new_project Step 4 | Technical design, deep enough for coding |
| `/components/NAME.md` | new_project Step 5 | Per-component detailed design (conditional) |
| `implementation_plan.md` | new_project Step 6 | Phased build order as checklist |
| `/phase_plans/phase_N.md` | Implementation | Per-phase plan written by coding agent |

## Artifact Conventions

All spec files use YAML frontmatter with a `status` field:

```yaml
---
status: draft
---
```

Valid statuses: `draft`, `complete`.

- Artifacts start as `draft` when created
- Mark `complete` after user confirmation
- If a completed artifact is edited, downstream artifacts cascade to `draft` (if they may be affected)

Dependency chain: `project_overview → functional_spec → architecture → components → implementation_plan`

Phase plans are outside the cascade — generated fresh during implementation.

## Modes

The skill operates in two modes:

- **Project mode**: Full spec planning → phased implementation. Use for anything that benefits from upfront design — new features, complex changes, multi-phase work.
- **Task mode**: Inline description → single-pass implementation. Use for well-understood, small-to-medium changes — bug fixes, small features, refactors.

Both modes use the same implement loop (coding agent → code review → commit). Task mode skips the spec planning steps.

## State Management

The file `.specs_skill_state/current_project.md` (git-ignored) tracks your active work:

```
Current Project: /specs/projects/project_name
```

or for tasks:

```
Current Task: tasks/task-slug
```

Task files live at `.specs_skill_state/tasks/[slug].md`.

This file is per-worktree (git-ignored), so you can have parallel worktrees with different active work.

## Monorepo Support

For monorepos (multiple sub-projects in one repo):

- `/spec setup` discovers sub-projects by scanning for root markers (`pyproject.toml`, `package.json`, etc.)
- Each sub-project gets its own `/specs/projects/` directory
- `/specs/monorepo.md` at repo root describes the layout
- Cross-project work lives at root `/specs/projects/`

→ See `/spec setup` for discovery and setup.

## Extensibility

The skill provides the process. Project-specific details come from your environment:

**From the skill:** The workflow, general guidance ("run automated checks"), persona-driven quality standards

**From your system prompt:** Test commands, lint/format commands, code style, CR standards, project-specific constraints

The skill references these generically: "run the project's automated checks" not "run `uv run ./checks.sh`."

→ See `/spec setup` for help configuring external knowledge.
