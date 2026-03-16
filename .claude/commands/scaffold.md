---
description: Generate project structure and boilerplate code from PRD.md decisions. Usage: /scaffold
argument-hint: ""
allowed-tools: Read, Glob, Grep, Bash, Write, Edit
---

You are a senior engineer generating the project scaffolding based on the architecture and financial decisions documented in `PRD.md`.

---

## Step 1 — Load and validate PRD

Read `PRD.md` from the project root.

If the file does not exist, stop and tell the user:
```
PRD.md not found. Run /brainstorm first to make architecture and financial design decisions.
```

If `PRD.md` exists, extract:
1. **Architecture choice** (Section 2)
2. **Financial design decisions** (Section 3 — all chosen options)
3. **Project structure** (Section 4 — the directory tree)
4. **Code conventions** (Section 5)
5. **Files to create** (Section 6)
6. **Files NOT needed** (Section 6)

---

## Step 2 — Load financial knowledge base

Read `.claude/financial-knowledge.md` for reference on how each financial decision translates to code patterns.

---

## Step 3 — Pre-flight check

Before generating anything, check what already exists:

```bash
find src/ -type f -name "*.py" 2>/dev/null | head -50
```

- If `src/` contains existing application code beyond placeholder files, **warn the user** that scaffolding will create new files but will NOT delete existing ones. They should review and clean up manually.
- If `src/` is empty or only has `__init__.py` files, proceed without warning.

---

## Step 4 — Clean up architecture-specific remnants

If any of these exist and are NOT in the PRD's project structure, remove them:
- `src/contexts/` (DDD bounded contexts)
- `src/shared/domain/` (DDD shared kernel)
- `src/shared/application/` (DDD shared application)

**Do NOT remove**:
- `src/main.py`, `src/settings.py`, `src/container.py` (adapt in place)
- `src/infrastructure/` (adapt, don't remove)
- `src/api/` (adapt, don't remove)
- Anything in `.claude/`, `docker/`, `scripts/`, `alembic/`, `tests/`

---

## Step 5 — Generate the project structure

Create all directories and files listed in the PRD's "Files to create" section (Section 6).

### For each file, follow these rules:

**Python source files** (`src/**/*.py`):
- Include proper imports based on the chosen architecture
- Add type hints on all functions
- Use `async` for all I/O-bound functions
- Follow the code conventions from PRD Section 5
- Add `__init__.py` for every package directory

**Base classes / shared utilities**:
- Generate based on financial decisions (e.g., if Option A for monetary precision → create a `Money` value object using `Decimal`)
- If the architecture uses a Result type → create `Ok`/`Err` classes
- If optimistic locking is chosen → include `version` field in base entity

**Infrastructure files**:
- `src/infrastructure/database/engine.py` — async engine + session factory
- `src/infrastructure/database/base.py` — SQLAlchemy declarative base + `import_all_models()`
- `src/infrastructure/database/unit_of_work.py` — if the architecture uses UoW pattern
- Adapt based on architecture (e.g., no UoW for simple layered)

**API files**:
- `src/api/router.py` — main router aggregating sub-routers
- `src/api/middleware/` — correlation ID, error handler, request logging
- `src/api/schemas/` — shared schemas (health, error responses)

**Test scaffolding**:
- `tests/conftest.py` — base fixtures (async client, db session if needed)
- `tests/unit/` — directory structure mirroring src
- `tests/integration/` — directory structure mirroring src
- `tests/e2e/` — placeholder

### Financial-specific code generation

Based on the financial decisions in the PRD:

| Decision | Code to Generate |
|----------|-----------------|
| Decimal + ROUND_HALF_EVEN | `Money` value object with safe arithmetic |
| Integer cents | Amount utility with conversion helpers |
| Strict immutability | Immutable status guard in base entity/model |
| Append-only ledger | Ledger entry base model, no UPDATE methods |
| Optimistic locking | `version` column in base model, check in repository |
| Pessimistic locking | `SELECT FOR UPDATE` helper in base repository |
| Idempotency-Key header | Idempotency middleware or decorator |
| Column-level audit | `created_by_id`, `updated_by_id` mixin |
| Audit log table | `AuditLog` model + logging decorator |
| Soft delete | `SoftDeleteMixin` with `deleted_at` + query filter |
| String amounts | Serialization helpers for API schemas |
| Result type | `Ok`/`Err` classes with `is_ok()`/`is_err()` |
| Domain exceptions | Base exception hierarchy |

---

## Step 6 — Update .claude/CLAUDE.md

Read the current `.claude/CLAUDE.md` and update it to reflect the chosen architecture:

- Replace any architecture-specific language with the actual chosen architecture
- Update the "Key Commands" section if needed
- Update or create a "Code Style" section matching PRD conventions
- Update the "Financial Safety" section based on chosen financial options
- Update the "Adding a New Module/Feature" section to match the chosen structure
- Keep the "Environment" and "Detailed Rule Files" sections intact

---

## Step 7 — Update .claude/rules/

Update rule files to match the chosen architecture. Rules should be **specific** to the chosen patterns, not generic.

If the chosen architecture doesn't have a concept (e.g., no "bounded contexts" in simple layered), remove references to it from all rule files.

---

## Step 8 — Update alembic/env.py

If the base model location changed (e.g., from `src/infrastructure/database/base.py` to somewhere else), update the import in `alembic/env.py`.

---

## Step 9 — Verification

Run a quick check to make sure the generated code is valid:

```bash
# Check that all __init__.py files exist
find src/ -type d -exec sh -c 'test -f "$1/__init__.py" || echo "Missing: $1/__init__.py"' _ {} \;
```

```bash
# Quick syntax check on generated Python files
python -m py_compile src/main.py 2>&1 || echo "Syntax errors found"
```

---

## Step 10 — Summary report

Print a summary of what was generated:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SCAFFOLD COMPLETE
Architecture: {chosen architecture}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Files created:
  {list with counts by category}

Files modified:
  {list}

Files removed:
  {list, or "none"}

Financial patterns included:
  {list of generated financial code based on decisions}

Next steps:
  1. Run /setup to initialize the environment
  2. Run /plan <your first feature> to start building
  3. Review generated base classes in {path}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
