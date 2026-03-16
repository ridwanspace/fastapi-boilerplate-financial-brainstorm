# FastAPI Financial

[![Python 3.12+](https://img.shields.io/badge/python-3.12%2B-blue?logo=python&logoColor=white)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115%2B-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Pydantic v2](https://img.shields.io/badge/Pydantic-v2-E92063?logo=pydantic&logoColor=white)](https://docs.pydantic.dev)
[![SQLAlchemy 2.0](https://img.shields.io/badge/SQLAlchemy-2.0%2B-D71F00?logo=sqlalchemy&logoColor=white)](https://www.sqlalchemy.org)
[![PostgreSQL 16](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org)
[![Redis 7](https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white)](https://redis.io)
[![Ruff](https://img.shields.io/badge/linter-Ruff-261230?logo=ruff&logoColor=white)](https://docs.astral.sh/ruff/)
[![mypy](https://img.shields.io/badge/type--checked-mypy--strict-blue)](https://mypy-lang.org)
[![pytest](https://img.shields.io/badge/tests-pytest-0A9EDC?logo=pytest&logoColor=white)](https://pytest.org)
[![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com)
[![Claude Code](https://img.shields.io/badge/Claude_Code-powered-D97757?logo=anthropic&logoColor=white)](https://claude.com/claude-code)
[![License](https://img.shields.io/badge/license-Proprietary-gray)]()

Production-grade FastAPI boilerplate for financial applications — with architecture decided through guided brainstorming, not hardcoded.

## What Makes This Different

Most boilerplates ship with a fixed architecture (MVC, DDD, hexagonal, etc.). This one doesn't. Instead, it ships with a **brainstorming workflow** powered by Claude Code that helps you choose the right architecture and financial patterns for your specific use case — then generates the code to match.

## Prerequisites

- Python 3.12+
- Docker & Docker Compose
- [Claude Code](https://claude.com/claude-code) CLI

## Quick Start

**1. Create a new repo from this template**

Click **"Use this template"** on GitHub, or via CLI:

```bash
gh repo create my-financial-app --template ridwanspace/fastapi-boilerplate-financial-brainstorm --clone
cd my-financial-app
```

**2. Open in Claude Code and run the workflow**

```bash
# First-time setup (deps, DB, migrations, smoke test)
/setup

# Brainstorm architecture with a senior architect
/brainstorm

# Generate project structure from your decisions
/scaffold

# Plan your first feature
/plan <feature description>

# Build it
/run-plan <plan-file> <task-id>
```

## Slash Commands

| Command | Purpose |
|---------|---------|
| `/brainstorm` | Discuss architecture + financial design with a senior software architect. Walks through trade-offs, proposes options with caveats, and produces `PRD.md` when you're ready. |
| `/scaffold` | Reads `PRD.md` and generates the full project structure — base classes, models, routers, test fixtures — matching your chosen architecture and financial patterns. |
| `/plan <feature>` | Generate a structured, implementation-ready development plan saved to `docs/plan/`. |
| `/run-plan <file> <task>` | Execute a specific task from a plan file, then update progress. |
| `/setup` | First-time environment setup — install deps, start Docker, create DB, run migrations, smoke test. |
| `/commit` | Safe commit workflow — format, lint, typecheck, unit test, review diff, then commit. |

## How Brainstorming Works

`/brainstorm` starts a multi-turn conversation where a senior software architect:

1. **Asks about your context** — what the app does, scale, compliance needs, team size
2. **Proposes architecture options** — comparing 2–3 relevant choices with sample folder structures, trade-offs, and a recommendation
3. **Walks through financial design decisions** — 10 areas from monetary precision to error handling, each with concrete options and caveats
4. **Generates `PRD.md`** — when you approve, documenting every decision for `/scaffold` to consume

### Financial Decision Areas

| # | Area | Example Options |
|---|------|----------------|
| 1 | Monetary Precision | Decimal + banker's rounding, integer cents, multi-precision |
| 2 | Transaction Immutability | Strict immutability, append-only ledger, soft state with TTL |
| 3 | Concurrency Control | Optimistic locking, pessimistic locking, event sourcing |
| 4 | Idempotency | Idempotency-Key header, natural key dedup, at-least-once |
| 5 | Audit Trail | Column-level, audit log table, event sourcing as audit |
| 6 | Soft Delete | Soft delete, hard delete + archive, retention-policy delete |
| 7 | Multi-Currency | Single currency, per-record currency, base currency + conversion |
| 8 | Balance Computation | Stored balance, computed balance, hybrid |
| 9 | API Serialization | String amounts, integer minor units, object with currency |
| 10 | Error Handling | Domain exceptions, Result type, Problem Details (RFC 9457) |

## What's Included (Architecture-Agnostic)

These ship with the boilerplate regardless of architecture choice:

```
├── .claude/                  # Claude Code configuration
│   ├── CLAUDE.md             # Project instructions
│   ├── financial-knowledge.md # Financial decision reference
│   ├── commands/             # All slash commands
│   ├── rules/                # Code generation rules
│   └── settings.local.json   # Permissions
├── src/
│   ├── main.py               # Minimal FastAPI app (health endpoint)
│   └── settings.py           # Pydantic settings
├── alembic/                  # Migration setup
├── docker/                   # Dockerfile + docker-compose (Postgres + Redis)
├── tests/                    # Test scaffolding
├── docs/plan/                # Development plans
├── Makefile                  # dev, run, test, lint, format, migrate
├── pyproject.toml            # Ruff, mypy, pytest, coverage config
├── requirements.txt          # Production deps
└── requirements-dev.txt      # Dev/test deps
```

## What Gets Generated (Architecture-Specific)

After `/brainstorm` → `/scaffold`, you get code tailored to your decisions:

- **Folder structure** matching the chosen architecture (layered, modular monolith, DDD, etc.)
- **Base classes** for entities/models with the chosen financial patterns (version columns, soft delete, audit fields, etc.)
- **Money utilities** matching your precision strategy (Decimal, integer cents, multi-precision)
- **Repository patterns** with your concurrency strategy (optimistic locking, pessimistic locking)
- **API conventions** with your serialization choice (string amounts, integer cents, amount objects)
- **Middleware** (correlation ID, error handling, request logging)
- **Test fixtures** (async client, DB session, factories)
- **Updated `.claude/rules/`** tuned to the chosen architecture

## Development

```bash
make dev              # Install all dependencies
make run              # Start dev server (hot reload)
make test-unit        # Run unit tests
make test-integration # Run integration tests (requires DB)
make test-e2e         # Run e2e tests
make lint             # Ruff + mypy
make format           # Ruff format + autofix
make migrate          # Apply pending migrations
make docker-up        # Start Postgres + Redis
make docker-down      # Stop Docker services
```

## Environment

Copy `.env.example` to `.env` before first run. Key variables:

| Variable | Required | Notes |
|----------|----------|-------|
| `DATABASE_URL` | Yes | `postgresql+asyncpg://postgres:postgres@localhost:5432/financial_db` |
| `REDIS_URL` | Yes | `redis://localhost:6379/0` |
| `JWT_SECRET_KEY` | Yes | Minimum 32 characters. Generate: `openssl rand -hex 32` |
| `GCS_PROJECT_ID` | Production | Google Cloud Storage project |
| `GCS_BUCKET_NAME` | Production | GCS bucket name |
| `SENTRY_DSN` | Optional | Error tracking |

`ALLOWED_ORIGINS` must be a JSON array string:
```
ALLOWED_ORIGINS=["http://localhost:3000","http://localhost:8080"]
```

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Framework | FastAPI |
| Database | PostgreSQL 16 (async via asyncpg) |
| ORM | SQLAlchemy 2.0 (async) |
| Migrations | Alembic |
| Validation | Pydantic v2 |
| Cache | Redis 7 |
| Auth | JWT (PyJWT) |
| Storage | Google Cloud Storage |
| Rate Limiting | SlowAPI |
| Error Tracking | Sentry |
| Linting | Ruff |
| Type Checking | mypy (strict mode) |
| Testing | pytest (unit / integration / e2e) |
| Containerization | Docker + Docker Compose |
