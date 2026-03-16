# Financial Domain Knowledge Base

This document is the reference for financial-specific concerns in this project.
It is loaded by `/brainstorm` and `/scaffold` to inform architecture decisions.
Each section represents a **decision area** — during brainstorming, the user chooses
which options apply to their use case, and those choices flow into the PRD.

---

## 1. Monetary Precision & Arithmetic

### Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **A. Decimal + ROUND_HALF_EVEN** | Python `Decimal` everywhere, banker's rounding, `Numeric(19,4)` in DB | Most financial apps — payments, invoicing, accounting |
| **B. Integer cents (minor units)** | Store amounts as integers (e.g., 1000 = $10.00), avoid floating point entirely | High-throughput systems, crypto (satoshis), simple billing |
| **C. Multi-precision** | Different precision per currency (e.g., JPY has 0 decimals, BTC has 8) | Multi-currency platforms, forex, crypto exchanges |

### Caveats
- **Option A**: Serialization traps — `Decimal` must be serialized as `str` in JSON APIs, never `float`
- **Option B**: Simpler but requires consistent conversion at API boundaries; display logic lives in the presentation layer
- **Option C**: Most complex — requires a currency registry with precision metadata; rounding rules vary per currency

---

## 2. Transaction Immutability & State Machine

### Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **A. Strict immutability** | Terminal states (`SETTLED`, `REVERSED`, `FAILED`) can never be mutated | Payments, ledger entries, compliance-heavy systems |
| **B. Append-only ledger** | Records are never updated — corrections are new entries (credit/debit pairs) | Double-entry bookkeeping, audit-critical systems |
| **C. Soft state with TTL** | Pending states auto-expire; only terminal states are immutable | Reservation systems, hold-and-capture flows |

### Caveats
- **Option A**: Simple to implement but reversal requires creating a new counter-transaction
- **Option B**: Most audit-friendly but higher storage cost; requires balance computation from event stream
- **Option C**: Needs a background worker/scheduler for expiration; race conditions around TTL boundaries

---

## 3. Concurrency Control

### Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **A. Optimistic locking (version column)** | Every aggregate has a `version` field; UPDATE checks `WHERE version = N` | Most CRUD-heavy financial apps with moderate contention |
| **B. Pessimistic locking (SELECT FOR UPDATE)** | Acquire row-level lock before mutation | High-contention scenarios (e.g., shared wallet balance) |
| **C. Event sourcing** | State derived from event stream; no direct UPDATE at all | Complex domain logic, full audit trail, CQRS |

### Caveats
- **Option A**: Clean and simple; fails gracefully with a 409 Conflict on contention
- **Option B**: Can cause deadlocks under high load; requires careful lock ordering
- **Option C**: Highest complexity; requires event store, projections, snapshots — overkill for simple CRUD

---

## 4. Idempotency Strategy

### Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **A. Idempotency-Key header + DB UNIQUE** | Client sends a key; server checks before INSERT, DB constraint is final guard | Payment creation, any non-retriable mutation |
| **B. Natural key deduplication** | Use business identifiers (e.g., `order_id + payment_type`) as uniqueness key | When clients can't generate UUIDs; batch processing |
| **C. At-least-once + idempotent handlers** | Accept duplicates; handlers check current state and short-circuit if already done | Event-driven systems, message queue consumers |

### Caveats
- **Option A**: Most explicit; requires clients to generate unique keys (UUID v4 recommended)
- **Option B**: Works well for structured workflows but brittle if business keys change
- **Option C**: Simplest for producers but pushes complexity to consumers; state checks add latency

---

## 5. Audit Trail & Compliance

### Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **A. Column-level audit** | `created_by_id`, `updated_by_id`, `created_at`, `updated_at` on every table | Basic compliance, know who changed what |
| **B. Audit log table** | Separate `audit_log` table recording every mutation with before/after snapshots | SOX compliance, PCI-DSS, regulated industries |
| **C. Event sourcing as audit** | The event stream IS the audit trail — complete history by design | When already using event sourcing (Option 3C) |

### Caveats
- **Option A**: Cheapest to implement but only records the last modifier, not the full history
- **Option B**: Full history but adds write amplification (every mutation = 2 writes); log table grows fast
- **Option C**: Richest audit but requires the full event sourcing infrastructure

