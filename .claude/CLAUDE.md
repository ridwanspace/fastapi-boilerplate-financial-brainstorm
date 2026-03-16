# FastAPI Financial — Claude Code Project Instructions

## Architecture

**Architecture is not predetermined.** Run `/brainstorm` to discuss and decide on the architecture
for this project. Decisions are recorded in `PRD.md` at the project root.

If `PRD.md` exists, follow the architecture, conventions, and folder structure defined there.
If `PRD.md` does not exist, the project is in the brainstorming phase — do not generate
architecture-specific code until the user runs `/brainstorm` and approves the PRD.

## Key Commands

```bash
make dev              # install all dependencies (including dev)
make run              # start the app (uvicorn, hot reload)
make test-unit        # pytest -m unit
make test-integration # pytest -m integration (requires TEST_DATABASE_URL)
make test-e2e         # pytest -m e2e
make lint             # ruff check + mypy
make format           # ruff format + ruff check --fix
make migrate          # alembic upgrade head
make migrate-create msg="..."  # alembic revision --autogenerate
```

All Makefile targets use `python -m <tool>` so they work correctly under conda environments.

## Slash Commands

| Command | Purpose |
|---------|---------|
| `/brainstorm` | Discuss architecture + financial design decisions with a senior architect. Produces `PRD.md`. |
| `/scaffold` | Generate project structure and boilerplate from `PRD.md` decisions. |
| `/plan <feature>` | Generate a structured development plan saved to `docs/plan/`. |
| `/run-plan <file> <task>` | Execute a specific task from a plan file. |
| `/setup` | First-time environment setup (deps, DB, migrations, smoke test). |
| `/commit` | Safe commit workflow (format → lint → typecheck → test → commit). |

## Workflow

1. **Brainstorm** → `/brainstorm` to decide architecture and financial patterns → produces `PRD.md`
2. **Scaffold** → `/scaffold` to generate folder structure and base code from PRD
3. **Plan** → `/plan <feature>` to create implementation plans
4. **Build** → `/run-plan <file> <task>` to execute plan tasks one at a time
5. **Commit** → `/commit` to safely commit changes

## Code Style

- Python 3.12+, async-first throughout
- Type hints required on all functions and methods
- Pydantic v2 for all request/response schemas; `model_config = ConfigDict(strict=True)` by default
- All code conventions specific to the chosen architecture are in `PRD.md`

## Testing

- Three markers: `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.e2e`
- Unit tests: pure logic, no I/O, no DB, no mocks of internals
- Integration tests: real DB via `db_session` fixture (per-test transaction rollback)
- Never mock the database in integration tests
- Run a single test: `pytest tests/unit/path/test_file.py -v`

## Financial Design

Financial-specific decisions (monetary precision, transaction model, concurrency strategy, etc.)
are made during `/brainstorm` and documented in `PRD.md`. The reference material for these
decisions lives in `.claude/financial-knowledge.md`.

**Until `/brainstorm` is run**, no financial patterns are assumed. After brainstorming,
the chosen patterns are enforced by the rule files and scaffold code.

## Environment

Copy `.env.example` → `.env` before first run. `JWT_SECRET_KEY` must be ≥ 32 chars.
Test env: `.env.test` (auto-loaded by `tests/conftest.py`).
Production: `APP_ENV=production` triggers startup validation.

**`ALLOWED_ORIGINS`** must be a JSON array string:
```
ALLOWED_ORIGINS=["http://localhost:3000","http://localhost:8080"]
```

## Detailed Rule Files

The following rule files contain specialized guidelines loaded based on the files you're working on:

| Rule File | When Loaded | Covers |
|-----------|-------------|--------|
| `.claude/rules/api.md` | Working on `src/**/api/**/*.py`, `src/api/**/*.py` | Request/response schemas, router conventions, rate limiting |
| `.claude/rules/infrastructure.md` | Working on `src/**/infrastructure/**/*.py`, `src/infrastructure/**/*.py` | SQLAlchemy models, repositories, database patterns |
| `.claude/rules/migrations.md` | Working on `alembic/**/*.py` | Migration workflow, column requirements |
| `.claude/rules/security.md` | **ALL code generation** | Credentials, JWT, CORS, SQL injection, logging |
| `.claude/rules/testing.md` | Working on `tests/**/*.py` | Test markers, naming, fixtures, coverage |
