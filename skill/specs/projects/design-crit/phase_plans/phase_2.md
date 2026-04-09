---
status: draft
---

# Phase 2: Reusable Phase Resource Files

## Overview

Create the seven reusable phase resource files in `references/design_crit/`. Each file provides detailed review guidance for one design concern area, addressed to the sub-agent. They follow the same three-section structure as existing `references/deep_cr/phase_*.md` files (What to Look For, Common Issues, Severity Guidance) but are adapted for spec/design review rather than code review.

## Steps

1. Create `references/design_crit/phase_completeness_review.md` — Completeness & Coverage review guide covering requirements traceability, edge cases, user flows, missing scenarios, out-of-scope clarity
2. Create `references/design_crit/phase_feasibility_review.md` — Technical Feasibility review guide covering technology fit, hard problem acknowledgment, performance assumptions, third-party dependencies, implementation ordering
3. Create `references/design_crit/phase_architecture_review.md` — Architecture review guide covering component boundaries, coupling, data flow, scalability, failure modes, extension points
4. Create `references/design_crit/phase_api_design_review.md` — API/Contract Design review guide covering interface completeness, naming consistency, error contracts, versioning, validation, idempotency
5. Create `references/design_crit/phase_data_model_design_review.md` — Data Model Design review guide covering entity completeness, relationships, query patterns, data lifecycle, migration/evolution
6. Create `references/design_crit/phase_security_design_review.md` — Security Design review guide covering trust boundaries, authentication model, authorization model, data classification, threat scenarios
7. Create `references/design_crit/phase_consistency_review.md` — Spec Consistency review guide covering cross-document terms, feature parity, contradictions, broken references, data flow agreement

## Tests

- NA (markdown/prompt writing project, no automated tests)
