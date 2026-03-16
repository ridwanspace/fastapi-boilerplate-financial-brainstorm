---
paths:
  - "tests/**/*.py"
---

# Testing Rules

## Test Markers

- `@pytest.mark.unit` — pure logic, no I/O, no DB, no external calls
- `@pytest.mark.integration` — real PostgreSQL via `db_session` fixture
- `@pytest.mark.e2e` — full HTTP stack via `async_client` fixture

## Unit Tests

- No mocks of internal logic
- No database, no file system, no network
- Test one behaviour per test function
- Use factory helper pattern — never repeat construction boilerplate
- `pytest.raises()` must always specify a **concrete exception class**, never bare `Exception`

## Integration Tests

- Use `db_session: AsyncSession` fixture from `tests/conftest.py`
- Each test runs inside a transaction that is rolled back on teardown
- **Never mock the database** — test against real PostgreSQL

## Test Naming

- `test_<what>_<expected_outcome>` — e.g., `test_settle_increments_version`
- Class per behaviour group: `TestPaymentCreation`, `TestPaymentSettlement`

## Coverage

- Minimum 80% overall
- Pure business logic: 100% target
- Run: `pytest --cov=src --cov-report=term-missing`
