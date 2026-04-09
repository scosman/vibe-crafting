# Data Model Design Review

## What to Look For

### Entity Completeness
- Every entity described in the functional spec is represented in the data model
- No phantom entities — the data model doesn't include entities that aren't mentioned anywhere in the functional spec (design without requirements)
- Entities have all the attributes needed to support the operations described in the spec (not just an entity name with two fields when the spec describes ten behaviors)

### Relationship Correctness
- Cardinality is explicitly stated and matches the functional spec — one-to-one, one-to-many, many-to-many are correctly modeled
- Ownership is clear — when a parent entity is deleted, what happens to its children (cascade delete, orphan, reassign)?
- Bidirectional relationships are intentional — if entity A references entity B and B references A, is there a reason, or is one direction sufficient?
- Self-referential relationships are designed carefully (tree structures, hierarchies, graph relationships)

### Query Pattern Support
- The data model supports the read patterns described in the spec — listing, filtering, sorting, searching, aggregating
- Queries that cross entity boundaries have a clear path (joins, denormalized fields, pre-computed views)
- High-frequency queries can be served efficiently — the model doesn't force expensive operations for common reads
- Reporting or analytics queries are considered if the spec describes dashboards, exports, or aggregate views

### Data Lifecycle
- How data is created is specified — defaults, required fields at creation, who/what creates it
- How data is updated is specified — which fields are mutable, what triggers updates, is update history preserved
- How data is archived or deleted is specified — soft delete vs hard delete, retention policies, cascading behavior
- Account/user deletion is addressed — what happens to all data associated with a user who leaves

### Migration and Evolution
- How the data model changes over time is considered — new fields, new entities, renamed concepts
- Migration strategy is defined for schema changes — how are they applied, is rollback possible
- Data volume growth is anticipated — what happens when a table grows from 1K rows to 10M rows
- Backward compatibility is considered — can old application versions coexist with the new schema during deployment

## Common Issues

- **Missing audit trails**: The spec describes actions with accountability ("who changed this setting and when") but the data model has no audit or history mechanism.
- **No soft-delete strategy when needed**: The spec implies data should be recoverable or visible in historical views, but the data model only supports hard deletion.
- **Denormalization without justification**: Data is duplicated across entities with no documented reason. When the source of truth is updated, the copies may not be — leading to inconsistency.
- **No consideration of data volume growth**: The model works fine with development-sized data but has design choices (polymorphic queries, full-table aggregations, deeply nested relationships) that won't scale.
- **Ignoring data classification**: The model stores sensitive data (PII, credentials, financial data) alongside non-sensitive data with no distinction in access control, encryption, or retention policy.
- **Enum fields with no evolution plan**: Status or type fields are defined with a fixed set of values but no consideration of what happens when a new value is needed. Is it a database enum (requires migration), a string (requires documentation), or a reference table (requires a lookup)?
- **Missing timestamps**: Entities that will need "when did this happen" queries but have no `created_at`, `updated_at`, or event timestamps.
- **Ambiguous null semantics**: A nullable field could mean "not set," "not applicable," "unknown," or "deliberately empty." The model doesn't distinguish between these, causing implementation confusion.

## Severity Guidance

**Critical:**
- Entities or relationships described in the functional spec are missing from the data model — the system can't store the data it needs to function
- Cardinality is wrong — a one-to-many relationship is modeled as one-to-one, preventing the system from representing valid real-world states
- No deletion or data lifecycle design for a system that handles user data — GDPR/regulatory compliance may be impossible
- Data model can't support a core query pattern described in the spec (e.g., "users can search by location" but no location data or indexing strategy)

**Moderate:**
- Audit/history mechanism is missing where the spec implies accountability or undo capability
- Denormalization exists without justification — creates risk of inconsistent data but doesn't block implementation
- Data volume growth is not considered — will cause performance problems but not immediately
- Missing timestamps on entities that will need temporal queries

**Mild:**
- Naming inconsistencies in the data model (entity or field names don't match spec terminology)
- Enum strategy is unspecified — workable but will cause minor friction when new values are needed
- Nullable fields with ambiguous semantics — implementable but confusing
- Minor missing attributes that are implicit (e.g., `created_at` on an entity where it's obvious but not stated)
