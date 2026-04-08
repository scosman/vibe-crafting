---
status: complete
---

# Phase 4: Update Setup Command to Add reviews/ to Gitignore

## What This Phase Builds

Updates `references/cmd_setup.md` so the setup command adds both `.specs_skill_state/` and `reviews/` to `.gitignore`, using the same check-and-append logic for both entries.

## Plan

### 1. Update Step 1 (Gitignore) in `references/cmd_setup.md`

- Change the heading description from just `.specs_skill_state/` to include `reviews/`
- Add `reviews/` to the check-and-append logic (same pattern as `.specs_skill_state/`)
- Add a note explaining why `reviews/` is gitignored (review artifacts are local, not committed)

### 2. Update Step 5 (Completion) summary

- Add `reviews/` to the summary line about gitignore

## Files

| Action | File |
|--------|------|
| Modify | `references/cmd_setup.md` |
