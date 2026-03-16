---
status: complete
---

# Implementation Plan: Spec Skill

Follow the architecture doc, copying the needed content into files.

 - [x] Build all files with this process:
 - find headers for each file
 - Use CLI commands with line-ranges to write files, not reading/writing tokens manually
 - sanity check each: head and tail check. No header mid-file for another file.

Cross-file review after all content is written. No new files — this is validation and consistency fixes.

- [x] Cross-reference audit: every file reference resolves, no broken links.
- [x] Read architecture and each file, look for copy bugs
- [x] Token budget verification: SKILL.md under 500 lines, no reference file unreasonably large.
- [x] Consistency check: terminology, persona descriptions, format conventions are uniform across files.
- [x] Self-containment check: `coding_phase_prompt.md` and `cr_agent_prompt.md` make sense read in isolation.
- [x] Reference loading map matches actual cross-references in the files.
- [ ] Validate with agent skills tooling (`skills-ref validate` or equivalent).

