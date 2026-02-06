# Prompts to Move into Repo

Prompt files that need to be captured from real usage and added to the repo (e.g., `prompts/` directory), then linked from the README.

## Spec & Planning Phase (Tool 1: Cursor + Opus)

- [ ] **Challenge Prompt** — "Ask it to challenge you" prompt used during early spec steps to force pushback on assumptions and weak points
- [ ] **Functional Spec Prompt** — Prompt to generate `functional_spec.md` from `project_overview.md`
- [ ] **Architecture Prompt** — Prompt to generate `architecture.md` from functional spec
- [ ] **Component Design Prompt** — Prompt to generate individual component designs + test plans
- [ ] **Implementation Plan Prompt** — Prompt to generate phased implementation ordering
- [ ] **Per-Phase Plan Prompt** — Prompt to generate detailed `phase_plan/phase_N.md` files

## Implementation Phase (Tool 2: Claude Code + GLM 4.7)

- [ ] **Phase Instructions** (`phase_instructions.md`) — Standing instructions for the agent during phase implementation (ask questions only at start, must complete tests/lint/format, write manual test plan if needed)
- [ ] **Implement Phase Prompt** — The "implement phase N" prompt pattern given to the coding agent
- [ ] **Manual Test Plan Prompt** — Prompt for generating `phase_N_manual_test_plan.md` for UI/untestable components

## Review Phase

- [ ] **Agent Phase Code Review Prompt** — Prompt to review `git diff` against spec and phase plan with smarter model
- [ ] **End-to-End Agentic Code Review Prompt** — Full codebase review prompt for GLM 4.7 that produces `issue_summary`

## Testing Phase (Tool 1: Cursor + Opus)

- [ ] **Manual Test Execution Prompt** — Prompt for interactively running through manual test plans and reporting results
- [ ] **Diagnostic UI Prompt** — Prompt for generating diagnostic/test UI within the app itself
