---
status: complete
---

# Implementation Plan: Implement Coordinator

## Phases

- [x] Phase 1: Core restructuring — Rewrite `cmd_implement.md` as the manager spec and restructure `coding_phase_prompt.md` for three invocation modes (initial, CR fix, commit). These two files are tightly coupled and must be updated together.
- [x] Phase 2: Supporting files and validation — Update `spawning_subagents.md` with the resume pattern, verify `cr_agent_prompt.md` needs no changes, then cross-reference audit and consistency check across all four files.
