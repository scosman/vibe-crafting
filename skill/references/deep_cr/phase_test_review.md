# Test Quality Review

## What to Look For

### Coverage of New/Changed Code
- New code paths have corresponding tests
- Changed behavior has updated tests that verify the new behavior (not just the old behavior with the test adjusted to pass)
- Edge cases identified in the implementation are tested: empty inputs, boundary values, error conditions
- If a bug was fixed, there's a regression test that would have caught the original bug

### Test Quality — Behavior vs Exercise
- Tests verify *behavior* (given this input, this outcome), not just *exercise code* (call the function, it doesn't crash)
- Assertions are specific — `assertEqual(result, expected_value)`, not just `assertNotNone(result)`
- Tests check for the absence of side effects where relevant (function didn't modify input, didn't write to database when it shouldn't have)
- Negative tests exist — verifying that invalid inputs are rejected, unauthorized access is denied

### Test Brittleness
- Tests don't break when internal implementation changes without behavior change (testing public interface, not private methods)
- Tests don't depend on specific ordering, timing, or system state
- Tests don't assert on exact error messages or log output unless that's the documented contract
- Tests don't rely on real external services (network calls, third-party APIs) — these should be mocked or use test doubles
- Snapshot tests are used sparingly and for appropriate things (serialization formats, not UI that changes frequently)

### Missing Edge Cases
- Null/None/undefined inputs where the code doesn't explicitly require non-null
- Empty collections (empty list, empty string, empty dict)
- Boundary values (0, -1, MAX_INT, empty string vs null)
- Concurrent access if the code is used in a concurrent context
- Error conditions: network failures, database errors, timeout scenarios
- Unicode and special characters in string inputs

### Test Isolation
- Each test can run independently — no dependency on test execution order
- Tests don't share mutable state (global variables, class-level state, shared database rows)
- Database tests use transactions that roll back, or each test sets up its own data
- File system tests use temporary directories that are cleaned up
- Tests don't modify environment variables without restoring them

### Mock Quality
- Mocks are placed at the correct boundary (mock the external dependency, not internal implementation details)
- Mocks verify interactions that matter (was the email service called?) but don't over-specify (don't assert on exact arguments unless they're the point of the test)
- Mock return values are realistic — they match the shape and types of real responses
- Over-mocking: if a test mocks 5 things to test 1 thing, the test is likely testing the mocking framework, not the code
- Mock assertions use `assert_called_with` or equivalent — not just `assert_called` (verifies correct arguments, not just that something was called)

### Test Naming and Readability
- Test names describe the scenario and expected outcome (`test_login_with_expired_token_returns_401`, not `test_login_3`)
- Test structure follows arrange-act-assert (or given-when-then) clearly
- Test setup is visible in the test — not hidden in deep fixture hierarchies
- Helper methods in tests are named clearly and don't obscure what's being tested

### Integration vs Unit Test Balance
- Unit tests cover individual functions/methods with mocked dependencies
- Integration tests verify that components work together correctly
- No false confidence from integration tests that only test the happy path
- Expensive tests (database, network, browser) are clearly marked and can be run separately

## Common Issues

- **Testing the mock**: Test mocks the database, then asserts the function returns the mocked value. This tests nothing — it verifies that the mock works.
- **Assert-free tests**: Test calls the function but has no assertions. It only catches exceptions, not wrong behavior.
- **Shared test state**: Test A writes to a database/global. Test B reads from it. Tests pass together, fail individually or in different order.
- **Copy-paste test inflation**: 20 near-identical tests that differ by one parameter. Should be a parameterized test.
- **Testing private implementation**: Tests import internal/private functions and test them directly. Refactoring the internals breaks all the tests even though behavior didn't change.
- **Missing error path tests**: Happy path is thoroughly tested. Error handling is untested — but error handling is where most bugs live.
- **Flaky time-dependent tests**: Tests that use `time.now()` or `Date.now()` and fail near midnight, on slow CI, or across timezones. Time should be injected or frozen.
- **Fixture sprawl**: Test fixtures define hundreds of fields. Most tests need 3 of them. Creates maintenance burden and obscures what each test actually needs.

## Severity Guidance

**Critical:**
- New code that handles security, payments, or data integrity has no tests
- Tests that pass but don't actually verify the behavior they claim to test (assert-free, testing the mock)
- Shared mutable state between tests that makes results unreliable — you can't trust the test suite
- Tests that call real external services (flaky, slow, and potentially dangerous in CI)

**Moderate:**
- Missing edge case tests for error handling paths
- Brittle tests that will break on legitimate refactors
- Over-mocking that makes tests meaningless
- Missing integration tests for critical workflows (only unit tests for code that's mainly about integration)

**Mild:**
- Test naming that's unclear but functional
- Minor fixture cleanup improvements
- Tests that could be parameterized for better coverage
- Missing tests for straightforward getters/setters or simple delegating functions
