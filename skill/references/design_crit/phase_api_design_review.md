# API/Contract Design Review

## What to Look For

### Interface Completeness
- All operations needed by each consumer are present — trace each consumer's user flows through the API surface and check for gaps
- CRUD is not enough: does the API support domain-specific operations (e.g., "approve order," "transfer ownership," "archive project") or does it force consumers to implement multi-step workflows from primitive operations?
- Bulk/batch operations are designed where consumers will need to operate on multiple resources at once
- Query capabilities match the read patterns described in the functional spec — if users need to filter, sort, or search, the API supports it

### Naming Consistency
- Resource names follow the same convention across all APIs (plural nouns, consistent casing)
- Field names are consistent across related resources — the same concept uses the same name everywhere (not `user_id` in one response and `userId` in another, not `created_at` in one resource and `createdDate` in another)
- Action names follow a consistent pattern — if one endpoint uses `archive`, another doesn't use `deactivate` for the semantically same operation
- Terminology matches the functional spec — the API uses the same domain terms as the rest of the design

### Error Contract
- Error response format is defined and consistent across all endpoints or interfaces
- Error codes or types are enumerated — consumers can programmatically distinguish between different error conditions
- Validation errors identify which fields failed and why (not just "invalid request")
- Error responses include enough information for the client to take corrective action
- Error behavior is specified for edge cases — what happens on concurrent modification, resource not found, rate limit exceeded

### Versioning Strategy
- How breaking changes will be managed is defined — URL versioning, header versioning, or explicit no-breaking-changes policy
- What constitutes a breaking change is defined (removing a field, changing a type, adding a required parameter)
- If versioned: migration path from old version to new version is considered

### Input / Output Validation
- Input constraints are specified — required vs optional fields, allowed values, string length limits, numeric ranges
- Who enforces validation is clear — is it the API layer, the business logic layer, or both?
- Output format is specified — what fields are always present, what fields are conditional, how are null/missing values represented

### Idempotency
- Mutating operations that should be safe to retry are identified (creating a payment, submitting an order)
- The idempotency mechanism is designed — idempotency keys, natural idempotency (PUT), or deduplication logic
- What happens on a duplicate request is specified — does it return the original result, an error, or silently succeed?

## Common Issues

- **CRUD-only thinking**: The API mirrors database tables — create, read, update, delete — but the domain has richer operations. Consumers end up implementing domain logic by orchestrating multiple CRUD calls, which is fragile and inconsistent.
- **Inconsistent naming across services**: One service uses camelCase, another uses snake_case. One service calls it `status`, another calls the same concept `state`. This multiplies integration effort and causes bugs.
- **Missing pagination**: Collection endpoints return all results. Works in development with 10 records. Breaks in production with 100,000.
- **No error schema**: Success responses have a defined shape, but error responses are ad-hoc strings or vary by endpoint. Consumers can't build reliable error handling.
- **Chatty interfaces**: Completing a single user action requires 5+ API calls in sequence. This is slow, fragile (any call can fail), and hard for consumers to implement correctly.
- **Missing webhooks or callbacks for async operations**: The API starts a long-running operation but provides no way for the consumer to know when it completes — forcing consumers to poll.
- **Implicit ordering dependencies**: Operations must be called in a specific sequence, but this isn't documented or enforced. Calling them out of order produces confusing errors or corrupt state.
- **Leaking internal model**: The API response structure mirrors the internal database schema instead of the domain model consumers need. Internal refactoring breaks the API contract.

## Severity Guidance

**Critical:**
- Operations required by the functional spec are missing from the API design — a consumer flow literally can't be implemented
- No error contract — consumers have no reliable way to handle failures, leading to broken user experiences
- Missing pagination on collections that will grow — this will cause outages at scale
- Breaking changes to existing contracts with no versioning strategy

**Moderate:**
- Inconsistent naming across APIs — increases integration effort and causes bugs but doesn't block implementation
- Missing idempotency on operations where retry safety matters (payment, order submission)
- CRUD-only design that forces consumers to orchestrate complex multi-call workflows
- Validation constraints are defined in prose but not reflected in the API specification

**Mild:**
- Minor naming inconsistencies within a single API (cosmetic, not confusing)
- Missing batch operations where individual calls work but are slower
- Versioning strategy is informal or implicit but the team is small and coordinated
- Webhook/callback design is missing for operations that are fast enough to poll
