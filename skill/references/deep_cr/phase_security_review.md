# Security Review

## What to Look For

### Authentication
- Authentication is required on all endpoints/routes that access user data or perform mutations
- Session/token handling is correct — tokens expire, sessions can be revoked, refresh token rotation works
- Password handling uses bcrypt, scrypt, or argon2 (not MD5, SHA-1, or SHA-256 without key stretching)
- Multi-factor authentication flows don't leak whether the first factor succeeded before the second factor is provided
- Login rate limiting is in place to prevent brute force attacks

### Authorization
- Authorization checks happen at the data layer, not just the route/controller layer
- Users can only access their own resources (or resources they have explicit permission for)
- Privilege escalation paths are closed — a regular user can't make themselves an admin by modifying request parameters
- IDOR (Insecure Direct Object Reference) — accessing `/users/123/data` when logged in as user 456 is blocked
- Role/permission checks use allowlists (check for "has permission"), not denylists (check for "is not blocked")

### Input Validation and Sanitization
- All user input is validated at system boundaries (API endpoints, form handlers, message consumers)
- Validation happens server-side, not just client-side
- SQL injection: parameterized queries or ORM — never string concatenation for SQL
- XSS: user input is escaped before rendering in HTML (use framework auto-escaping, not manual escaping)
- Command injection: user input is never passed to shell commands (`exec`, `system`, `subprocess.run` with `shell=True`)
- Path traversal: user input is never used directly in file paths without sanitization (`../../../etc/passwd`)
- Deserialization: untrusted data is never deserialized with unsafe methods (Python `pickle`, Java `ObjectInputStream`, Ruby `Marshal`)

### CSRF Protection
- State-changing requests require CSRF tokens (or use SameSite cookies + CORS properly)
- CSRF tokens are tied to the user session, not globally shared
- API endpoints that accept non-JSON content types are protected (JSON-only APIs with proper content-type checking are naturally CSRF-resistant)

### Secrets Management
- No hardcoded secrets, API keys, passwords, or tokens in code
- Secrets are loaded from environment variables or a secrets manager
- Secret values are not logged, even at debug level
- `.env` files are in `.gitignore`
- Default values for secret configs are not real credentials (no `password: "changeme"` in config files)

### Dependency Security
- No dependencies with known critical CVEs (check with `npm audit`, `pip audit`, `cargo audit`, etc.)
- Dependencies are fetched from official registries (not arbitrary URLs or git repos)
- Lock files are committed (prevents supply chain attacks via version drift)

### CORS and CSP Configuration
- CORS allows only the expected origins (not `*` on authenticated endpoints)
- Content Security Policy headers restrict script sources appropriately
- Sensitive cookies have `Secure`, `HttpOnly`, and `SameSite` attributes

### Sensitive Data Exposure
- Error responses don't include stack traces, SQL queries, or internal paths in production
- Logs don't contain passwords, tokens, PII, or full credit card numbers
- API responses don't over-return — endpoints don't send fields the client doesn't need (especially sensitive ones like password hashes, internal IDs, admin flags)
- Debug endpoints are disabled or access-controlled in production

### Cryptographic Choices
- Using well-known libraries for crypto (not hand-rolled implementations)
- AES-256 or ChaCha20 for symmetric encryption (not DES, 3DES, or RC4)
- RSA >= 2048 bits or Ed25519 for asymmetric operations
- TLS 1.2+ for network communication
- Random number generation uses cryptographically secure sources (`secrets` in Python, `crypto.randomBytes` in Node, not `Math.random`)

## Common Issues

- **Auth check in middleware only**: Authorization checked at the route/middleware level but the service layer doesn't verify permissions. Adding a new route that calls the same service bypasses auth.
- **Logging sensitive data**: `logger.info(f"Login attempt: {username}, {password}")` or logging full request bodies that contain tokens/credentials.
- **String concatenation in queries**: Building SQL or NoSQL queries by concatenating user input. The ORM is right there — use it.
- **Overly permissive CORS**: Setting `Access-Control-Allow-Origin: *` because "it works." Fine for public APIs, critical failure for authenticated endpoints.
- **JWT without validation**: Accepting JWTs without verifying the signature, checking expiration, or validating the issuer/audience claims.
- **Hardcoded secrets in test files**: Test files with real API keys or credentials. Tests should use mock values or test-specific secrets.
- **Unprotected file upload**: Accepting uploads without validating file type, size limits, or storing in a safe location. Path traversal + arbitrary file write = remote code execution.
- **Using `eval` or equivalent**: Evaluating user-controlled strings as code. This is always a critical vulnerability.

## Severity Guidance

**Critical:**
- SQL injection, XSS, CSRF, command injection, or path traversal vulnerabilities
- Missing authentication on data-modifying endpoints
- Missing authorization (any authenticated user can access any resource)
- Hardcoded secrets or credentials in code (not test fixtures with fake values)
- Use of `eval`, `exec`, or deserialization of untrusted input
- GPL or copyleft dependency added (licensing, not a runtime security issue, but immediate critical per project standards)

**Moderate:**
- CORS misconfiguration on authenticated endpoints
- Missing rate limiting on authentication endpoints
- Sensitive data in logs (PII, tokens)
- Dependencies with known moderate vulnerabilities
- Missing CSRF protection on state-changing forms
- Cookies without `Secure`/`HttpOnly`/`SameSite` attributes

**Mild:**
- Missing Content-Security-Policy headers (defense in depth, not primary protection)
- Using SHA-256 for password hashing (weak but not immediately exploitable)
- Verbose error messages that reveal framework versions
- Missing subresource integrity on CDN-loaded scripts
