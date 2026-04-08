# Data Model Review

## What to Look For

### Schema Design
- Tables represent clear domain concepts (not kitchen-sink tables with dozens of unrelated columns)
- Normalization is appropriate — duplicated data has a reason (performance, query simplicity) and that reason is documented
- Denormalization choices don't create update anomalies (data that can get out of sync)
- Junction/join tables are used for many-to-many relationships (not comma-separated IDs in a column)
- Polymorphic associations are handled cleanly (STI, separate tables, or discriminator column — not ambiguous foreign keys)

### Index Coverage
- Every foreign key column has an index
- Columns used in WHERE clauses, JOIN conditions, and ORDER BY have appropriate indexes
- Composite indexes match query patterns (column order matters — most selective first, or matching the query's column order)
- No redundant indexes (an index on `(a, b)` makes a standalone index on `(a)` redundant)
- Unique constraints are backed by unique indexes where business rules require uniqueness
- Consider partial indexes for queries that filter on a specific condition (e.g., `WHERE deleted_at IS NULL`)

### Migration Safety
- **Backwards compatibility**: Can the old application version run against the new schema? (Important for zero-downtime deploys)
- **Data loss risk**: Does the migration drop columns, tables, or modify data types in a way that loses information?
- **Rollback plan**: Can the migration be reversed? Down migrations should be tested.
- **Large table operations**: Adding columns, indexes, or constraints to large tables can lock the table. Use concurrent index creation, nullable columns first, or batched backfills.
- **Data backfill**: If new columns need values for existing rows, is there a backfill step? Does it handle large datasets without timeouts?

### Constraints
- NOT NULL on columns that should never be null (don't rely on application code alone)
- Foreign key constraints exist for relationships (prevents orphaned records)
- Unique constraints on business-unique fields (email, slug, external ID)
- Check constraints for value ranges or enum-like fields where the database supports them
- Default values are sensible (not just the database default of NULL)

### Naming Conventions
- Table names follow project convention (plural vs singular, snake_case vs PascalCase)
- Column names are descriptive and consistent (`created_at` not `ts`, `user_id` not `uid` — unless the project consistently uses abbreviations)
- Index names follow a pattern (`idx_tablename_column1_column2`)
- Foreign key columns are named `referenced_table_id` (e.g., `user_id`, not `owner` or `user`)
- Boolean columns read as predicates (`is_active`, `has_permission`, not `active_flag`)

### Soft Delete vs Hard Delete
- If soft delete is used: queries filter on the soft-delete column (deleted records don't leak into results)
- Unique constraints account for soft-deleted records (can you create a new record with the same unique field as a soft-deleted one?)
- Cascading behavior is correct — soft-deleting a parent handles children appropriately

### Temporal Data
- Timestamps include timezone or are stored in UTC with clear documentation
- `created_at` and `updated_at` are present on mutable tables
- `updated_at` is actually updated on changes (trigger or application code)
- Date vs datetime is used appropriately (a birthday is a date, an event time is a datetime)
- Time range queries work correctly with the chosen precision

### Enum and Status Fields
- Enum values are extensible — adding a new status doesn't require schema migration (string column vs database enum type)
- Status transitions are documented or enforced (not just an open string field that accepts anything)
- Default status values are correct for new records

## Common Issues

- **Missing foreign keys**: Columns named `user_id` or `order_id` with no foreign key constraint. The database can't enforce referential integrity, leading to orphaned records.
- **Over-normalization**: Breaking data into so many tables that every query requires 5+ joins. Sometimes a denormalized column or a JSON field is the right call.
- **Nullable everything**: Making every column nullable because "we might not always have this data." This pushes validation entirely to the application and makes NULL semantics ambiguous.
- **Text columns for structured data**: Storing JSON, CSV, or delimited data in a text column when a proper relation or JSON column type would be more appropriate.
- **Missing indexes on foreign keys**: Every ORM creates foreign key columns. Not every ORM creates indexes on them. Unindexed foreign keys cause slow joins and slow cascading deletes.
- **Enum as database type**: Using `CREATE TYPE ... AS ENUM`. Adding values requires a migration, and removing values is painful. String columns with application-level validation are usually more practical.
- **Implicit precision loss**: Changing a column from DECIMAL to FLOAT, or storing currency as a floating-point type. Money should be stored as integers (cents) or DECIMAL.
- **Migration without down**: Write-only migrations with no rollback path. When something goes wrong in production, the only option is a manual fix.

## Severity Guidance

**Critical:**
- Migrations that drop columns or tables with no backup/rollback plan
- Missing constraints that could cause data corruption (orphaned records, duplicate entries where uniqueness is required)
- Data type changes that lose precision (DECIMAL to FLOAT for monetary values)
- Large table migrations that will lock the table in production (add column with default, add index without CONCURRENTLY on Postgres)

**Moderate:**
- Missing indexes on foreign keys or commonly queried columns
- Nullable columns that should have NOT NULL constraints
- Naming inconsistencies that break project conventions
- Soft delete implementation that leaks deleted records into queries

**Mild:**
- Minor naming improvements (more descriptive column names)
- Missing `updated_at` triggers (if updates are infrequent)
- Index naming that doesn't follow the project pattern
- Documentation gaps on non-obvious denormalization choices
