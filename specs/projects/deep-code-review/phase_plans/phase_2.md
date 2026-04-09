# Phase 2: Deep CR Phase Sub-Agent Prompt and Reusable Phase Resources

## What This Phase Builds

Two categories of files:

1. **`references/deep_cr_phase_prompt.md`** — The self-contained prompt given to every deep CR phase sub-agent. Defines role, context loading, review approach, output format.

2. **7 reusable phase resource files in `references/deep_cr/`** — Domain-specific review guides loaded by sub-agents when running reusable template phases.

## Implementation Steps

- [x] Create `references/deep_cr_phase_prompt.md` following the pattern from `cr_agent_prompt.md` but scoped for phase-based review
- [x] Create `references/deep_cr/` directory
- [x] Create `references/deep_cr/phase_ui_review.md`
- [x] Create `references/deep_cr/phase_api_review.md`
- [x] Create `references/deep_cr/phase_data_model_review.md`
- [x] Create `references/deep_cr/phase_security_review.md`
- [x] Create `references/deep_cr/phase_test_review.md`
- [x] Create `references/deep_cr/phase_dependency_review.md`
- [x] Create `references/deep_cr/phase_performance_review.md`

## Key Design Decisions

- The phase prompt is self-contained (like `cr_agent_prompt.md`) — the sub-agent reads it and has everything it needs
- Phase resource files follow a consistent structure: What to Look For, Common Issues, Severity Guidance
- The phase prompt references `shared/cr_review_standards.md` for baseline standards; phase resources add domain-specific depth
- Sub-agents should flag critical issues outside their focus area if noticed, but depth is on their assigned concern

## Dependencies

- Phase 1 (complete): `references/shared/cr_review_standards.md` must exist — the phase prompt references it

## Checks/Tests

NA — this is a prompt/skill project with all markdown files.

## Attestation

- [x] All steps completed
- [x] Phase plan followed
- [x] Checks/Tests: NA
