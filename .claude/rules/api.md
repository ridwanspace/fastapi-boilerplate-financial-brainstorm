---
paths:
  - "src/**/api/**/*.py"
  - "src/api/**/*.py"
---

# API Layer Rules

## Request / Response Schemas

- All request schemas: `model_config = ConfigDict(strict=True)`
- Check `PRD.md` for the chosen API amount serialization strategy (string amounts, integer cents, or amount objects)
- Include version and idempotency fields in financial resource responses if those patterns were chosen in `PRD.md`

## Router Conventions

- Map exceptions to HTTP status codes at the API layer — not in services/handlers
- `POST` endpoints that create resources: accept `Idempotency-Key` header if idempotency was chosen in `PRD.md`
- Always inject authentication dependencies on protected routes
- FastAPI and slowapi require certain parameters in handler signatures even when unused
  (`request: Request` for rate-limited routes). Suppress with `# noqa: ARG001`

## Rate Limiting

- Auth endpoints must be decorated with `@_limiter.limit("10/minute")`
- Token refresh: `@_limiter.limit("20/minute")`
- New sensitive endpoints: apply explicit per-route limits
