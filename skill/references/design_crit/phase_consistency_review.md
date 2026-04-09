# Spec Consistency Review

## What to Look For

### Cross-Document Term Consistency
- The same concept uses the same name in every spec file — not "user" in the functional spec, "account" in the architecture, and "member" in the component spec
- Acronyms and abbreviations are defined on first use and used consistently afterward
- Technical terms match their standard industry meaning — the spec doesn't redefine well-known terms to mean something different
- Glossary or terminology section (if one exists) matches actual usage throughout the specs

### Feature Parity
- Bidirectional mismatch detection: identify features described in the functional spec but missing from the architecture, AND features designed in the architecture that have no corresponding requirement
- Scope drift between documents: a feature described simply in the functional spec is designed at a very different complexity level in the architecture (or vice versa)
- The implementation plan covers all designed features — no designed features are silently excluded from the build plan
- UI design (if it exists) matches the functional spec — screens and flows align with described features

### Contradictory Statements
- The spec doesn't say X in one place and Y in another — constraints, limits, behaviors, and rules are consistent
- Numbers agree — if the functional spec says "maximum 10 items" and the architecture says "paginated at 25 per page," which is it?
- Behavioral descriptions match — if the functional spec says "users can delete their account" but the architecture says "accounts are deactivated, not deleted," the specs disagree
- Prioritization is consistent — if the functional spec marks a feature as required, it shouldn't be listed as optional or future work in the implementation plan

### Broken Internal References
- Sections that reference other sections or files point to things that actually exist
- Referenced formats, protocols, or standards are used consistently where they're cited
- Cross-references describe the target accurately — a reference that says "see Authentication in architecture.md" should point to a section that actually discusses authentication
- Removed or renamed sections haven't left dangling references elsewhere in the specs

### Data Flow Agreement
- Producer and consumer specs agree on data format — if the architecture says a component emits JSON events, the consuming component's spec should expect JSON events with compatible schema
- Data frequency assumptions match — a producer that sends hourly batches and a consumer that expects real-time streaming are incompatible
- Error handling at data boundaries agrees — the producer's error behavior and the consumer's error handling are compatible
- Shared data structures (request/response schemas, event payloads, configuration formats) are defined once and referenced consistently, not redefined differently in each spec

## Common Issues

- **Renamed concepts**: A concept was renamed partway through spec writing, and the old name lingers in some files. "Workspace" was renamed to "project" but the architecture still refers to "workspace" in three sections.
- **Feature scope drift**: The functional spec describes a simple version of a feature, but the architecture designs a much more complex version (or vice versa). The two specs describe different products.
- **Inconsistent constraint values**: The functional spec says passwords must be 8+ characters. The architecture says 12+. The UI design shows a "6 character minimum" label. Three specs, three different numbers.
- **Copy-paste divergence**: A description was copied between specs and then modified in one but not the other. The two copies now describe slightly different behavior.
- **Orphaned sections**: A feature was removed from the functional spec during revision, but the corresponding architecture section and implementation plan phases still exist. The specs describe something that's no longer wanted.
- **Implicit version skew**: The functional spec was updated recently but the architecture was written months ago. The functional spec references features that the architecture predates and doesn't cover.
- **Contradictory error behavior**: The functional spec says "show a friendly error and let the user retry." The architecture says "redirect to the home page on error." The UI design shows a modal dialog with a support link. Three different error experiences.

## Severity Guidance

**Critical:**
- Contradictory behavioral specifications — two specs describe incompatible behavior for the same operation, making it impossible to implement both correctly
- Features in the functional spec with no corresponding architecture design — implementers will have to invent the design, likely inconsistently
- Data flow disagreements between producer and consumer specs — components will fail to integrate at runtime
- Constraint contradictions on critical values (security thresholds, data limits, SLA targets)

**Moderate:**
- Terminology inconsistency that could cause implementer confusion — different names for the same concept across specs
- Feature scope drift between functional spec and architecture — the specs describe the same feature at different levels of complexity
- Orphaned spec sections describing removed features — won't cause bugs but will waste implementer time and cause confusion
- Broken references between spec documents — sections point to content that has moved or been renamed

**Mild:**
- Minor terminology variations that are unlikely to cause confusion (readers can figure it out from context)
- Formatting or structural inconsistencies between spec files (different heading levels, different section ordering)
- Cross-references that are correct but could be more specific (pointing to a whole document when a specific section would be clearer)
- Implicit version skew where the differences are cosmetic rather than behavioral
