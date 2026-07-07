# Testing Patterns Reference

## Test Organization

Structure tests to mirror the source directory. Co-locate test files with source files or maintain a parallel `__tests__` directory. Name test files `{module}.test.{ext}`.

## Unit Test Patterns

| Pattern | When to Use | Example |
|---------|-------------|---------|
| **Arrange-Act-Assert** | Default for all unit tests | Set up state → call function → verify result |
| **Table-driven tests** | Multiple input/output combinations | `test.each([[input, expected], ...])` |
| **Mock boundaries** | External services, databases, APIs | Mock at the boundary, test logic in isolation |
| **Error path testing** | Every function that can throw | Test both success and failure paths |
| **Snapshot testing** | Serializable output (JSON, HTML) | Use sparingly; prefer explicit assertions |

## Integration Test Patterns

| Pattern | When to Use |
|---------|-------------|
| **API contract testing** | Verify request/response shapes match spec |
| **Auth flow testing** | Test authenticated and unauthenticated paths |
| **Database integration** | Use test database with migrations, seed, and teardown |
| **Service mocking** | Mock external APIs at the HTTP level (MSW, nock) |

## E2E Test Patterns

| Pattern | When to Use |
|---------|-------------|
| **Auth fixture** | Authenticate once, reuse session state across tests |
| **Page object model** | Abstract page interactions into reusable classes |
| **Visual regression** | Screenshot comparison for UI consistency |
| **Console error filtering** | Ignore known benign errors (favicon 404, HMR, DevTools) |
| **Retry strategy** | Flaky tests get max 2 retries before investigation |

## Visual Regression

Set a pixel diff threshold (20–30%) to account for animation timing and font rendering differences. Store baselines in git. Generate diffs in CI but keep them in `.gitignore`.

## Mocking Strategy

| Layer | Mock Tool | Rule |
|-------|-----------|------|
| HTTP | MSW, nock | Mock at network level, not at import level |
| Database | Test DB or in-memory | Use real queries against test schema |
| Time | `vi.useFakeTimers()` | For time-dependent logic |
| Randomness | Seeded generators | For reproducible tests |
| File system | `memfs` or temp dirs | Clean up after each test |

## Coverage Targets

| Metric | Minimum | Ideal |
|--------|---------|-------|
| Line coverage | 70% | 85%+ |
| Branch coverage | 60% | 80%+ |
| Critical path coverage | 95% | 100% |
| Security test coverage | 90% | 100% |

Coverage is a guide, not a goal. 100% coverage with weak assertions is worse than 70% coverage with strong assertions.
