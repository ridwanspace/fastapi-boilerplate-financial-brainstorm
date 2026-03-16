---
description: Run all fresh setup steps — configure env, start infrastructure, create test DB, run migrations, and verify the server starts. Usage: /setup
argument-hint: ""
allowed-tools: Bash, Read, Write, Edit
---

You are performing a guided first-time setup for this FastAPI financial project.
The user has already cloned their repo and is in the correct Python environment.

Work through each step **in order**. After each step, verify it succeeded before continuing.
If a step fails, stop, explain the error clearly, and suggest a fix — do not proceed past failures.

---

## Step 1 — Verify Python environment

```bash
python --version
which python
```

- If Python < 3.12 → stop and tell the user to activate the correct environment.

---

## Step 2 — Install dependencies

```bash
make dev
```

Verify:
```bash
python -c "import fastapi; print('FastAPI', fastapi.__version__)"
```

---

## Step 3 — Configure environment

```bash
ls -la .env 2>/dev/null && echo "EXISTS" || echo "MISSING"
```

- If `.env` is **missing** → copy from example:
```bash
cp .env.example .env
```

---

## Step 4 — Validate `.env` required fields

Read `.env` and check that `DATABASE_URL`, `REDIS_URL`, `JWT_SECRET_KEY` are set and not placeholders.

- `JWT_SECRET_KEY` must be ≥ 32 chars. Generate with: `openssl rand -hex 32`
- `DATABASE_URL` should match Docker postgres: `postgresql+asyncpg://postgres:postgres@localhost:5432/financial_db`
- `REDIS_URL` → `redis://localhost:6379/0`

Do **not** write values into `.env` automatically — tell the user what to set.

---

## Step 5 — Start infrastructure

```bash
docker compose -f docker/docker-compose.yml up -d postgres redis
```

Wait for health checks (up to 30 seconds):
```bash
for i in $(seq 1 6); do
  STATUS=$(docker compose -f docker/docker-compose.yml ps --format json 2>/dev/null | python -c "
import sys, json
lines = sys.stdin.read().strip().split('\n')
services = [json.loads(l) for l in lines if l]
healthy = all(s.get('Health') == 'healthy' for s in services if s.get('Service') in ['postgres', 'redis'])
print('healthy' if healthy else 'waiting')
" 2>/dev/null || echo "waiting")
  if [ "$STATUS" = "healthy" ]; then
    echo "All services healthy"
    break
  fi
  echo "Waiting for services... ($i/6)"
  sleep 5
done
docker compose -f docker/docker-compose.yml ps
```

---

## Step 6 — Create the test database

```bash
docker compose -f docker/docker-compose.yml exec postgres \
  psql -U postgres -lqt | grep -c financial_test_db || echo "0"
```

If `0`:
```bash
docker compose -f docker/docker-compose.yml exec postgres \
  psql -U postgres -c "CREATE DATABASE financial_test_db;"
```

---

## Step 7 — Run migrations

```bash
make migrate
```

Verify:
```bash
python -m alembic current
```

---

## Step 8 — Run unit tests

```bash
make test-unit
```

- If tests pass → continue.
- If tests fail → show failures and stop.

---

## Step 9 — Start the server (smoke test)

```bash
python -m uvicorn src.main:app --host 0.0.0.0 --port 8000 &
SERVER_PID=$!
sleep 4
HTTP_STATUS=$(curl -s -o /tmp/health_response.json -w "%{http_code}" http://localhost:8000/api/v1/health)
cat /tmp/health_response.json
kill $SERVER_PID 2>/dev/null
wait $SERVER_PID 2>/dev/null
```

---

## Step 10 — Setup complete summary

```
Setup complete!

  Python:      <version>
  Environment: <venv/conda path>
  Database:    postgresql://localhost:5432/financial_db  (healthy)
  Redis:       redis://localhost:6379/0  (healthy)
  Migrations:  up to date
  Unit tests:  passed

Start the dev server:
  make run

Open the API:
  http://localhost:8000/docs
  http://localhost:8000/api/v1/health

Next steps:
  - Run /brainstorm to decide on architecture (if PRD.md doesn't exist yet)
  - Run /scaffold to generate project structure (if PRD.md exists)
  - Run /plan <feature> to start planning
```
