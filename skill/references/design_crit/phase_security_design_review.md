# Security Design Review

## What to Look For

### Trust Boundaries
- Trust boundaries are identified — where does trusted internal communication end and untrusted external input begin?
- Each boundary has defined validation — what checks happen when data crosses from untrusted to trusted zones?
- Service-to-service communication is inside or outside trust boundaries, and this is explicit (not assumed)
- Third-party integrations are treated as untrusted — data received from external services is validated, not blindly trusted

### Authentication Model
- How users (and services, if applicable) are identified is specified — token-based, session-based, certificate-based
- Token lifecycle is designed — issuance, expiration, refresh, revocation
- Session management is defined — what invalidates a session, how long sessions last, what happens on concurrent sessions
- Service-to-service authentication is designed if the architecture includes multiple services (API keys, mutual TLS, OAuth client credentials)
- Account recovery flows are specified — password reset, locked account, lost second factor

### Authorization Model
- Permission structure is defined — what permissions exist, how are they assigned, what do they grant
- Enforcement points are identified — where in the architecture are permissions checked (API gateway, service layer, data layer)
- Principle of least privilege is applied — users and services start with no access and are granted what they need, not start with full access and have some removed
- Resource-level authorization is designed — not just "can this user access this feature" but "can this user access this specific resource"
- Admin and elevated-privilege operations are identified and have additional authorization requirements

### Data Classification
- Sensitive data is identified — PII, credentials, financial data, health data, authentication tokens
- Protection at rest is designed — encryption for sensitive data in storage, key management
- Protection in transit is designed — TLS for all external communication, encryption for sensitive internal communication where appropriate
- Data minimization is considered — the system doesn't collect or store more data than it needs
- Retention and disposal is addressed — how long is sensitive data kept, how is it permanently removed

### Threat Scenarios
- Key threats are identified for the specific system being designed (not a generic list, but threats relevant to this architecture)
- Blast radius is considered — if a component is compromised, what does the attacker gain access to
- Input-based attacks are addressed — injection, XSS, CSRF, deserialization, depending on the technology stack
- Denial of service is considered — rate limiting, resource limits, queue depth limits
- Supply chain risks are acknowledged — dependencies, build pipeline, deployment infrastructure

## Common Issues

- **Auth at the edge only**: Authentication and authorization are checked at the API gateway or frontend, but backend services accept any request from inside the network. One compromised service = full access to everything.
- **Overly broad permissions**: The authorization model has two levels — "user" and "admin" — when the spec describes many different operations with different sensitivity levels. Users get more access than they need.
- **Sensitive data in logs or URLs**: The design logs request bodies (which contain passwords or tokens) or passes sensitive data as URL parameters (which appear in browser history, server logs, and proxy logs).
- **Missing rate limiting design**: The spec describes user-facing endpoints but no throttling or rate limiting is designed. Automated attacks can hammer the system unchecked.
- **No session invalidation on security events**: Password change, role change, or security breach doesn't invalidate existing sessions. An attacker who stole a session token retains access even after the password is changed.
- **Implicit trust between services**: Internal services communicate without authentication. The architecture assumes the network perimeter is the security boundary, which fails in cloud environments and when a single service is compromised.
- **Missing input validation at trust boundaries**: Data enters the system through an API but is only validated deep in the business logic layer. By then, it may have already been stored, logged, or forwarded to other services.
- **No encryption key management design**: The spec mentions encryption but doesn't address key storage, rotation, access control for keys, or what happens when a key is compromised.

## Severity Guidance

**Critical:**
- No authentication design for a system that handles user data or performs sensitive operations
- Authorization model is missing or has only binary access (full access vs no access) for a system with multiple user roles or resource ownership
- Sensitive data (credentials, PII, financial) is stored without encryption design or transmitted without transport security
- Trust boundaries are undefined — the system can't distinguish between trusted and untrusted input

**Moderate:**
- Authentication is designed but session/token lifecycle is incomplete (no expiration, no revocation, no refresh)
- Authorization exists but enforcement points are unclear — permissions are defined but where they're checked isn't specified
- Rate limiting or abuse prevention is not designed for user-facing endpoints
- Data classification is implicit — the spec handles some sensitive data carefully but doesn't systematically identify what's sensitive

**Mild:**
- Threat scenarios are generic rather than specific to this system's architecture
- Encryption key management is not detailed (keys are used but rotation/storage isn't designed)
- Minor gaps in defense-in-depth (single layer of protection that works but could be stronger)
- Audit logging for security events is not designed (when was access granted, who changed permissions)
