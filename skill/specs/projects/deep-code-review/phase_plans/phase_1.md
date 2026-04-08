# Phase 1: Extract shared review standards and update existing CR prompt

## What

Extract review dimensions (sections 1-4), severity labels, and the issues-to-watch-for list from the functional spec into a new shared reference file. Update the existing CR prompt to reference it instead of inlining those sections.

## Files

### New
- `references/shared/cr_review_standards.md` — Self-contained reference file with review dimensions, issues to watch for, and severity labels. Addressed to whatever agent loads it.

### Modified
- `references/cr_agent_prompt.md` — Replace inline "Review Dimensions" (sections 1-4) and "Severity Labels" with a single-line reference to the new shared file.

## Steps

1. Create `references/shared/cr_review_standards.md` with three sections:
   - **Review Dimensions** — Move verbatim from `cr_agent_prompt.md` sections 1-4 (Spec/Task Compliance, Code Quality, Consistency, Project-Specific Standards)
   - **Issues to Watch For** — The list from the functional spec (bugs, poor names, code in wrong place, editing globals, json.dumps, GPL deps, dead code, inconsistent error handling, missing validation, hardcoded values)
   - **Severity Labels** — Move verbatim from `cr_agent_prompt.md` (Critical/Moderate/Mild definitions)

2. Modify `references/cr_agent_prompt.md`:
   - Remove sections "## Review Dimensions" through "### 4. Project-Specific Standards"
   - Remove section "## Severity Labels" with its content
   - Insert reference line in their place: `→ Read [references/shared/cr_review_standards.md](shared/cr_review_standards.md) for review dimensions, issues to watch for, and severity labels. Apply all of them.`
   - Keep everything else unchanged: role/persona, context loading, create a review plan, output format, re-review protocol

## Verification

- The shared file is self-contained and makes sense when read in isolation
- The CR prompt still reads coherently with the reference replacing inline content
- No content is lost — all review dimensions and severity labels appear in the shared file
- The issues-to-watch-for list from the functional spec is included in the shared file

## Attestation

- Checks/Tests: NA (prompt/skill project — all files are markdown)
