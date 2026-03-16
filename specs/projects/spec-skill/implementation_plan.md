---
status: draft
---

# Implementation Plan: Spec Skill

14 files total. Build in dependency order: shared foundations first, then sub-agent prompts, then the content-heavy planning steps, then command orchestration, then a final integration pass.

All files live under `spec/` (skill root) and `spec/references/`.

## Phase 1: Foundation

SKILL.md and the two shared protocol files. Establishes the routing structure, artifact conventions, and cross-cutting protocols that everything else depends on.

- [ ] `spec/SKILL.md` — Entry point. Frontmatter, process overview, command routing table, project structure/artifact conventions, state management, monorepo support, extensibility note. ~200-300 lines.
- [ ] `spec/references/pushback.md` — When/how to push back, format template, calibration guidance. ~40-50 lines.
- [ ] `spec/references/spawning_subagents.md` — What sub-agents are, what to pass, examples for common tools, fallback. ~35-45 lines.

## Phase 2: Sub-Agent Prompts

Self-contained prompts passed to spawned agents. Written before the command files that use them so the prompts are designed from the sub-agent's perspective, not retrofitted.

- [ ] `spec/references/cr_agent_prompt.md` — CR sub-agent prompt. Persona, review dimensions, severity labels, output format, re-review protocol. Must work in isolation. ~60-80 lines.
- [ ] `spec/references/coding_phase_prompt.md` — Coding sub-agent prompt. Role, context loading, phase plan writing, build loop, self-review via CR sub-agent, non-interactive rule. Must work in isolation. ~80-100 lines.

## Phase 3: Planning Step References

The four step files loaded by `cmd_new_project.md` during the planning flow. These are the core value of the skill — guidance for writing excellent specs.

- [ ] `spec/references/step_functional_spec.md` — How to write a functional spec through iterative Q&A. Process, what to cover, quality bar, pushback reference. ~100-120 lines.
- [ ] `spec/references/step_architecture.md` — How to write architecture docs. Technical coverage areas, depth requirement, 1-phase vs 2-phase decision, pushback reference. ~100-120 lines.
- [ ] `spec/references/step_ui_design.md` — UI/UX design guidance by project type. UX lens, platform conventions, pushback reference. Conditional step. ~60-80 lines.
- [ ] `spec/references/step_component_designs.md` — Per-component detailed design. When to use, what to cover, interface depth, test plans. Conditional step. ~50-60 lines.

## Phase 4: Command Orchestration

The five command files that tie everything together. Each orchestrates a user-facing command by routing to steps, sub-agents, and shared protocols.

- [ ] `spec/references/cmd_new_project.md` — Planning flow orchestrator. Pre-check, set active project, question format, planning persona, step 1 (inline), step 6 (inline), routing to step_* files for steps 2-5. ~120-150 lines.
- [ ] `spec/references/cmd_implement.md` — Implementation orchestrator. Pre-checks, routing (next/all/phase N), single-phase loop (10 steps), CR iteration loop, implement-all coordinator, coding persona. References spawning_subagents, cr_agent_prompt, coding_phase_prompt. ~120-150 lines.
- [ ] `spec/references/cmd_code_review.md` — Standalone CR command. Scope determination, sub-agent execution, re-review protocol. References spawning_subagents, cr_agent_prompt. ~60-80 lines.
- [ ] `spec/references/cmd_continue.md` — State-based routing. Read active project, determine current state from artifact statuses, route to next step or implement. ~40-50 lines.
- [ ] `spec/references/cmd_setup.md` — Repo setup. Gitignore, directory creation, monorepo detection, external knowledge check with tool detection. ~60-80 lines.

## Phase 5: Integration Pass

Cross-file review after all content is written. No new files — this is validation and consistency fixes.

- [ ] Cross-reference audit: every file reference resolves, no broken links.
- [ ] Token budget verification: SKILL.md under 500 lines, no reference file unreasonably large.
- [ ] Consistency check: terminology, persona descriptions, format conventions are uniform across files.
- [ ] Self-containment check: `coding_phase_prompt.md` and `cr_agent_prompt.md` make sense read in isolation.
- [ ] Reference loading map matches actual cross-references in the files.
- [ ] Validate with agent skills tooling (`skills-ref validate` or equivalent).
