# Completeness & Coverage Review

## What to Look For

### Requirements Traceability
- Every functional requirement described in the functional spec has a corresponding design in the architecture or component specs
- Every user-facing feature has a defined implementation path — no features are listed in the functional spec and then silently dropped in the architecture
- If an implementation plan exists, every phase maps back to functional requirements (no orphaned phases that build things nobody asked for)

### Edge Cases and Error Handling
- Error scenarios are specified, not just the happy path — what happens when a request fails, a service is unavailable, input is invalid, data is missing
- Empty states are defined — what does the UI show with zero data, what does the API return for empty collections
- Boundary conditions are addressed — maximum sizes, minimum values, character limits, time boundaries
- Partial failure behavior is specified — what happens when step 3 of 5 fails, is there rollback or partial state

### User Flows
- All paths through the system are specified, not just the primary happy path
- Alternate flows are defined — what if the user cancels, goes back, refreshes, opens in a new tab
- First-time user experience is distinct from returning user experience where relevant
- Admin/operator flows are specified if the system has admin functionality

### Missing Scenarios
- Timeout behavior: what happens when an operation takes too long — is there a deadline, does the user see a spinner forever, is there a retry
- Concurrent access: what happens when two users edit the same resource, two requests arrive simultaneously, a background job runs while a user is active
- Invalid input: what constitutes invalid input for each operation, who validates it, what error does the user see
- Recovery: after a failure, what state is the system in, can the user retry, is cleanup needed
- Offline/degraded: if any network dependency is unavailable, what happens — graceful degradation or hard failure

### Out-of-Scope Clarity
- The spec explicitly states what the system does NOT do (avoids scope creep during implementation)
- Features mentioned as future work are clearly labeled as out of scope for the current design
- Integration points that are deferred are identified (e.g., "V1 uses local storage; V2 will add cloud sync")

## Common Issues

- **Happy-path-only specs**: The functional spec describes what happens when everything works. No mention of errors, edge cases, or failure recovery. Implementers are left guessing how to handle the other 80% of real-world scenarios.
- **Undefined error recovery**: The spec says "display an error" but doesn't specify what the error says, whether the user can retry, or what state the system is in after the error.
- **Missing cleanup and teardown**: Resources are created but there's no design for how they're cleaned up — temporary files, expired sessions, orphaned records, abandoned uploads.
- **Assumed-but-not-stated prerequisites**: The design assumes the user has already completed step X before reaching step Y, but this isn't enforced or documented. What happens if they haven't?
- **Unspecified error behavior**: An operation is designed with its success path but no error response format, error codes, or guidance on what happens when it fails — callers are left guessing.
- **Missing data lifecycle**: The spec describes how data is created and read but not how it's updated, archived, or deleted. What happens to user data when they close their account?

## Severity Guidance

**Critical:**
- A functional requirement has no corresponding design — implementers will have to invent the design during coding, leading to inconsistency or mid-implementation redesign
- No error handling design for operations that will definitely fail in production (network calls, user input, external dependencies)
- Missing concurrent access design for data that will be accessed by multiple users or processes simultaneously

**Moderate:**
- Edge cases are unaddressed but unlikely to cause implementation failure (rare user flows, unusual input combinations)
- Empty states are undefined — implementers will handle them but inconsistently
- Cleanup/teardown is missing for resources that accumulate slowly (won't fail immediately but will cause operational issues)

**Mild:**
- Out-of-scope boundaries are implicit rather than explicit (readers can infer, but stating it prevents scope questions)
- Minor user flows are unspecified (e.g., what happens when the user double-clicks a button)
- Future work is mentioned without being clearly deferred
