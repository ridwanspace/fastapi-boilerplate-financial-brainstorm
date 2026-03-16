---
paths:
  - "src/**/infrastructure/**/*.py"
  - "src/infrastructure/**/*.py"
---

# Infrastructure Layer Rules

## SQLAlchemy Models

- Check `PRD.md` for the chosen monetary precision strategy:
  - If Decimal: `Mapped[Decimal]` with `Numeric(precision=19, scale=4)` — never `Float`
  - If integer cents: `Mapped[int]` with `BigInteger`
- Include `CheckConstraint` for amount and currency columns where applicable
- Register every new model in the database base module's `import_all_models()`

## Repositories / Data Access

- Check `PRD.md` for chosen concurrency strategy:
  - If optimistic locking: validate `version` before UPDATE
  - If pessimistic locking: use `SELECT FOR UPDATE`
- Check `PRD.md` for soft-delete decision:
  - If soft delete: all `SELECT` queries filter `WHERE deleted_at IS NULL`
- Check `PRD.md` for immutability decision:
  - If strict immutability: check immutable statuses before any UPDATE

## GCS Storage

- All GCS SDK calls wrapped with `asyncio.to_thread()` — SDK is synchronous
- GCS SDK and redis client have no type stubs — suppress mypy with targeted comments