---

## 6. Soft Delete vs Hard Delete

### Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **A. Soft delete (deleted_at)** | `WHERE deleted_at IS NULL` on all queries; records are never physically removed | Most financial apps — regulatory retention, undo support |
| **B. Hard delete with archive** | Delete from hot table, move to archive/cold storage | High-volume tables where soft-delete degrades query performance |
| **C. Retention-policy delete** | Soft delete initially; background job hard-deletes after retention period (e.g., 7 years) | Compliance with data retention AND right-to-erasure (GDPR) |

### Caveats
- **Option A**: Simple but tables grow forever; partial indexes on `deleted_at IS NULL` are essential
- **Option B**: More complex; requires archive table schema sync; hard to undo
- **Option C**: Best of both worlds but requires a scheduled job and retention policy configuration

---

## 7. Multi-Currency Support

### Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **A. Single currency** | One currency hardcoded or configured at app level | Domestic-only apps, single-market products |
| **B. Per-record currency** | `currency` column (ISO 4217) on every monetary record; no implicit conversion | Multi-currency storage, user wallets in different currencies |
| **C. Base currency + conversion** | Store in original currency AND convert to base currency for reporting | Multi-currency with consolidated reporting/analytics |

### Caveats
- **Option A**: Simplest; no conversion logic needed
- **Option B**: Must enforce same-currency arithmetic (never add USD + EUR directly)
- **Option C**: Requires exchange rate service, rate snapshots at transaction time, rounding policy per currency pair

---

## 8. Balance Computation

### Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **A. Stored balance (materialized)** | `balance` column on account/wallet, updated on every transaction | Low-latency balance reads, simple systems |
| **B. Computed balance (derived)** | `SUM(credits) - SUM(debits)` computed on read | Audit-correct by design, no drift possible |
| **C. Hybrid (cached + verified)** | Stored balance for reads; periodic reconciliation job verifies against computed | Best of both — fast reads, provable correctness |

### Caveats
- **Option A**: Fast reads but risk of drift if a transaction write fails mid-way; needs careful locking
- **Option B**: Provably correct but slow for accounts with many transactions; needs pagination/windowing
- **Option C**: Production-grade but adds operational complexity (reconciliation jobs, drift alerts)

---

## 9. API Amount Serialization

### Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **A. String amounts** | `"amount": "10.50"` — all monetary values as strings in JSON | Safest — no floating point surprises across languages |
| **B. Integer minor units** | `"amount_cents": 1050` — integers representing smallest currency unit | When clients prefer integers; mobile/embedded systems |
| **C. Object with currency** | `"amount": {"value": "10.50", "currency": "USD"}` | Multi-currency APIs where currency must always travel with amount |

### Caveats
- **Option A**: Requires client-side parsing to numeric type; most payment APIs use this (Stripe, PayPal)
- **Option B**: No parsing needed but display formatting pushed to client; awkward for multi-precision currencies
- **Option C**: Most explicit but verbose; every amount field becomes an object

---

## 10. Error Handling for Financial Operations

### Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **A. Domain exceptions + HTTP mapping** | Typed domain exceptions mapped to HTTP status codes at the API layer | Standard REST APIs |
| **B. Result type (Ok/Err)** | Handlers return `Result[DTO, Error]`; no exceptions cross layer boundaries | When you want exhaustive error handling at compile time |
| **C. Problem Details (RFC 9457)** | Standardized error response format with `type`, `title`, `status`, `detail` | Public APIs, OpenAPI-first design |

### Caveats
- **Option A**: Simple and familiar; risk of unhandled exceptions leaking internal details
- **Option B**: More explicit but adds boilerplate; every caller must handle both branches
- **Option C**: Best for API consumers but requires consistent implementation across all endpoints

---

## How This Feeds Into the PRD

During `/brainstorm`, the architect walks through each section above and helps the user
choose the right option for their use case. The chosen options are recorded in the PRD
under a "Financial Design Decisions" section, which `/scaffold` then uses to generate
the appropriate boilerplate code, base classes, and infrastructure patterns.
