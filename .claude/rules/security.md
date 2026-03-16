# Security Rules

- **Never hard-code credentials** — all secrets via environment variables only
- JWT secret must be ≥ 32 characters — enforced by `Field(min_length=32)` in `Settings`
- CORS: never use `allow_origins=["*"]`, `allow_methods=["*"]`, or `allow_headers=["*"]` — always explicit lists
- Rate limiting: all new auth-adjacent endpoints must have explicit `@_limiter.limit(...)` decorator
- Input validation: validate at system boundaries (API layer) — trust internal layer data
- SQL: always use SQLAlchemy ORM or parameterised statements — never f-string or `.format()` into queries
- GCS signed URLs: set appropriate expiry (default 15 minutes) — never generate permanent public URLs
- Do not log sensitive values: no passwords, tokens, PII, or raw amounts with account identifiers together
