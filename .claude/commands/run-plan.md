---
description: Execute a specific task from a plan file, then update plan progress. Usage: /run-plan <plan-file> <task-id>
argument-hint: "<plan-file-name-or-number> <task-id>"
allowed-tools: Read, Glob, Grep, Bash, Edit, Write
---

You are a senior engineer executing a specific task from a development plan.

**Arguments received**: $ARGUMENTS

---

## Step 1 — Parse arguments

`$ARGUMENTS` contains two tokens: `<plan-file> <task-id>`

Examples:
```
/run-plan 00-wallet-refund.md B1.1
/run-plan 03 Task-2.3
/run-plan docs/plan/02-billing.md 2.3
```

Parsing rules:
- **First token**: may be a full path, filename, or number prefix (e.g., `03` → glob `docs/plan/03-*.md`).
- **Second token**: task section to execute. Match case-insensitively against section headers.

If `$ARGUMENTS` is missing or malformed, stop and show usage.

---

## Step 2 — Load project context

Read `PRD.md` (if it exists) to understand:
- Architecture and folder structure conventions
- Financial design decisions
- Code conventions

Read `.claude/CLAUDE.md` for project-level standards.

Then read the full plan file. Extract:
1. Plan metadata (title, module, depends-on, status)
2. The target task section
3. "What Already Exists" table

---

## Step 3 — Pre-flight: explore the codebase

Before writing code, search for what already exists relevant to this task.
Read every file you will touch. Never modify a file you haven't read.

---

## Step 4 — Status report

```
PLAN : {plan filename}
TASK : {task-id} — {task title}

Files to create  : {list or "none"}
Files to modify  : {list or "none"}
Already exists   : {relevant existing files}
Recommendation   : CONTINUE | SKIP (already done) | BLOCKED (missing prereq)
```

---

## Step 5 — Implement

Implement exactly what the plan task describes. Follow:

- Code conventions from PRD.md
- Architecture patterns from PRD.md
- Financial safety rules from PRD.md
- Python 3.12+, async-first, type hints on everything
- Pydantic v2 for schemas: `model_config = ConfigDict(strict=True)`

---

## Step 6 — Write tests

Write all tests specified in the plan task:
- `@pytest.mark.unit` — pure logic, no I/O
- `@pytest.mark.integration` — real PostgreSQL, transaction rollback
- **Never mock the database** in integration tests
- Test naming: `test_{what}_{expected_outcome}`

---

## Step 7 — Verify

```bash
make format
make lint
make typecheck
pytest {test files} -v
```

Fix any failures before proceeding.

---

## Step 8 — MANDATORY: Update the plan file

After successful execution, update the plan file:
- Mark completed acceptance criteria `[x]`
- Mark completed test checkboxes `[x]`
- Update "Testing Requirements" summary
- Update `**Updated**` date
- Append execution log entry at the bottom

---

## Step 9 — Final report

```
TASK COMPLETE: {task-id} — {task title}
Plan file updated: {plan file path}

Files created:
  {list}

Files modified:
  {list}

Tests:
  Unit       : {N} passed
  Integration: {N} passed (or "requires TEST_DATABASE_URL")

Remaining tasks in this plan:
  {list or "None — plan complete"}

Run next:
  /run-plan {plan-file} {next-task-id}
```
