# REST API Review

## What to Look For

### Resource Naming and URL Design
- Resources are nouns, not verbs (`/users`, not `/getUsers`)
- Plural nouns for collections (`/users`, not `/user`)
- Nested resources reflect ownership (`/users/{id}/orders`, not `/orders?user_id={id}` — unless orders are a top-level resource)
- URL paths are kebab-case or snake_case consistently (not mixed)
- No unnecessary nesting depth (max 2-3 levels)

### HTTP Methods and Status Codes
- GET for reads, POST for creates, PUT/PATCH for updates, DELETE for deletes
- PUT for full replacement, PATCH for partial updates — not interchangeable
- 201 for successful creation (not 200)
- 204 for successful deletion with no body (not 200 with empty body)
- 404 for missing resources (not 200 with null)
- 409 for conflicts (duplicate creation, concurrent edit)
- 422 for validation errors (not 400 for everything)

### Request/Response Schema Design
- Consistent field naming across all endpoints (camelCase or snake_case, not mixed)
- Consistent envelope pattern (or lack of one) — don't wrap some responses but not others
- Response includes the resource ID (not just on creation)
- Timestamps use ISO 8601 format with timezone
- Nested objects vs flat fields are used consistently
- Nullable fields are explicitly documented or typed, not silently omitted

### Pagination, Filtering, Sorting
- Collection endpoints support pagination (not unbounded result sets)
- Pagination uses cursor-based or offset/limit consistently
- Response includes total count or next-page indicator
- Filter parameters have clear naming (`status=active`, not `s=a`)
- Sort parameter syntax is consistent across endpoints

### Versioning
- API version strategy is clear (URL path, header, or query param — pick one)
- Breaking changes don't land in unversioned endpoints
- If versioned: old version behavior is preserved

### Error Response Format
- Errors follow a consistent structure across all endpoints
- Error responses include: error code/type, human-readable message, field-level details for validation errors
- Error messages don't leak internal details (stack traces, SQL errors, internal paths)
- Validation errors identify which fields failed and why

### Authentication and Authorization
- All endpoints that should require auth actually check it
- Authorization is checked at the resource level (not just "is logged in" but "can this user access this resource")
- Auth failures return 401 (not authenticated) vs 403 (not authorized) correctly
- Sensitive operations require appropriate auth levels

### Idempotency
- PUT and DELETE are idempotent (repeated calls produce the same result)
- POST endpoints for critical operations support idempotency keys
- Retry-safe: clients can safely retry failed requests without side effects

### Rate Limiting and Input Constraints
- Rate limiting is applied to public/high-traffic endpoints
- Request body size limits are enforced
- Array/list parameters have maximum length limits
- String fields have maximum length constraints

## Common Issues

- **Inconsistent error formats**: Some endpoints return `{"error": "msg"}`, others return `{"message": "msg", "code": 123}`. Pick one format and use it everywhere.
- **Leaking internal state**: Returning database IDs, internal field names, or implementation details that couple clients to backend structure.
- **Missing pagination**: Returning all results in a collection. Works with 10 records, breaks with 10,000.
- **Verb-based URLs**: `/api/getUserById` instead of `/api/users/{id}`. This isn't a REST API — it's RPC with extra steps.
- **Ignoring HTTP semantics**: Using POST for everything, returning 200 for errors, using GET with request bodies.
- **Overfetching by default**: Returning the entire object graph when clients only need a few fields. No field selection or sparse fieldsets.
- **Inconsistent null handling**: Sometimes omitting null fields, sometimes including them. Clients can't tell if a field is missing or null.
- **Auth check in controller only**: Authorization checked at the route level but not enforced in the service layer. Easy to bypass when adding new routes.

## Severity Guidance

**Critical:**
- Missing authentication or authorization on endpoints that modify data
- Endpoints that return unbounded result sets with no pagination (data growth = outage)
- Error responses that leak sensitive internal information (SQL errors, stack traces, internal paths)
- Breaking changes to existing API contracts without versioning

**Moderate:**
- Inconsistent error response formats across endpoints
- Wrong HTTP status codes (200 for errors, 404 for validation failures)
- Missing idempotency on critical mutating operations (payment, order creation)
- Inconsistent naming conventions across endpoints

**Mild:**
- Minor naming inconsistencies (camelCase in one field, snake_case in another within the same response)
- Missing pagination on small, bounded collections
- Slightly verbose response structures that could be simplified
- Missing HATEOAS links (only relevant if the project uses hypermedia)
