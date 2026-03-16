---
description: Run format → lint → typecheck → unit tests, then commit staged changes with a conventional commit message. Usage: /commit [optional message hint]
argument-hint: "[optional message hint]"
allowed-tools: Bash, Read, Glob, Grep
---

You are performing a safe, pre-validated git commit for a FastAPI financial project.
Work through the steps below **in order**. Stop and report to the user if any step fails — do not skip failures.

## Project context

- Language: Python 3.12+, async-first
- Architecture: defined in `PRD.md` (if it exists) — check for specifics
- Test markers: `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.e2e`
- No hard-coded credentials

Optional message hint from user: $ARGUMENTS

---

## Step 1 — Check working tree

Run `git status` and `git diff --stat HEAD` to understand what is staged and what is not.

- If **nothing is staged**, tell the user and stop. Do not auto-stage everything.
- Show the user a summary of staged vs unstaged files before proceeding.

```bash
git status
git diff --stat HEAD
```

---

## Step 2 — Format

```bash
make format
```

If ruff modifies any files that were already staged, re-stage them automatically:

```bash
git diff --name-only | xargs -r git add
```

Report any files that were reformatted.

---

## Step 3 — Lint

```bash
make lint
```

- If lint fails, **show the errors** and stop. Do not commit over lint failures.

---

## Step 4 — Type check

```bash
make typecheck
```

- If mypy reports errors, **show the errors** and stop.

---

## Step 5 — Unit tests

```bash
make test-unit
```

- If any unit test fails, **show the failure output** and stop.
- Remind the user to run `make test-integration` separately if they changed infrastructure code.

---

## Step 6 — Review staged diff

```bash
git diff --cached
```

Check for these issues:
- Any `float` used for monetary values (check PRD for chosen monetary strategy)
- Any hard-coded secrets or credentials
- Any `allow_origins=["*"]` or `allow_methods=["*"]` in CORS config
- Any raw f-string SQL queries

If PRD.md exists, also check architecture-specific rules (e.g., no `HTTPException` in domain layer if using clean architecture).

If any issues found, **report them and stop** — do not commit.

---

## Step 7 — Inspect recent commits for style

```bash
git log --oneline -10
```

---

## Step 8 — Draft commit message

Write a conventional commit message based on the staged diff and optional hint: `$ARGUMENTS`

Format:
```
<type>(<scope>): <short summary>

<optional body: what changed and why, max 72 chars per line>
```

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `ci`

Rules:
- Summary line ≤ 72 characters
- Use imperative mood ("add", "fix", "remove" — not "added", "fixes")
- Reference issue/ticket numbers if provided in `$ARGUMENTS`

Show the drafted message to the user and ask for confirmation before committing.

---

## Step 9 — Commit

```bash
git commit -m "$(cat <<'EOF'
<your drafted message here>

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## Step 10 — Post-commit summary

After a successful commit:
1. Show commit hash and message (`git log -1 --oneline`)
2. Remind about integration tests if infrastructure code was changed
3. Do **not** push unless the user explicitly asks
